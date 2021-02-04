# Export API

## Overview
The API provides an `export` endpoint for creating, fetching, and deleting a cached export of repository version. Exports are automatically generated upon creation of a new source or collection version and cached, so requesting an export is a quick operation even for a large repository. The recommended method for determining if an export is available after creating a new repository version is by checking the status code of HEAD request to the export, eg `HEAD /[:ownerType/]:owner/:repoType/:repo/:repoVersion/export/`. The status code is `303` if the export is ready for download, or `204` if it is still processing. Note that the old method of determining when an export is available is by checking the `is_processing` flag of the repository version, which is set to `False` after processing of the new repost version is complete. This method is still correct, but if you only need an export and not a fully processed repository version, then the Export API method is more performant as creating and uploading an export to S3 is a much faster operation.

The Export API enables a client to manage cached exports manually for special situations, such as triggering the creation of a new export that did not get cached correctly. The filename of the export contains the export's `lastUpdated` timestamp, which can be used to fetch the diff between an export and the current state of a source or collection. A `Location` response header is included in a successful `GET` request to redirect a client to the download URL, which is designed to be used only once. Therefore, to ensure that the `Location` URL works correctly, you must make a new export request each time you want to download an export file.

An export is a compressed zip of the JSON results of issuing a GET for a specific repository version. For example, the JSON results contained in an export are equivalent to the following request:
```
GET /[:ownerType/]:owner/:repoType/:repo/:repoVersion/?includeConcepts=true&includeMappings=true&includeRetired=true&limit=0
```
In the above:
* `:ownerType` is "orgs" or "users", or it is omitted if `:owner` is "user"
* `:owner` is a username if `:ownerType` is "users", an organization ID if `:ownerType` is "orgs", or "user"
* `:repoType` is "sources" or "collections"
* `:repo` is a source or collection ID, depending on the value of `:repoType`
* `:repoVersion` is a source or collection version ID

The export filename takes the following form:
```
[:repositoryId]_[:repoVersion].[:lastUpdated].zip
```
To fetch the diff between a `lastUpdated` timestamp and the current state of a source or collection:
```
GET /[:ownerType/]:owner/:repoType/:repo/:repoVersion/?includeConcepts=true&includeMappings=true&includeRetired=true&limit=0&updatedSince=:lastUpdated
```

Refer to [[sources]] and [[collections]] documentation for details on the above requests.

The [[Subscriptions]] documentation describes how the export functionality can be used to subscribe to a source or collection to keep a client system in synch.

### Future Considerations
* Add `lastUpdated` into the response header of GET request (right now lastUpdated is only stored in the filename)
* The current status and progress of the creation of a repository export may be made available through the Flower package



## Get an export of a repository version
* Get the export URL for the specified repository version. This has three possible results:
    * If the export exists, returns a status code of `303 See Other` with the `Location` property in the response header set to the fully specified download URL
    * If the export file does not exist but the URL is correct, returns `204 No Content`
    * If the export URL is non-existent, returns `404 Not Found`
```
GET /[:ownerType/]:owner/:repoType/:repo/:repoVersion/export/
HEAD /[:ownerType/]:owner/:repoType/:repo/:repoVersion/export/
```
* Notes
    * `:repoVersion` is required - exports can only be created for repository versions. If `HEAD` version is requested, the API will return `405 Not Allowed`.
    * The API returns the fully specified URL of the export file in the `Location` attribute of the response header. The `Location` is designed to be used only once -- to ensure that the `Location` works correctly, you must request another `Location` each time you want to download an export file.
    * This request first performs a check as to whether the export file already exists. If it exists, it then generates an `Location`. These actions are performed on an Amazon Web Server and may take up to 30 seconds to process. The timeout of the client system making the request should set its timeout period accordingly.

