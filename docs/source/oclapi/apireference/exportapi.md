# Export API

## Overview
The API provides an `export` endpoint for creating, fetching, and deleting a cached export of repository version. Exports are automatically generated upon creation of a new source or collection version and cached, so requesting an export is a quick operation even for a large repository. The recommended method for determining if an export is available after creating a new repository version is by checking the status code of a `HEAD` request to the export, eg `HEAD /[:ownerType/]:owner/:repoType/:repo/:repoVersion/export/`. The status code is `302` if the export is ready for download, `208` if it is still being created, or `204` if there is no export. Don't follow the redirect when checking availability (for example, use `curl -I` without `-L`): the signed URL is valid only for `GET`. The repository version's `is_processing` flag is not a reliable signal for this: it turns `False` when processing of the new version ends, even if creating the export failed. Use the Export API to find out whether an export exists.

The Export API enables a client to manage cached exports manually for special situations, such as triggering the creation of a new export that did not get cached correctly. The filename of the export contains the export's `lastUpdated` timestamp, which can be used to fetch the diff between an export and the current state of a source or collection. A `GET` request to the export endpoint redirects to a signed download URL, and the download carries the filename in its `Content-Disposition` response header.

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

The API names the export file, so every client saves it under the same name:
```
[:ownerType]_[:owner]_[:repoType]_[:repo]_[:repoVersion]_[:expansion]_[:lastUpdated].zip
```
* `_[:expansion]` appears only when the export includes an expansion: a collection version's default expansion.
* `:repoVersion` is the version ID as stored (OCL never adds a `v`), or `HEAD` for an export of HEAD. Characters other than letters, digits, `.`, `_`, `-` and `@` become `-` in the filename, so `1.0 beta` becomes `1.0-beta`.
* `:lastUpdated` always comes last, formatted `YYYY-MM-DD_HHMMSS` (UTC). It is taken from the cached export, so it describes the export's content: the time of the repository version's last concept or mapping change when the export was created (for a version with no concepts or mappings, the time the version was last updated).
* IDs keep their case and may contain underscores, so take the repository's identity from the JSON inside the export, not by splitting the filename.
* If a cached export's storage key carries no timestamp, the API sends no filename, and the client chooses one (most save it under the storage name).

For example:
```
orgs_CIEL_sources_CIEL_v2026-03-23_2026-03-23_073036.zip
orgs_PIH_collections_PIHEMR_Concepts_1.0_autoexpand-1.0_2026-09-30_123456.zip
```
To fetch the diff between a `lastUpdated` timestamp and the current state of a source or collection, convert the timestamp to ISO 8601 first (`2026-09-30_123456` becomes `2026-09-30T12:34:56Z`):
```
GET /[:ownerType/]:owner/:repoType/:repo/:repoVersion/?includeConcepts=true&includeMappings=true&includeRetired=true&limit=0&updatedSince=:lastUpdated
```

Refer to [[sources]] and [[collections]] documentation for details on the above requests.

The [[Subscriptions]] documentation describes how the export functionality can be used to subscribe to a source or collection to keep a client system in synch.

### Future Considerations
* Add `lastUpdated` into the response header of GET request (right now lastUpdated is only stored in the filename)
* The current status and progress of the creation of a repository export may be made available through the Flower package



## Get an export of a repository version
* Download the export for the specified repository version, or check its availability. The possible results:
    * If the export exists, `GET` returns `302 Found`, redirecting to a signed download URL. The download's `Content-Disposition` header contains the filename. `HEAD` returns the same `302` with no body (useful for checking availability without downloading).
    * If the export is still being created, returns `208 Already Reported`
    * If the export file does not exist but the URL is correct, returns `204 No Content`
    * If the export URL is non-existent, returns `404 Not Found`
    * If the export exists but no download URL could be generated, returns `500 Internal Server Error`
