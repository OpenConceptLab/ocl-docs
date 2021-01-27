# CSV Import

### Overview
OCL provides APIs for importing CSV files and converting CSV files to JSON. CSV files submitted to the API must be compatible with the standard CSV template, which is designed to support the most common import scenarios. For scenarios not supported by the standard CSV template, you may work directly with [ocldev.oclcsvtojsonconverter](https://github.com/OpenConceptLab/ocldev/blob/master/ocldev/oclcsvtojsonconverter.py#L20) or compose import scripts in JSON, which support all OCL features.

Multiple resource types, owners, and repositories can be mixed in a single CSV file. Columns are designed to reuse the same columns across resource types where possible. Columns that are not applicable to a particular resource type are ignored. Optional columns are omitted if blank.

### Example:

The following code snippet uses the `ocldev` package to load a CSV file, convert to JSON, validate, and submit via the OCL bulk import API.

[Example CSV Import file](https://docs.google.com/spreadsheets/d/1pM3XlcFw5f3UJggPjIm43RkKpdsnBGhoOclTlWSI5Y8/edit?usp=sharing)

```python
from ocldev import oclresourcelist
from ocldev import oclfleximporter
import json

csv_filename = 'CSV Import Example.csv'
ocl_env_url = 'https://api.staging.aws.openconceptlab.org'
ocl_api_token = 'mytoken'

csv_resource_list = oclresourcelist.OclCsvResourceList.load_from_file(csv_filename)
json_resource_list = csv_resource_list.convert_to_ocl_formatted_json()
json_resource_list.validate()

import_response = oclfleximporter.OclBulkImporter.post(
    input_list=json_resource_list, api_url_root=ocl_env_url, api_token=ocl_api_token)
import_response.raise_for_status()
import_response_json = import_response.json()
bulk_import_task_id = import_response_json['task']

bulk_import_status_url = '%s/manage/bulkimport/?task=%s' % (ocl_env_url, bulk_import_task_id)
print('Bulk Import Task ID: %s' % bulk_import_task_id)
print('Bulk Import Status URL: %s' % bulk_import_status_url)
```

### CSV Columns
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
