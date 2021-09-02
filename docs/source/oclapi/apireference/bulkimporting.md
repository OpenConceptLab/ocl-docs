# Bulk Import API

## Overview
OCL exposes a method for submitting a bulk import file to the OCL API that is processed asynchronously on the server. A bulk import file may include creates, updates, or deletes for multiple owners and repositories. This approach is significantly more efficient than using individual REST API calls to modify or create one resource at a time. A bulk import file is processed using the credentials provided in the bulk import request.

Note that OCL also provides two Django management commands for running imports directly on the OCL server by a system administrator (see [Server-Side Bulk Imports](#Server-Imports)). Use of the Django management commands is deprecated, however, and we do not guarantee support moving forward. The bulk import API method presented here will replace these Django management commands in the future.

## Bulk Import Scripts
### Syntax
A bulk import script is a JSON lines file, where each line is an OCL-formatted JSON resource. The syntax of each resource is the same as described elsewhere in the OCL documentation, with three modifications:
* Each resource must include a `type` attribute specifying any of OCL's valid resource types, eg: `Organization`, `Source`, `Collection`, `Source Version`, `Collection Version`, `Concept`, `Mapping`, or `Reference`. 
* For all resources other than orgs and users, each resource must define an owner and, if applicable, a repository. These are defined using one or more of these attributes:
    * `owner` - Either a username or an organization mnemonic, based on the value of `owner_type`
    * `owner_type` - Either `Organization` or `User`
    * `source` - Mnemonic of a source, for a `Source Version`, `Concept`, `Mapping` or other relevant resource
    * `collection` - Mnemonic of a collection, for a `Collection Version`, `Reference` or other relevant resource
* Each resource may optionally provide processing directives. Currently supported processing directives are:
    * `__action`: There are 4 action types supported:
        * `CREATE_OR_UPDATE` (default) - By default, the bulk importer will attempt to update a resource if it already exists; otherwise it will try to create a new resource.
        * `CREATE` - The bulk importer will attempt to create a new resource regardless of whether it already exists
        * `UPDATE` - The bulk importer will attempt to update a resource regardless of whether it exists
        * `DELETE` - The bulk importer will attempt to delete a resource. Note that `DELETE` requests in a bulk import script occur synchronously. 
        * _`SKIP`_ (not currently implemented) - The bulk importer will skip the resource
        * _`DELETE_IF_EXISTS`_ (not currently implemented) - The bulk importer will attempt to delete a resource if it confirms that it exists
    * `__cascade`: For resources of type `Reference`, it is possible to specify whether and how mappings are cascaded:
        * `None` (default) - No cascading will occur.
        * `sourcemappings` - Mappings stored in the same source whose `from_concept` matches a concept that is being added to a collection will also be added. Note that the `to_concept` for each mapping is not added.

### Bulk Import Script Example
The following bulk import script would create an organization, a source, and a concept:
```
{"type": "Organization", "id": "MyOrg", "name": "My Demo Organization"}
{"type": "Source", "id": "MyTestSource", "short_code": "MyTestSource", "name": "My Test Source", "full_name": "My Test Source", "owner": "MyOrg", "owner_type": "Organization", "description": "Using this source just for testing purposes", "source_type": "Dictionary", "public_access": "View", "default_locale": "en", "supported_locales": "en", "custom_validation_schema": "None"}
{"type": "Concept", "retired": false, "datatype": "None", "concept_class": "Disaggregate", "source": "MyTestSource", "extras": null, "descriptions": null, "owner": "MyOrg", "owner_type": "Organization", "external_id": "HSpL3hSBx6F", "id": "HSpL3hSBx6F", "names": [{"locale": "en", "locale_preferred": true, "external_id": null, "name": "50+, Male, Negative", "name_type": "Fully Specified"}]}
```

## Parallel vs Inline Processing
By default, OCL attempts to process bulk imports in parallel using multiple workers where it can, providing a significant performance improvement. OCL will process a sequential list of resources of the same type, eg `Concept` or `Mapping`, in parallel, pausing before moving onto a resource of a different type. For example, if a bulk import script contains 5 concepts and 5 mappings, in that order, the 5 concepts would be processed in parallel and then the 5 mappings would be processed in parallel after the concepts had all been processed.

Note that any `DELETE` action will occur in sequence and finish processing before moving to the next resource.


## Bulk Import Permissions
The bulk importer processes a bulk import script using the credentials provided in the bulk import request (eg. the `Authorization` request header). All actions taken by the bulk importer use these credentials, meaning that the user must have the required permissions for each action. This includes GET requests that the bulk importer submits to determine whether resources already exist in OCL.

## POST a Bulk Import to the standard queue
* Post a JSON bulk import file for asynchronous processing in the standard queue. The standard queue has multiple workers processing in parallel, and therefore bulk imports may not be processed in the order that they are submitted.
```
POST /manage/bulkimport/
```

* POST Request Parameters:
  * **test_mode** - default=`false`; set to `true` to only run a test import \<NOT CURRENTLY SUPPORTED!\>
  * **update_if_exists** - default=`true`; set to `false` to skip updating resources that already exist

## POST a Bulk Import to a user assigned queue
* Adds a JSON bulk import file for asynchronous processing in a user assigned queue. User assigned queues process bulk import files using only one worker, therefore guaranteeing that they will be processed in the order in which they are submitted.
```
POST /manage/bulkimport/:queue/
```
  * `:queue` - User-assigned queue mnemonic

* POST Request Parameters:
  * **test_mode** - default=`false`; set to `true` to only run a test import \<NOT CURRENTLY SUPPORTED!\>
  * **update_if_exists** - default=`true`; set to `false` to skip updating resources that already exist

## Get a list of active and recent bulk imports for a user in the standard and user assigned queues
```
GET /manage/bulkimport/
```
* GET Request Parameters:
  * Root user only:
    * **username** - optionally filter by username; for root, bulk imports for all users are returned by default

NOTE: Returns an empty list `[]` if no recent or active bulk imports are queued

## Get a list of active and recent bulk imports for a user in a specified queue
```
GET /manage/bulkimport/:queue/
```
* GET Request Parameters:
  * Root user only:
    * **username** - optionally filter by username; for root, bulk imports for all users are returned by default

NOTE: Returns an empty list `[]` if no recent or active bulk imports are queued

## Get the status or results of a previously submitted bulk import
```
GET /manage/bulkimport/?task=:taskid[&result=:format]
```
* GET Request Parameters:
  * **task** (Required for GET request) - Task ID of a previously submitted bulk import request
  * **result** (Optional) - default="summary"; format of the results to be returned. Options are:
    * **summary** -- one line of plain text (see `OclImportResults.get_detailed_summary()`)
```
Processed 348 of 348 -- 346 NEW (200:39, 201:307); 1 UPDATE (200:1); 1 DELETE (200:1)
```

* **report** -- longer report of plain text (see `OclImportResults.display_report()`)

```
REPORT OF IMPORT RESULTS:
/orgs/DATIM-MOH-BW-FY19/collections/HTS-TST-N-MOH-HllvX50cXC0/:
NEW 200:
[{"message": "Added the latest versions of concept to the collection. Future updates will not be added automatically.",
"added": true, "expression":
...
```

* **json** -- full results object serialized to JSON (see `OclImportResults.to_json()`)

```
{
    "count": 348,
    "elapsed_seconds": 94.10947012901306,
    "total_lines": 348,
    "num_skipped": 0,
    "results": {
        "/orgs/DATIM-MOH-BW-FY19/collections/HTS-TST-N-MOH-HllvX50cXC0/": {
            "NEW": {
                "200": [
                    {
                        "obj_type": "Reference",
                        "text": "{\"data\": {\"expressions\": [\"/orgs/DATIM-MOH-BW-FY19/sources/DATIM-Alignment-Indicators/mappings/MAP-DATIM-HAS-OPTION-HTS_TST_N_MOH-HllvX50cXC0/\", \"/orgs/PEPFAR/sources/DATIM-MOH-FY19/concepts/HTS_TST_N_MOH/\", \"/orgs/PEPFAR/sources/DATIM-MOH-FY19/concepts/HllvX50cXC0/\", \"/orgs/DATIM-MOH-BW-FY19/sources/
```

* Notes:
  * Bulk imports are processed on behalf of the requesting user. Each line in an import file is processed separately and the user must have appropriate permissions for each line to be processed successfully.
  * The payload for POSTs should contain resources to be created (each in a new line). Note that you are able to mix multiple resource types.

### Example Request - POST
```
POST /manage/bulkimport/
```
```
{"type": "Source", "id": "JonTestSource", "short_code": "JonTestSource", "name": "Jon test source", "full_name": "Jon test source", "owner": "paynejd", "owner_type": "User", "description": "", "source_type": "Indicator Registry", "public_access": "View", "default_locale": "en", "supported_locales": "en", "custom_validation_schema": "None"}
{"retired": false, "datatype": "None", "type": "Concept", "concept_class": "Disaggregate", "source": "JonTestSource", "extras": null, "descriptions": null, "owner": "paynejd", "owner_type": "User", "external_id": "HSpL3hSBx6F", "id": "HSpL3hSBx6F", "names": [{"locale": "en", "locale_preferred": true, "external_id": null, "name": "50+, Male, Negative", "name_type": "Fully Specified"}]}
```

###  Response - POST
* In response you will receive a JSON with `status` and `task` attributes. The `task` attribute contains a UUID of the asynchronous task, which you can use to further query for the status of the task or, when processing is complete, the results of the import. For example:
```
{
    "status": "PENDING",
    "task": "2344a457-cfdf-4985-ae0f-b2797d33a1a2"
}
```

### Example Request - GET
```
GET /manage/bulkimport/?task=2344a457-cfdf-4985-ae0f-b2797d33a1a2&result=json
```

### Response - GET
* 200: Import finished processing (though there may have been an error)
    * If the import is complete, the response will contain the results of the import in the requested format (summary, report, or JSON)
* 200 (in the future this will be 202):
    ***PENDING**: If the bulk import is queued and processing has not yet begun, the response will have a status of `PENDING` (the same as above)
    * **STARTED**: If the bulk import is being processed and is not yet complete, the response will have a status of `STARTED`
* 404: The import task ID was not found for the requesting user


## CSV Import

### Overview

OCL provides python scripts and an API endpoint for bulk importing directly from a CSV file. An OCL bulk import CSV file must be compatible with the standard CSV bulk import template, which is described here. The CSV format is often a simpler format to work in than JSON, and allows users to more easily collaborate on creating a bulk import script in a spreadsheet. However, the CSV format does not support all OCL API features. For features not supported by the standard CSV template, you may compose bulk import scripts in JSON, which supports all OCL features. For advanced use cases, you may also implement a custom CSV to JSON converter using [ocldev.oclcsvtojsonconverter](https://github.com/OpenConceptLab/ocldev/blob/master/ocldev/oclcsvtojsonconverter.py#L20).

A few notes:
* **Resource types:** Multiple resource types (eg. concepts, mappings, organizations, sources, and collections) can be mixed in a single CSV bulk import file.
* **Order matters:** Resources are imported in the order that they appear in a bulk import script. Mappings, collection references, and source/collection versions must come after any resource that they refer to, or those resources must already exist in the target OCL environment.
* Columns are designed to reuse the same columns across resource types where possible.
* Columns that are not applicable to a particular resource type are ignored. Optional columns are omitted if blank.

### Example 1: Simple of adding concepts & mappings to existing source

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
    
```text
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

#### Use case 2: Advanced use case — loading a complete set of starter content

Organize the CSV to make sure the order of metadata is `Organizations>Sources>Concepts>Mappings`

This [Example CSV Import file](https://docs.google.com/spreadsheets/d/1pM3XlcFw5f3UJggPjIm43RkKpdsnBGhoOclTlWSI5Y8/edit?usp=sharing) has a complete set of starter content which can be used as a template to load an Organization, Sources, Collections, Concepts and Mappings.


### Running the CSV import using a Python script

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

#### CSV Import Reference (Columns)
##### Organization
* **resource_type** - "Organization"
* **id** - OCL resource identifier for the organization
* **name** - Name of the organization
* **company** (Optional)
* **website** (Optional)
* **location** (Optional)
* **public_access** (Optional) - default="View"
* **attr:\<custom-attribute-key\>** (Optional) - custom attributes
* **attr_value[\<index\>]**, **attr_key[\<index\>]** (Optional) - custom attributes

##### Repositories (Sources and Collections)
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

##### Concepts
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

##### Standalone Mappings (Internal or External)
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

##### Repository Versions (Sources or Collections)
* **resource_type** - "Source Version" or "Collection Version"
* **owner_id** - OCL resource identifier of the OCL user or organization that will own this object
* **owner_type** (Optional) - "Organization" or "User"; default="Organization"
* **source** OR **collection** - identifier of the source or collection for the new repository version
* **id** - Identifier for the repository version (e.g. "v1.0")
* **description** - Description of the repository version
* **released** (Optional) - default=False
* **retired** (Optional) - default=False
