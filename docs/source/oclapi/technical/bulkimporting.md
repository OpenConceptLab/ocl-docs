# Bulk Importing

## Overview
OCL exposes a method for submitting a bulk import file to the OCL API that is processed asynchronously on the server. A bulk import file may include creates, updates, or deletes for multiple owners and repositories. This approach is significantly more efficient than using individual REST API calls to modify or create one resource at a time. A bulk import file is processed using the credentials provided in the bulk import request.

Note that OCL also provides two Django management commands for running imports directly on the OCL server by a system administrator (see [Server-Side Bulk Imports](#Server-Imports)). Use of the Django management commands is deprecated, however, and we do not guarantee support moving forward. The bulk import API method presented here will replace these Django management commands in the future.

## Bulk Import Scripts
### Syntax
A bulk import script is a JSON lines file, where each line is an OCL-formatted JSON resource. The syntax of each resource is the same as described elsewhere in the OCL documentation, with three modifications:
* Each resource must include a `type` attribute specifying a valid resource type, eg `Concept`, `Source`, `Organization`
* For all resources other than orgs and users, each resource must define an owner and, if applicable, a repository. These are defined using one or more of these attributes: `owner`, `owner_type`, `source`, `collection`.
* Each resource may optionally provide processing directives. Currently supported processing directives are:
    * `__action`: There are 4 action types supported:
        * `CREATE_OR_UPDATE` (default) - By default, the bulk importer will attempt to update a resource if it already exists; otherwise it will try to create a new resource.
        * `CREATE` - The bulk importer will attempt to create a new resource regardless of whether it already exists
        * `UPDATE` - The bulk importer will attempt to update a resource regardless of whether it exists
        * `DELETE` - The bulk importer will attempt to delete a resource
        * _`SKIP`_ (not currently implemented) - The bulk importer will skip the resource
        * _`DELETE_IF_EXISTS`_ (not currently implemented) - The bulk importer will attempt to delete a resource if it confirms that it exists
    * `__cascade`: For resources of type `Reference`, it is possible to specify whether and how mappings are cascaded:
        * `None` (default) - No cascading will occur. Only the
        * `sourcemappings` - Mappings stored in the same source whose `from_concept` matches a concept that is being added to a collection will also be added

### Bulk Import Script Example
The following bulk import script would create an organization, a source, and a concept:
```
{"type": "Organization", "id": "MyOrg", "name": "My Demo Organization"}
{"type": "Source", "id": "MyTestSource", "short_code": "MyTestSource", "name": "My Test Source", "full_name": "My Test Source", "owner": "MyOrg", "owner_type": "Organization", "description": "Using this source just for testing purposes", "source_type": "Dictionary", "public_access": "View", "default_locale": "en", "supported_locales": "en", "custom_validation_schema": "None"}
{"type": "Concept", "retired": false, "datatype": "None", "concept_class": "Disaggregate", "source": "MyTestSource", "extras": null, "descriptions": null, "owner": "MyOrg", "owner_type": "Organization", "external_id": "HSpL3hSBx6F", "id": "HSpL3hSBx6F", "names": [{"locale": "en", "locale_preferred": true, "external_id": null, "name": "50+, Male, Negative", "name_type": "Fully Specified"}]}
```

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

## Get a list of active and recent bulk imports for a user in a specified queue
```
GET /manage/bulkimport/:queue/
```
* GET Request Parameters:
  * Root user only:
    * **username** - optionally filter by username; for root, bulk imports for all users are returned by default

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
