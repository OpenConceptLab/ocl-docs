# Sources

## Overview
The API exposes a representation of sources which are versioned repositories of concepts (or codes, terms, measures, metadata definitions) and mappings. A `Source` is used to create and edit concepts and mappings, whereas a `Collection` is used to build a set of concepts and mappings from one or more sources. A `Source` can be used to build a FHIR `CodeSystem` or a FHIR `ConceptMap` -- see [OCL FHIR Overview](https://docs.openconceptlab.org/en/latest/oclfhir/overview.html) for more information.

Examples of a `Source` include an OpenMRS concept dictionary (e.g. CIEL, AMPATH, PIH), a reference terminology or codeset (e.g. SNOMED CT, ICD-10), or indicator registry (e.g. WHO Indicator Registry). Custom dictionaries are also supported, which are useful for representing local or proprietary content or content that is still under development. A `Source` is owned by either a user or an organization.

OCL supports internal and external sources. An internal source is one whose concepts and mappings are being managed on OCL. Its metadata as well as concepts, classes, datatypes, mappings, maptypes, etc. may all be edited by a user with sufficient privileges. An external source acts as placeholder for mappings to concepts that are not stored in OCL. For example, the SNOMED CT terminology is not hosted on OCL, but the IHTSDO organization and SNOMED CT source do exist as a placeholder to map to SNOMED CT terms.

The public's access to a `Source` may be set to `Edit`, `View` or `None`. Typically sources are marked as `View`, which allows any authenticated user to view the content of the source but only users with additional permissions may edit the source. If a source is marked as `Edit`, then any authenticated OCL user may make changes to it, so this should be used only in rare situations. A private source is one that is marked as `None`-- only user's with explicitly shared access may search or perform actions on the concepts in a private source.

Example uses:
* `GET /orgs/WHO/sources/ICD-10-2010/` - Get an organization's source
* `GET /users/johndoe/sources/my-source/` - Get a user's source
* `GET /user/sources/my-source/` - Get the authenticated user's source

### Versioning of sources
* The current state of a source's metadata and all of its resources is referred to as its `HEAD`. All changes to a source's metadata and resources are made to the `HEAD` of the source, and prior source versions are not affected. The source `HEAD` is used when no source version is otherwise specified in a request. For example, `GET /user/sources/MySource/HEAD/` and `GET /user/sources/MySource/` will retrieve the `HEAD` source version, whereas `GET /user/sources/MySource/v1.0/` will retrieve the source version with an ID of "v1.0".
* Versions of a source are a frozen pointer to the state of the source's concepts, mappings, and metadata at a specific point in time, similar to "tags" in GitHub. The "frozen" data includes the source metadata (name, descriptions, etc.).
* Source versions can be marked as "released" or "retired" to indicate to users how the contents of a source version are intended to be used. Any number of source versions may be marked as "released".
* The `latest` source version is a magic keyword that automatically refers to the most recent released version of a source, e.g. `GET /user/sources/MySource/latest/`

## Get a single source
* Get a public source owned by an oragnization or user
```
GET /users/:user/sources/:source/
GET /orgs/:org/sources/:source/
GET /user/sources/:source/
```
* Parameters (only applicable when including child concepts or mappings in the response)
    * `includeConcepts` (optional) boolean - set to true to include the concepts owned by this source in a child attribute named `concepts`
    * `includeMappings` (optional) boolean - set to true to include the mappings owned by this source in a child attribute named `mappings`
    * `offset` (optional) integer - zero-based index of the concepts and mappings to return; default is 0; note that this is only applicable if `includeConcepts` or `includeMappings` is set to true
    * `updatedSince` (optional) datetime - ISO 8601 timestamp (e.g. `2011-11-16T14:26:15Z`) that filters the returned concepts and mappings to only those updated after the specified date/time; note that this is only applicable if `includeConcepts` or `includeMappings` is set to true
    * `includeRetired` (optional) boolean - set to true to include retired mappings or concepts in the response; note that this is only applicable if `includeConcepts` or `includeMappings` is set to true
    * `limit` (optional) numeric - set to the maximum number of concepts and mappings to return with the source; note that this is only applicable if `includeConcepts` or `includeMappings` is set to true

### Response
* Typical response -- with `includeMappings` and `includeConcepts` set to `false`
```
Status: 200 OK
```
```JSON
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
    "updated_by": "johndoe"
}
```

* If `includeConcepts` or `includeMappings` parameters are set to `true`, source details remain the same as above with the addition of `concepts` and/or `mappings` attributes that include JSON lists of the results.
```
Status: 200 OK
```
```
{
   "type": "Source",
   ...  # Source details included here

   "concepts": [
      ...      # List of concept results included here
   ],

   "mappings": [
      ...      # List of mapping results included here
   ]
}
```



## List all sources for specific user or organization
* List public sources owned by an organization or user
```
GET /users/:user/sources/
GET /orgs/:org/sources/
GET /user/sources/
```
* Notes
    * Private sources owned by the organization are only returned for users that are members of the organization
* Parameters
    * **q** (optional) string - Search criteria (search across: "name", "full_name" and "description")
    * **sortAsc/sortDesc** (optional) string - Sort results on one of the following fields: "best_match" (default), "last_update", "name"
    * **sourceType** (optional) string - Filter results to a given source type, e.g. "dictionary", "reference"
    * **locale** (optional) string - Filter results to those with a given locale in their supported_locales, e.g. "en", "fr"
    * **customValidationSchema** (optional) string - Filter results to a given validationSchema, e.g. "OpenMRS"

### Response
```
Status: 200 OK
```
```JSON
[
    {
        "short_code": "ICD-10-2010",
        "name": "ICD-10-WHO 2010",
        "url": "/orgs/WHO/sources/ICD-10-2010/",
        "owner": "WHO",
        "owner_type": "Organization",
        "owner_url": "/orgs/WHO/"
    }
]
```

## List all sources for all of a user's organizations
```
GET /users/:user/orgs/sources/
GET /user/orgs/sources/
```
* Notes
    * Private sources owned by the organization are only returned for users that are members of the organization
* Parameters
    * **q** (optional) string - Search criteria (search across: "name", "full_name" and "description")
    * **sortAsc/sortDesc** (optional) string - Sort results on one of the following fields: "best_match" (default), "last_update", "name"
    * **sourceType** (optional) string - Filter results to a given source type, e.g. "dictionary", "reference"
    * **locale** (optional) string - Filter results to those with a given locale in their supported_locales, e.g. "en", "fr"
    * **customValidationSchema** (optional) string - Filter results to a given validationSchema, e.g. "OpenMRS"

### Response
```
Status: 200 OK
```
```JSON
[
    {
        "short_code": "ICD-10-2010",
        "name": "ICD-10-WHO 2010",
        "url": "/orgs/WHO/sources/ICD-10-2010/",
        "owner": "WHO",
        "owner_type": "Organization",
        "owner_url": "/orgs/WHO/"
    }
]
```




## List all public sources
* Use the `/sources/` endpoint to list or search public sources across users and organizations
```
GET /sources/
```
* Parameters
    * **q** (optional) string - Search criteria (search across: "name", "full_name" and "description")
    * **sortAsc/sortDesc** (optional) string - Sort results on one of the following fields: "last_update" (default), "name"
    * **sourceType** (optional) string - Filter results to a given source type, e.g. "dictionary", "reference"
    * **locale** (optional) string - Filter results to those with a given locale in their supported_locales, e.g. "en", "fr"
    * **customValidationSchema** (optional) string - Filter results to a given validationSchema, e.g. "OpenMRS"

### Response
```
Status: 200 OK
```
```JSON
[
    {
        "short_code": "ICD-10-2010",
        "name": "ICD-10-WHO 2010",
        "url": "/orgs/WHO/sources/ICD-10-2010/",
        "owner": "WHO",
        "owner_type": "Organization",
        "owner_url": "/orgs/WHO/"
    }
]
```



## Create source
* Create new source owned by the authenticated user
```
POST /user/sources/
POST /orgs/:org/sources/
```
* Notes
    * "id" cannot be modified after the source has been created
    * The source "id" must be unique within a specific owner's sources and collections - meaning a source "id" may be reused by other users and organizations
    * An authenticated user must be an administrator of the organization to create a new source within that organization
* Input
    * **id** (required) string - same as short_code
    * **external_id** (optional) string - external ID used for import/export
    * **short_code** (required) string - short version of the source name (usually an acronym e.g. ICD-10 or LOINC) used to identify the source in the URL
    * **name** (required) string - Commonly used name for the source
    * **full_name** (optional) string - fully specified name of the source; automatically set to the value of "name" if blank
    * **source_type** (optional) string - "dictionary" (default), "reference", "externalDictionary"
    * **public_access** (optional) string - "View" (default), "Edit", "None"
    * **default_locale** (optional) string - 2-letter code for the default language of the source; default is "en"
    * **supported_locales** (required) string - comma-separated list of 2-letter language codes supported by the source (e.g. "en,es,sw"); default is "en"
    * **website** (optional) string - website for more information on the source
    * **description** (optional) string - description of the source
    * **custom_validation_schema** (optional) string - name of the custom validation schema
    * **extras** (optional) json dictionary - additional metadata for the resource
```JSON
{
    "id": "loinc2",
    "short_code": "loinc2",
    "name": "LOINC v2",
    "full_name": "Logical Observation Identifiers Names and Codes",
    "source_type": "Dictionary",
    "public_access": "View",
    "default_locale": "en",
    "supported_locales": "en",
    "website": "http://loinc.org/",
    "description": "A universal code system for identifying laboratory and clinical observations.",
    "extras": { "my_extra_field": "my_extra_value" }
}
```

### Response
* Status: 201 Created
* Location: http://api.openconceptlab.com/WHO/sources/ICD-10/
```JSON
{
    "type": "Source",
    "uuid": "8d492ee0-c2cc-11de-8d13-0010c6dffd0f",
    "id": "loinc2",
    "external_id": "",

    "short_code": "loinc2",
    "name": "LOINC v2",
    "full_name": "Logical Observation Identifiers Names and Codes",
    "source_type": "Dictionary",
    "public_access": "View",
    "default_locale": "en",
    "supported_locales": "en,fr",
    "website": "http://loinc.org/",
    "description": "A universal code system for identifying laboratory and clinical observations.",

    "extras": { "my_extra_field": "my_extra_value" },

    "owner": "Regenstrief",
    "owner_type": "Organization",
    "owner_url": "/orgs/Regenstrief/",

    "url": "/orgs/Regenstrief/sources/loinc2/",
    "versions_url": "/orgs/Regenstrief/sources/loinc2/versions/",
    "concepts_url": "/orgs/Regenstrief/sources/loinc2/concepts/",
    "mappings_url": "/orgs/Regenstrief/sources/loinc2/mappings/",

    "versions": 3,
    "active_concepts": 15000,
    "active_mappings": 3243,

    "created_on": "2008-01-14T04:33:35Z",
    "created_by": "johndoe",
    "updated_on": "2012-02-18T09:10:16Z",
    "updated_by": "johndoe"
}
```



## Edit source
* Partial update of a source owned by the authenticated user

```
PUT /user/sources/:source/
POST /orgs/:org/sources/:source/
```
* Notes
    * The "id" and "short_code" of the source cannot be updated after it has been created
    * The authenticated user must have admin access to the source to perform edits
* Input
    * **external_id** (optional) string - external ID used for import/export
    * **name** (optional) string - Commonly used name for the source
    * **full_name** (optional) string - fully specified name of the source; automatically set to the value of "name" if blank
    * **source_type** (optional) string - "dictionary" (default), "reference", "externalDictionary"
    * **public_access** (optional) string - "View" (default), "Edit", "None"
    * **default_locale** (optional) string - 2-letter code for the default language of the source; default is "en"
    * **supported_locales** (optional) string - comma-separated list of 2-letter language codes supported by the source (e.g. "en,es,sw"); default is "en"
    * **website** (optional) string - website for more information on the source
    * **description** (optional) string - description of the source
    * **extras** (optional) json dictionary - additional metadata for the resource
```JSON
{
    "name": "LOINC v2",
    "full_name": "Logical Observation Identifiers Names and Codes",
    "source_type": "Dictionary",
    "pulic_access": "View",
    "default_locale": "en",
    "supported_locales": "en,es",
    "website": "http://loinc.org/",
    "description": "A universal code system for identifying laboratory and clinical observations.",
    "extras": { "my_extra_field": "my_extra_value" }
}
```

### Response
* Status: 200 OK
* Returns the updated JSON representation of the source - same as `GET /user/sources/:source/`



## Deactivate a source
* Deactivate the specified source owned by the authenticated user
```
DELETE /user/sources/:source/
DELETE /orgs/:org/sources/:source/
```
* Notes
    * This only **deactivates** the source using an internal flag. This effectively hides the source and any data or metadata it contains from OCL. Note that this may not necessarily delete the contents from the system.
    * Authentication information for a user with administrative access to the source must be passed with the request. E.g. `curl -u "username" "/users/johndoe/"`

### Response
* Status: 204 No Content



## Get single version of a source
* Get a single version of a source, where `:version` is "latest" or a source version ID
```
GET /user/sources/:source/:version/
GET /users/:user/sources/:source/:version/
GET /orgs/:org/sources/:source/:version/
```
* Notes
    * Use magic keyword "latest" to get the most recently created **released** version of a source

### Response
* Status: 200 OK
```JSON
{
    "type": "Source Version",
    "id": "2.1",
    "external_id": "",
    "description": "50 new codes",
    "released": "false",

    "url": "/orgs/Regenstrief/sources/loinc2/2.2/",
    "source_url": "/orgs/Regenstrief/sources/loinc2/",
    "parent_version_url": "/orgs/Regenstrief/sources/loinc2/2.0/",      
    "previous_version_url": "/orgs/Regenstrief/sources/loinc2/2.1/",
    "root_version_url": "/orgs/Regenstrief/sources/loinc2/1.0/",

    "extras": {},

    "created_on": "2008-01-14T04:33:35Z",
    "created_by": "johndoe",
    "updated_on": "2010-02-14T04:33:35Z",
    "updated_by": "johndoe",

    "source": {
    }
}
```



## List children of a source version -- DEPRECATED
**DEPRECATED** This endpoint will be removed -- do not use
* Get the children of a specific version
```
GET /user/sources/:source/:version/children/
GET /users/:user/sources/:source/:version/children/
GET /orgs/:org/sources/:source/:version/children/
```

### Example
```
GET /orgs/Regenstrief/sources/loinc2/2.0/children/
```

### Response
* Status: 200 OK
```JSON
[
    {
        "id": "2.1",
        "released": "false",
        "url": "/orgs/Regenstrief/sources/loinc2/2.1/"
    },
    {
        "id": "2.2",
        "released": "false",
        "url": "/orgs/Regenstrief/sources/loinc2/2.2/"
    }
]
```



## List all versions of a source
* List all versions of a source
```
GET /user/sources/:source/versions/
GET /users/:user/sources/:source/versions/
GET /orgs/:org/sources/:source/versions/
```
* Parameters
    * `released` (optional) boolean - set to true to include only source versions with the `released` attribute set to `true`
    * `processing` (optional) boolean - set to true to include only source versions with the `_ocl_processing` attribute set to `true`

### Response
* Status: 200 OK
```JSON
[
    {
        "id": "2.1",
        "released": "false",
        "url": "/orgs/Regenstrief/sources/loinc2/2.1/"
    }
]
```



## Create new version of a source
* Create new version of a source
```
POST /user/sources/:source/versions/
POST /users/:user/sources/:source/versions/
POST /orgs/:org/sources/:source/versions/
```
* Notes
    * "id" cannot be changed after a source version is created
    * "parent_version_url" is deprecated and should not be used
    * "root_version_url" is set automatically by the API to the first user-generated source version. If the source version being created is the first source version, then "root_version_url" is set to itself.
    * Currently, "previous_version_url" must be set to the ID of the appropriate previous source version. In the future, the API will set this automatically.
* Inputs
    * **id** (required) string
    * **released** (optional) string - "true" or "false"
    * **description** (optional) string
    * **external_id** (optional) string
    * **extras** (optional) JSON dictionary
    * **previous_version** (optional) string            # In the future, this will be automatically populated by OCL
    * **parent_version** (optional) string              # Deprecated -- do not use - this will be removed
```JSON
{
    "id": "2.45",
    "released": "false",
    "description": "next version",
    "previous_version": "2.44"
}
```

### Response
* Status: 201 Created
* Location: http://api.openconceptlab.com/Regenstrief/collections/loinc2/2.45/
```JSON
{
    "type": "Source Version",
    "id": "2.45",
    "external_id": "",
    "released": "false",
    "description": "new version",

    "url": "/orgs/Regenstrief/sources/loinc2/2.45/",
    "source_url": "/orgs/Regenstrief/sources/loinc2/",
    "parent_version_url": "/orgs/Regenstrief/sources/loinc2/2.0/",          
    "previous_version_url": "/orgs/Regenstrief/sources/loinc2/2.44/",
    "root_version_url": "/orgs/Regenstrief/sources/loinc2/1.0/",

    "extras": {},

    "created_on": "2008-01-14T04:33:35Z",
    "created_by": "johndoe",
    "updated_on": "2008-01-14T04:33:35Z",
    "updated_by": "johndoe",

    "source": {
        
    }
}
```



## Edit a source version
* Edit a source version
```
POST /user/sources/:source/:version/
POST /users/:user/sources/:source/:version/
POST /orgs/:org/sources/:source/:version/
```
* Notes
    * "id" cannot be changed after a version is created
    * Editing of "previous_version" and "parent_version" is deprecated as these will be automatically set by the API
* Inputs
    * **released** (optional) string - "true" or "false"
    * **description** (optional) string
    * **external_id** (optional) string
    * **extras** (optional) JSON dictionary
    * **previous_version** (optional) string            # Editing of this field is deprecated and will be removed
    * **parent_version** (optional) string              # Editing of this field is deprecated and will be removed
```JSON
{
    "released": "true",
    "description": "officially released version"
}
```

### Response
* Status: 200 OK
```JSON
{
    "type": "Source Version",
    "id": "2.45",
    "external_id": "",
    "released": "true",
    "description": "officially released version",

    "url": "/orgs/Regenstrief/sources/loinc2/2.45/",
    "source_url": "/orgs/Regenstrief/sources/loinc2/",
    "parent_version_url": "/orgs/Regenstrief/sources/loinc2/2.0/",        
    "previous_version_url": "/orgs/Regenstrief/sources/loinc2/2.44/",
    "root_version_url": "/orgs/Regenstrief/sources/loinc2/1.0/",

    "extras": {},

    "created_on": "2008-01-14T04:33:35Z",
    "created_by": "johndoe",
    "updated_on": "2008-01-14T04:33:35Z",
    "updated_by": "johndoe",

    "source": {
        
    }
}
```



## Deactivate a source version
* Deactivate a source version
```
DELETE /user/sources/:source/:version/
DELETE /users/:user/sources/:source/:version/
DELETE /orgs/:org/sources/:source/:version/
```
* Notes
    * A deactivated version can no longer be accessed
    * Authentication information for a user with administrative access to the source must be passed with the request. E.g. `curl -u "username" "/users/johndoe/"`

### Response
* Status: 204 No Content


## Retrieve and clear processing flag on a source version
### Retrieve processing flag on a source version
```
GET /orgs/:org/sources/:source/:version/processing/
```

### Response
* Processing flag value: True or False

### Clear processing flag on a source version
```
POST /orgs/:org/sources/:source/:version/processing/
```

### Response
* Status: 200 OK


## Search and Filter Behavior
* Text Search (e.g. `q=criteria`) - NOTE: Plus-sign (+) indicates relative relevancy weight of the term
    * source.short_code (++++), source.name (++++), source.full_name (+++), source.description (+)
* Facets
    * **locale** - source.supported_locales
    * **sourceType** - source.source_type
    * **owner** - source.owner
    * **ownerType** - concept.owner_type
* Filters
    * ??
* Sort
    * **bestMatch** (default) - see search fields above
    * **name** (Asc/Desc) - source.name
    * **lastUpdate** (Asc/Desc) - source.updated_on