### Example
* Fetch the export URL for v2016-08-22 of the CIEL source
```
GET /orgs/CIEL/sources/CIEL/v2016-08-22/export/
```
* Fetch the export URL for v1.2 of the CIEL Starter Set
```
GET /orgs/CIEL/collections/StarterSet/v1.2/export/
```

### Response
* If the export file exists, follow the redirect to the `Location` response header:
```
Status: 303 See Other
Response Header:
Location: https://ocl-source-export-production.s3.amazonaws.com/CIEL/CIEL_55572f688a86f24b48d935be.20150516122820.tgz?Signature=fmBSI6hL9IhN4mu4W5x%2FPFs5uxw%3D&Expires=1431864368&AWSAccessKeyId=...
```
* If the URL is valid but the export file does not exist:
```
Status: 204 No Content
Response Header:
Location:
```
* If the URL is valid but the export file is still being processed:
```
Status: 208 Already Reported
```
* If the URL is non-existent
```
Status: 404 Not Found
```
* If the request is otherwise invalid - return the appropriate error code



## Create an export of a repository version
* Create an export file for the specified repository version; if it already exists, no action is taken
```
POST /[:ownerType/]:owner/:repoType/:repo/:repoVersion/export/
```
* Notes
    * `:repoVersion` is required. If `HEAD` version is requested, the API will return `405 Not Allowed`.
    * This request only triggers the creation of the export file and does **NOT** return the `Location`. It is necessary to follow up with a GET request after the file has been processed in order to get the `Location`

### Example
* Create the export file for v2.2 of the CIEL source
```
POST /orgs/CIEL/sources/CIEL/v2.2/export/
```

