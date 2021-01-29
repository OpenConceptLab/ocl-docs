# CSV Import

## Overview

OCL provides python scripts and an API endpoint for bulk importing directly from a CSV file. An OCL bulk import CSV file must be compatible with the standard CSV bulk import template, which is described here. The CSV format is often a simpler format to work in than JSON, and allows users to more easily collaborate on creating a bulk import script in a spreadsheet. However, the CSV format does not support all OCL API features. For features not supported by the standard CSV template, you may compose bulk import scripts in JSON, which supports all OCL features. For advanced use cases, you may also implement a custom CSV to JSON converter using [ocldev.oclcsvtojsonconverter](https://github.com/OpenConceptLab/ocldev/blob/master/ocldev/oclcsvtojsonconverter.py#L20).

A few notes:
* **Resource types:** Multiple resource types (eg. concepts, mappings, organizations, sources, and collections) can be mixed in a single CSV bulk import file.
* **Order matters:** Resources are imported in the order that they appear in a bulk import script. Mappings, collection references, and source/collection versions must come after any resource that they refer to, or those resources must already exist in the target OCL environment.
* Columns are designed to reuse the same columns across resource types where possible.
* Columns that are not applicable to a particular resource type are ignored. Optional columns are omitted if blank.

## Example 1: Simple of adding concepts & mappings to existing source

This [OCL_CSV_Bulk_Import_Example_01](https://docs.google.com/spreadsheets/d/1Ih1LXEVyu3PDb262zVpQlyCPIrwu-H02Oay2eKwrIXE/edit?usp=sharing) is a basic example of loading new concepts and mappings for an existing Organization and source that are already loaded in OCL.

Organize the CSV to make sure the order of metadata is `Concepts>Mappings`

| resource_type | owner_id | source | id | name | concept_class | datatype | description | attr:test | map_source[dsme1] | map_to_concept_owner_id[dsme1] | map_to_concept_source[dsme1] | map_to_concept_id[dsme1] | extmap_to_concept_owner_id[loinc1] | extmap_to_concept_source[loinc1] | extmap_to_concept_id[loinc1] | extmap_to_concept_name[loinc1] | map_target | map_type | map_from_concept_url | map_from_concept_owner_id | map_from_concept_source | map_from_concept_id | map_to_concept_owner_id | map_to_concept_source | map_to_concept_id | map_to_concept_name |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| Concept | Example-Country-A | Example-Country-A | 5835 | ID | Question | Text | ID | TRUE |  |  |  |  | Regenstrief | LOINC | 103948 | ID |  |  |  |  |  |  |  |  |  |  |
| Concept | Example-Country-A | Example-Country-A | 5150 | Informed Code | Question | Text | Informed Code | FALSE | Mapping-Example-Country-A-to-CDD | DSME-CDD | CDD | CC01 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| Concept | Example-Country-A | Example-Country-A | 8686 | Informed Name | Question | Text | Informed Name | TRUE | Mapping-Example-Country-A-to-CDD | DSME-CDD | CDD | CC02 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| Concept | Example-Country-A | Example-Country-A | 3530 | Total Resident | Question | Numeric | Total Resident | FALSE |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| Mapping | Example-Country-A | Example-Country-A |  |  |  |  |  | hey |  |  |  |  |  |  |  |  |  | kind of thinking about | /orgs/PaperPie/sources/StinkYfeet/concepts/L-big-toe/ | spspdp |  | ji | idjeifkdkdididfjdkdkf | idj | idj | here's a to concept name |
| Mapping | Example-Country-A | Example-Country-A |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  | PurplePie | SmellySocks | LeftFoot |  |  | 9jd9 | 9dj9-name |
| Mapping | Example-Country-A | Example-Country-A |  |  |  |  |  |  |  |  |  |  |  |  |  |  | External |  |  | testowner | testsource | testcid | test-to-concept-owner | test-to-concept-sourc | test-to-concept-id | test-to-concept-name |

<details>
    <summary>Expand to view raw CSV for this example</summary>
    
```csv
resource_type,owner_id,source,id,name,concept_class,datatype,description,attr:test,map_source[dsme1],map_to_concept_owner_id[dsme1],map_to_concept_source[dsme1],map_to_concept_id[dsme1],extmap_to_concept_owner_id[loinc1],extmap_to_concept_source[loinc1],extmap_to_concept_id[loinc1],extmap_to_concept_name[loinc1],map_target,map_type,map_from_concept_url,map_from_concept_owner_id,map_from_concept_source,map_from_concept_id,map_to_concept_owner_id,map_to_concept_source,map_to_concept_id,map_to_concept_name
Concept,Example-Country-A,Example-Country-A,5835,ID,Question,Text,ID,TRUE,,,,,Regenstrief,LOINC,103948,ID,,,,,,,,,,
Concept,Example-Country-A,Example-Country-A,5150,Informed Code,Question,Text,Informed Code,FALSE,Mapping-Example-Country-A-to-CDD,DSME-CDD,CDD,CC01,,,,,,,,,,,,,,
Concept,Example-Country-A,Example-Country-A,8686,Informed Name,Question,Text,Informed Name,TRUE,Mapping-Example-Country-A-to-CDD,DSME-CDD,CDD,CC02,,,,,,,,,,,,,,
Concept,Example-Country-A,Example-Country-A,3530,Total Resident,Question,Numeric,Total Resident,FALSE,,,,,,,,,,,,,,,,,,
Mapping,Example-Country-A,Example-Country-A,,,,,,hey,,,,,,,,,,kind of thinking about,/orgs/PaperPie/sources/StinkYfeet/concepts/L-big-toe/,spspdp,,ji,idjeifkdkdididfjdkdkf,idj,idj,here's a to concept name
Mapping,Example-Country-A,Example-Country-A,,,,,,,,,,,,,,,,,,PurplePie,SmellySocks,LeftFoot,,,9jd9,9dj9-name
Mapping,Example-Country-A,Example-Country-A,,,,,,,,,,,,,,,External,,,testowner,testsource,testcid,test-to-concept-owner,test-to-concept-sourc,test-to-concept-id,test-to-concept-name
```
</details>

### Use case 2: Advanced use case — loading a complete set of starter content

Organize the CSV to make sure the order of metadata is `Organizations>Sources>Concepts>Mappings`

This [Example CSV Import file](https://docs.google.com/spreadsheets/d/1pM3XlcFw5f3UJggPjIm43RkKpdsnBGhoOclTlWSI5Y8/edit?usp=sharing) has a complete set of starter content which can be used as a template to load an Organization, Sources, Collections, Concepts and Mappings.


## Running the CSV import using a Python script

The following code snippet is an example of how to use the `ocldev` package to load a CSV file, convert to JSON, validate, and submit via the OCL bulk import API.

```python
"""
Load an OCL-formatted CSV file, convert to JSON, validate and submit to OCL via
the bulk import API.
"""
from ocldev import oclresourcelist
from ocldev import oclfleximporter
import json

# Config Settings
csv_filename = 'Simple CSV Import example.csv'
ocl_env_url = 'https://api.staging.aws.openconceptlab.org'
ocl_api_token = 'my-token-here'

# Load CSV file 
csv_resource_list = oclresourcelist.OclCsvResourceList.load_from_file(csv_filename)

# Convert CSV resources to JSON 
json_resource_list = csv_resource_list.convert_to_ocl_formatted_json()

# Print the converted JSON resources
json_data = ''
for line in json_resource_list:
    json_data += json.dumps(line,indent=1) + '\n'
print(json_data)

# Validate the converted JSON resources
json_resource_list.validate()

# Send request using Bulk import and print the status endpoint. 
# To check the status of import using the endpoint URL send a 
# request header "Authorization" with value "Token my-server-token"
import_response = oclfleximporter.OclBulkImporter.post(
    input_list=json_resource_list, api_url_root=ocl_env_url, api_token=ocl_api_token)
import_response.raise_for_status()
import_response_json = import_response.json()
bulk_import_task_id = import_response_json['task']
bulk_import_status_url = '%s/manage/bulkimport/?task=%s' % (ocl_env_url, bulk_import_task_id)

print('Bulk Import Task ID: %s' % bulk_import_task_id)
print('Bulk Import Status URL: %s' % bulk_import_status_url)
```

### CSV Import Reference (Columns)
#### Organization
* **resource_type** - "Organization"
* **id** - OCL resource identifier for the organization
* **name** - Name of the organization
* **company** (Optional)
* **website** (Optional)
* **location** (Optional)
* **public_access** (Optional) - default="View"
* **attr:\<custom-attribute-key\>** (Optional) - custom attributes
* **attr_value[\<index\>]**, **attr_key[\<index\>]** (Optional) - custom attributes

#### Repositories (Sources and Collections)
* **resource_type** - "Source" or "Collection"
* **owner_id** - OCL resource identifier of the OCL user or organization that will own this object
* **owner_type** (Optional) - "Organization" or "User"; default="Organization"
* **id** - OCL resource identifier
* **name** - Primary name of the repository
* **short_code** (Optional) - Automatically set to `id` if omitted
* **full_name** (Optional) - Automatically set to `name` if omitted
* **description** (Optional) - Description of the repository
* **external_id** (Optional) - Optional external identifier for the repository
* **source_type** / **collection_type** (Optional) - eg. "Dictionary", "Interface Terminology", "Indicator Registry", etc.
* **default_locale** (Optional) - default="en"
* **supported_locales** (Optional) - default="en"
* **website** (Optional)
* **custom_validation_schema** (Optional)
* **public_access** (Optional) - default="View"
* **attr:\<custom-attribute-key\>** (Optional) - custom attributes
* **attr_value[\<index\>]**, **attr_key[\<index\>]** (Optional) - custom attributes

#### Concepts
* **resource_type** - "Concept"
* **owner_id** - OCL resource identifier of the OCL user or organization that will own this object
* **owner_type** (Optional) - "Organization" or "User"; default="Organization"
* **source** - OCL resource identifier of the source in which this concept will be defined
* **id** - OCL resource identifier
* **retired** (Optional) - default=False
* **external_id** (Optional) -
* **concept_class** -
* **datatype** (Optional) - default="None"
* Initial name:
  * **name** -
  * **name_locale** (Optional) - default="en"
  * **name_locale_preferred** (Optional) - default=True
  * **name_type** (Optional) - default="Fully Specified"
  * **name_external_id** (Optional) -
* Additional names:
  * **name[\<index\>]** -
  * **name_locale[\<index\>]** (Optional) - default="en"
  * **name_locale_preferred[\<index\>]** (Optional) - default=False
  * **name_type[\<index\>]** (Optional) -
  * **name_external_id[\<index\>]** (Optional) -
* Initial Description:
  * **description** -
  * **description_locale** (Optional) - default="en"
  * **description_locale_preferred** (Optional) - default=False
  * **description_type** (Optional) -
  * **description_external_id** (Optional) -
* Additional Descriptions:
  * **description[\<index\>]** -
  * **description_locale[\<index\>]** (Optional) - default="en"
  * **description_locale_preferred[\<index\>]** (Optional) - default=False
  * **description_type[\<index\>]** (Optional) -
  * **description_external_id[\<index\>]** (Optional) -
* **attr:\<custom-attribute-key\>** (Optional) - custom attributes
* **attr_value[\<index\>]**, **attr_key[\<index\>]** (Optional) - custom attributes
* Internal Concept Mappings: (Where the concept defined in the row is the `from_concept`)
  * **map_target** (Optional) - default="Internal"
  * **map_owner_id** (Optional) - Automatically set to the concept `owner_id` if omitted
  * **map_owner_type** (Optional) - Automatically set to the concept `owner_type` if omitted
  * **map_source** (Optional) - Automatically set to the concept `source` if omitted
  * **map_type[\<index\>]** (Optional) - default="Same As"
  * to_concept must provide a minimum set of fields to resolve to a to_concept_url
    * **map_to_concept_url[\<index\>]** (Optional)
    * **map_to_concept_id[\<index\>]** (Optional)
    * **map_to_concept_name[\<index\>]** (Optional)
    * **map_to_concept_owner_id[\<index\>]** (Optional)
    * **map_to_concept_owner_type[\<index\>]** (Optional) - default="Organization"
    * **map_to_concept_source[\<index\>]** (Optional)
* External Concept Mappings: (Where the concept defined in the row is the `from_concept`)
  * **extmap_target** (Optional) - default="External"
  * **extmap_owner_id** (Optional) - Automatically set to the concept `owner_id` if omitted
  * **extmap_owner_type** (Optional) - Automatically set to the concept `owner_type` if omitted
  * **extmap_source** (Optional) - Automatically set to the concept `source` if omitted
  * **extmap_type[\<index\>]** (Optional) - default="Same As"
  * to_concept must provide a minimum set of fields to resolve to a to_concept_url
    * **extmap_to_concept_id[\<index\>]**
    * **extmap_to_concept_name[\<index\>]** (Optional)
    * **extmap_to_concept_owner_id[\<index\>]**
    * **extmap_to_concept_owner_type[\<index\>]** (Optional) - default="Organization"
    * **extmap_to_concept_source[\<index\>]**

#### Standalone Mappings (Internal or External)
* **resource_type** - "Mapping" or "External Mapping"
* **owner_id** - OCL resource identifier of the OCL user or organization that will own this object
* **owner_type** (Optional) - "Organization" or "User"; default="Organization"
* **source** - OCL resource identifier of the source in which this concept will be defined
* **map_type** (Optional) - default="Same As"
* from_concept must provide a minimum set of fields to resolve to a `from_concept_url`
  * **map_from_concept_url** (Optional)
  * **map_from_concept_id** (Optional)
  * **map_from_concept_owner_id** (Optional)
  * **map_from_concept_owner_type** (Optional) - default="Organization"
  * **map_from_concept_source** (Optional)
* to_concept must provide a minimum set of fields to resolve to a `to_concept_url`
  * **map_to_concept_url** (Optional)
  * **map_to_concept_id** (Optional)
  * **map_to_concept_name** (Optional)
  * **map_to_concept_owner_id** (Optional)
  * **map_to_concept_owner_type** (Optional) - default="Organization"
  * **map_to_concept_source** (Optional)

#### Repository Versions (Sources or Collections)
* **resource_type** - "Source Version" or "Collection Version"
* **owner_id** - OCL resource identifier of the OCL user or organization that will own this object
* **owner_type** (Optional) - "Organization" or "User"; default="Organization"
* **source** OR **collection** - identifier of the source or collection for the new repository version
* **id** - Identifier for the repository version (e.g. "v1.0")
* **description** - Description of the repository version
* **released** (Optional) - default=False
* **retired** (Optional) - default=False