```
GET /[:ownerType/]:owner/:repoType/:repo/:repoVersion/export/
HEAD /[:ownerType/]:owner/:repoType/:repo/:repoVersion/export/
```
* Notes
    * `:repoVersion` is required. `HEAD` exports are available only to staff, superusers and the repository's owner (the owning user, or members of the owning organization); other users who can see the repository get `405 Not Allowed`.
    * Most HTTP clients follow the redirect automatically; for example, `curl -L -OJ` saves the file under its name. The signed URL expires, so request a new one each time you download.
    * When the API names the export, the download's `Content-Disposition` header contains the filename (e.g. `attachment; filename="orgs_CIEL_sources_CIEL_v2026-03-23_2026-03-23_073036.zip"`). The signed URL carries the same value in its `response-content-disposition` parameter.

### Example
* Download the export for v2016-08-22 of the CIEL source
```
GET /orgs/CIEL/sources/CIEL/v2016-08-22/export/
```
* Check export availability for v1.2 of the CIEL Starter Set
```
HEAD /orgs/CIEL/collections/StarterSet/v1.2/export/
```

### Response
* If the export file exists (`GET` and `HEAD` both redirect):
```
Status: 302 Found
Response Header:
Location: [signed download URL]
```
* The download from the signed URL:
```
Status: 200 OK
Response Header:
Content-Disposition: attachment; filename="orgs_CIEL_sources_CIEL_v2016-08-22_2016-08-22_101500.zip"
```
* If the URL is valid but the export file does not exist:
```
Status: 204 No Content
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
* Create an export file for the specified repository version. If one already exists, no action is taken unless `force=true` is passed, which creates it again.
```
POST /[:ownerType/]:owner/:repoType/:repo/:repoVersion/export/
POST /[:ownerType/]:owner/:repoType/:repo/:repoVersion/export/?force=true
```
* Notes
    * `:repoVersion` is required. `HEAD` exports are available only to staff, superusers and the repository's owner (the owning user, or members of the owning organization); other users who can see the repository get `405 Not Allowed`.
    * This request only triggers the creation of the export file and does **NOT** return the export. It is necessary to follow up with a GET request after the file has been processed in order to download it.

### Example
* Create the export file for v2.2 of the CIEL source
```
POST /orgs/CIEL/sources/CIEL/v2.2/export/
```

### Response
* The API checks these cases in order: an export being created (`208`) takes precedence over `force` and `noRedirect`, and `force=true` takes precedence over `noRedirect`.
* If an export file is currently being created:
```
Status: 208 Already Reported
```
* If no export file already exists (or `force=true` was passed) and processing is initiated:
```
Status: 202 Accepted
```
* If the same export job is already queued:
```
Status: 409 Conflict
```
* If the export file already exists, the response points to the export endpoint; download it with a GET to the same export URL you posted to. For a `HEAD` export, the `URL` header leaves out `HEAD/`, so use the URL you posted to rather than the header.
```
Status: 303 See Other
Response Header:
URL: /[:ownerType/]:owner/:repoType/:repo/:repoVersion/export/
```
* If the export file already exists and `noRedirect=true` was passed:
```
Status: 204 No Content
```
* If the request is otherwise invalid - return the appropriate error code



## Delete an export file
* Deletes a specific export file
```
DELETE /[:ownerType/]:owner/:repoType/:repo/:repoVersion/export/
```
* Notes
    * `HEAD` exports are available only to staff, superusers and the repository's owner (the owning user, or members of the owning organization); other users who can see the repository get `405 Not Allowed`.
    * The passed authorization token must have administrative access to the repository (staff, superusers or the repository's owner) in order to delete the export file; otherwise the API returns `403 Forbidden`.
    * DELETE removes the export cached under the version's current `lastUpdated`. An export of the same version cached under an earlier timestamp isn't removed, and GET may still return it.

### Example
* Delete the export file for v2.2 of the CIEL source
```
DELETE /orgs/CIEL/sources/CIEL/v2.2/export/
```

### Response
* If the file exists, it is deleted:
```
Status: 204 No Content
```
* If the file does NOT exist:
```
Status: 404 Not Found
```
* If the user doesn't have administrative access to the repository:
```
Status: 403 Forbidden
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