### Response
* If no export file already exists and processing is initiated:
```
Status: 202 Accepted
```
* If an export file is currently being processed:
```
Status: 409 Conflict
```
* If the export file already exists (This won't return `Location` header, to get `Location` header please use GET request)
```
Status: 303 See Other
```
* If the request is otherwise invalid - return the appropriate error code



## Delete an export file
* Deletes a specific export file
```
DELETE /[:ownerType/]:owner/:repoType/:repo/:repoVersion/export/
```
* Notes
    * If `HEAD` version is requested, the API will return `405 Not Allowed`.
    * The passed authorization token must have administrative access to the repository in order to delete the export file

### Example
* Create the export file for v2.2 of the CIEL source
```
DELETE /orgs/CIEL/sources/CIEL/v2.2/export/
```

### Response
* If the file exists, it is deleted with 204
```
Status: 204 Success
```
* If the file does NOT exist
```
Status: 404 No Content
```
* If the request is otherwise invalid - return the appropriate error code


## Full Example
```json
{
    "type": "Source",
    "uuid": "8d492ee0-c2cc-11de-8d13-0010c6dffd0f",
    "id": "ICD-10-2010",
    "external_id": "",
    "short_code": "ICD-10-2010",
    "name": "ICD-10-WHO 2010",
    "full_name": "International Classification of Diseases v10 2010",
    "source_type": "Dictionary",
    "public_access": "View",
    "default_locale": "en",
    "supported_locales": "en,fr",
    "website": "http://www.who.int/classifications/icd/",
    "description": "The International Classification of Diseases (ICD) is the standard diagnostic tool for epidemiology, health management and clinical purposes. This includes the analysis of the general health situation of population groups.",
    "extras": { "my_extra_field": "my_extra_value" },
    "owner": "WHO",
    "owner_type": "organization",
    "owner_url": "/orgs/WHO/",
    "url": "/orgs/WHO/sources/ICD-10/",
    "versions_url": "/orgs/WHO/sources/ICD-10/versions/",
    "concepts_url": "/orgs/WHO/sources/ICD-10/concepts/",
    "mappings_url": "/orgs/WHO/sources/ICD-10/mappings/",
    "versions": 3,
    "active_concepts": 15000,
    "active_mappings": 3243,
    "created_on": "2008-01-14T04:33:35Z",
    "created_by": "johndoe",
    "updated_on": "2008-02-18T09:10:16Z",
    "updated_by": "johndoe",
    "concepts": [
        {
            "type": "Concept",
            "uuid": "8d492ee0-c2cc-11de-8d13-0010c6dffd0f",
            "id": "A15.1",
            "external_id": "19jf93jf9j39fii399du9393",
            "concept_class": "Diagnosis",
            "datatype": "None",
            "retired": false,
            "display_name": "Tuberculosis of lung, confirmed by culture only",
            "display_locale": "en",
            "names": [
                {
                    "type": "ConceptName",
                    "uuid": "akdiejf93jf939f9",
                    "external_id": "1fddfenkcineh9",
                    "name": "Tuberculosis of lung, confirmed by culture only",
                    "locale": "en",
                    "locale_preferred": "true",
                    "name_type": "None"
                },
                {
                    "type": "ConceptName",
                    "uuid": "90jmcna4-lkdhf78",
                    "external_id": "12345677",
                    "name": "Tuberculose pulmonaire, confirmée par culture seulement",
                    "locale": "fr",
                    "locale_preferred": "true",
                    "name_type": "None"
                }
            ],
            "descriptions": [
                {
                    "type": "ConceptDescription",
                    "uuid": "aY873Hbmkdi09jeh",
                    "external_id": "abcdefghijklmnopqrstuvwxyz",
                    "description": "Tuberculous bronchiectasis, fibrosis of lung, pneumonia, pneumothorax, confirmed by sputum microscopy with culture only",
                    "locale": "en",
                    "locale_preferred": "true",
                    "description_type": "None"
                }
            ],
            "extras": { "parent": "A15" },
            "source": "ICD-10-2010",
            "owner": "WHO",
            "owner_type": "Organization",
            "version": "abc345jf9fj",
            "url": "/orgs/WHO/sources/ICD-10-2010/concepts/A15.1/",
            "version_url": "/orgs/WHO/sources/ICD-10-2010/concepts/A15.1/abc345jf9fj/",
            "source_url": "/orgs/WHO/sources/ICD-10-2010/",
            "owner_url": "/orgs/WHO/",
            "mappings_url": "/orgs/WHO/sources/ICD-10-2010/concepts/A15.1/mappings/",
            "extras_url": "/orgs/WHO/sources/ICD-10-2010/concepts/A15.1/extras/",
            "versions": 9,
            "created_on": "2008-01-14T04:33:35Z",
            "created_by": "johndoe",
            "updated_on": "2008-02-18T09:10:16Z",
            "updated_by": "johndoe"
        }    
    ],
    "mappings": [
        {
            "type": "Mapping",
            "uuid": "8jf8j-39fnnkdked",
            "external_id": "a9d93ffjjen9dnfekd9",
            "retired": "false",
            "map_type": "Same As",
            "from_source_owner": "WHO",
            "from_source_owner_type": "Organization",
            "from_source_name": "ICD-10-2010",
            "from_concept_code": "A15.1",
            "from_concept_code": "Tuberculosis of lung, confirmed by culture only",
            "from_source_url": "/orgs/WHO/sources/ICD-10-2010/",
            "from_concept_url": "/orgs/WHO/sources/ICD-10-2010/concepts/A15.1/",
            "to_source_owner": "IHTSDO",
            "to_source_owner_type": "Organization",
            "to_source_name": "SNOMED",
            "to_concept_code": "154283005",
            "to_concept_name": "Pulmonary Tuberculosis",
            "to_source_url": "/orgs/IHTSDO/sources/SNOMED/",
            "source": "ICD-10-2010",
            "owner": "WHO",
            "owner_type": "Organization",
            "url": "/orgs/WHO/sources/ICD-10-2010/mappings/8jf8j-39fnnkdked/",
            "created_on": "2008-01-14T04:33:35Z",
            "created_by": "johndoe",
            "updated_on": "2008-02-18T09:10:16Z",
            "updated_by": "johndoe"
        }
    ]
}
```
