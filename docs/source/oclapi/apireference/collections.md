# Collections

## Overview
The API exposes a representation of `collections`, which are versioned containers of references to `concepts` and `mappings`. Note that `sources` are used to actually author `concepts` and `mappings`, whereas `collections` are used to organize existing `concepts` and `mappings` into useful logical groupings, such as a diabetes value set or a Community Maternal-Child Starter Set. Collections reference concepts and mappings by combining one or more expressions that return concepts and mappings. Some expressions may be processed dynamically, meaning that the `concepts` and `mappings` in a collection may change automatically.

In the future, `collections` will be used to represent a variety of intensional and extensional value sets, where intensional value sets define a set of properties that are evaluated (or "expanded") to determine the collection members, and extensional value sets explicitly state each member of the value set. At this time, `collections` can be used to represent any type of extensional value set by adding individual concepts and mappings into a collection. Currently, `collections` support a limited number of intensional value sets, where the expression that is added cannot be dynamic, such as a source version or collection version.

### Expressions
* Expressions have been split into Phase 1 and Future Phases, where phase 1 includes expressions that result in either individual concepts and mappings or static lists of concepts and mappings (such as a source or collection version). Expressions follow the REST API syntax.
* Phase 1 support for expressions:
    * Expressions to add individual concepts:
        * Latest version of a concept: `/orgs/:org/sources/:source/concepts/:concept/`
        * Specific concept version: `/orgs/:org/sources/:source/concepts/:concept/:conceptVersion/`
    * Expressions to add individual mappings:
        * Latest version of a mapping: `/orgs/:org/sources/:source/mappings/:mapping/`
        * Specific mapping version: `/orgs/:org/sources/:source/mappings/:mapping/:mappingVersion/`
* Possible support for expressions in Future Phases:
    * Expressions to add individual concepts:
        * Concept from a specific source version: `/orgs/:org/sources/:source/:sourceVersion/concepts/:concept/`
    * Expressions to add individual mappings:
        * Mapping from a specific source version: `/orgs/:org/sources/:source/:sourceVersion/mappings/:mapping/`
    * Expressions to add concepts from a source or collection version:
        * All concepts from specific source version: `/orgs/:org/sources/:source/:sourceVersion/concepts/`
        * All concepts from specific collection version: `/orgs/:org/collections/:collection/:collectionVersion/concepts/`
    * Expressions to add mappings from a source or collection version:
        * All concepts from specific mapping version: `/orgs/:org/collections/:collection/:collectionVersion/mappings/`
        * All concepts from specific collection version: `/orgs/:org/collections/:collection/:collectionVersion/mappings/`
    * Expressions to add all mappings for a concept:
        * All direct mappings for a concept owned by the same source as the concept: `/orgs/:org/sources/:source/concepts/:concept/mappings/`
        * All direct and inverse mappings for a concept owned by the same source as the concept: `/orgs/:org/sources/:source/concepts/:concept/mappings/?includeInverseMappings=true`
    * Parameters or filters may be included to filter the results. For example:
        * All public concepts matching search criteria: `/concepts/?q=malaria`
        * All concepts from a single source matching search criteria: `/orgs/CIEL/sources/CIEL/concepts/?class=Drug`
    * Expressions to add concepts based on relationships
        * E.g. All concepts that are descendants of a concept
    * Expressions to add concepts from the top-level search endpoints
        * All public concepts that meet specific criteria: `/concepts/?q=:criteria`
        * All public direct mappings for a concept: `/mappings/?fromConcept=:concept`
    * Expressions to add all resources from a source or collection with a single expression:
        * All concepts and mappings from a source: `/orgs/:org/sources/:source/[:sourceVersion/]`
        * All concepts and mappings from a collection: `/orgs/:org/collections/:collection/[:collectionVersion/]`
    * Expressions to add concepts and mappings from the `HEAD` of a source or collection
        * All concepts from head of source: `/orgs/:org/sources/:source/concepts/`
        * All concepts from head of collection: `/orgs/:org/collections/:collection/concepts/`
        * All mappings from head of source: `/orgs/:org/collections/:collection/mappings/`
        * All mappings from head of collection: `/orgs/:org/collections/:collection/mappings/`

### Implementation Considerations
* Collections may contain concepts that share the same code, which means that to ensure unique identification of a concept, the owner and source must also be included. Ideally, the API can support use of only the concept ID if the ID is unique within the collection, and it would always support the fully specified form. Here are examples of using the fully specified owner, repository and ID to reference a concept within a collection:
        * `/orgs/CIEL/collections/DiabetesStarterSet/concepts/CIEL:CIEL:1234/`
        * `/orgs/CIEL/collections/DiabetesStarterSet/concepts/OCL:MaternalHealthCoreDataset:1234/`
* Mapping UUIDs are unique across all of OCL, so mappings can be uniquely identified the same as they are within a source:
    * `/orgs/CIEL/collections/DiabetesStarterSet/mappings/kdi03993fkie919u1/`

### Versioning of collections
* The current state of a collection's metadata and all of its referenced resources is referred to as the `HEAD`. All changes to a collection's metadata or references are made to the `HEAD`, and prior collection versions are not affected. The collection `HEAD` is used when no collection version is otherwise specified in a request. For example, `GET /user/collections/MyCollection/` will retrieve the `HEAD` whereas `GET /user/collections/MyCollection/v1.0/` will retrieve the collection version with an ID of "v1.0".
* At any  time, the current state of a collection may be saved by creating a new collection version (e.g. POST /orgs/:org/collections/:collection/versions/). Metadata and resource references from a specific version of a collection may be retrieved by explicitly stating a version number (e.g. GET /orgs/:org/collections/:collection/:version/ or GET /orgs/:org/collections/:collection/:version/concepts/).
* Collections versions can be marked as "released" or "retired" to indicate to users how the contents of a collection version are intended to be used. Any number of versions may be marked as "released".

### Future Considerations
* Collections should support multi-lingual names and descriptions in the future



## Get a single collection
* Get a collection owned by a user or organization, where an optional `:collectionVersion` is "latest" or a specific ID of a collection version. If `:collectionVersion` is not provided, `HEAD` is assumed.
```
GET /users/:user/collections/:collection/[:collectionVersion/]
GET /orgs/:org/collections/:collection/[:collectionVersion/]
GET /user/collections/:collection/[:collectionVersion/]
```
* Parameters (only applicable when including child concepts or mappings in the response)
    * `includeConcepts` (optional) boolean - set to true to include the concepts owned by this collection in a child attribute named `concepts`
    * `includeMappings` (optional) boolean - set to true to include the mappings owned by this collection in a child attribute named `mappings`
    * `includeReferences` (optional) boolean - set to true to include the references owned by this collection in a child attribute named `references`
    * `offset` (optional) integer - zero-based index of the concepts and mappings to return; default is 0; note that this is only applicable if `includeConcepts` or `includeMappings` is set to true
    * `updatedSince` (optional) datetime - ISO 8601 timestamp (e.g. `2011-11-16T14:26:15Z`) that filters the returned concepts and mappings to only those updated after the specified date/time; note that this is only applicable if `includeConcepts` or `includeMappings` is set to true
    * `includeRetired` (optional) boolean - set to true to include retired mappings or concepts in the response; note that this is only applicable if `includeConcepts` or `includeMappings` is set to true
    * `limit` (optional) numeric - set to the maximum number of concepts and mappings to return with the source; note that this is only applicable if `includeConcepts` or `includeMappings` is set to true

### Response
* Typical response -- with `includeMappings`, `includeConcepts` and `includeReferences` set to `false`
```
Status: 200 OK
```
```JSON
{
    "type": "Collection",
    "uuid": "8d492ee0-c2cc-11de-8d13-0010c6dffd0f",
    "id": "Community-MCH",
    "external_id": "",

    "short_code": "Community-MCH",
    "name": "Community-MCH Core Dataset",
    "full_name": "Community Maternal-Child Health Core Dataset",
    "collection_type": "Core Dataset",
    "public_access": "View",
    "supported_locales": "en,es",
    "website": "",
    "description": "",
    "active_concepts": 20,
    "active_mappings": 26,

    "extras": {},

    "owner": "OCL",
    "owner_type": "Organization",
    "owner_url": "/orgs/OCL/",

    "url": "/orgs/OCL/collections/Community-MCH/",
    "versions_url": "/orgs/OCL/collections/Community-MCH/versions/",
    "concepts_url": "/orgs/OCL/collections/Community-MCH/concepts/",
    "mappings_url": "/orgs/OCL/collections/Community-MCH/mappings/",

    "versions": 4,

    "created_on": "2008-01-14T04:33:35Z",
    "created_by": "johndoe",
    "updated_on": "2008-02-18T09:10:16Z",
    "updated_by": "johndoe"
}
```

* If `includeConcepts`, `includeMappings` or `includeReferences` parameters are set to `true`, collection details remain the same as above with the addition of `concepts`, `mappings`, and/or `references` attributes that include JSON lists of the results.
```
Status: 200 OK
```
```
{
   "type": "Collection",
   ...  # Collection details included here

   "concepts": [
      ...      # List of concept results included here
   ],

   "mappings": [
      ...      # List of mapping results included here
   ],

   "references": [
      ...      # List of references in the collection
   ]
}
```



## List all collections for a specific user or organization
* List collections owned by a user or organization
```
GET /users/:user/collections/
GET /orgs/:org/collections/
GET /user/collections/
```
* Parameters
    * **q** (optional) string - Search criteria (searches across: "name", "full_name" and "description")
    * **sortAsc/sortDesc** (optional) string - Sort the results along one of these fields: "bestMatch" (default), "name", "last_update"
    * **collection_type** (optional) string - Filter results to a particular collection type, e.g. "Subset", "Value Set". Known issue: This type cannot have spaces in it.
    * **locale** (optional) string - Filter results to the ones that include a particular locale in their supported locales, e.g. "en", "fr".
    * **contains** (optional) string - Filter results to collections that contain a referenced to the specified relative URL for a concept version or mapping version, e.g. `/orgs/MOH/sources/HMIS-Indicators/concepts/HIV01-01/5a120687f7dccb0064ee8d2f/`
    * **customValidationSchema** (optional) string - Filter results to a given validationSchema, e.g. "OpenMRS"

### Response
```
Status: 200 OK
```
```JSON
[
    {
        "id": "Community-MCH",
        "name": "Community-MCH Core Dataset",
        "url": "/orgs/OCL/collections/Community-MCH/",
        "owner": "OCL",
        "owner_type": "Organization",
        "owner_url": "/orgs/OCL/"
    }
]
```


## List all collections for all of a user's organizations
```
GET /users/:user/orgs/collections/
GET /user/orgs/collections/
```
* Notes
    * Private collections owned by the organization are only returned for users that are members of the organization
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
        "url": "/orgs/WHO/collections/ICD-10-2010/",
        "owner": "WHO",
        "owner_type": "Organization",
        "owner_url": "/orgs/WHO/"
    }
]
```



## List all public collections
* Use the `/collections/` endpoint to list or search public collections across users and organizations
```
GET /collections/
```
* Parameters
    * **q** (optional) string - Search criteria (searches across: "name", "full_name" and "description")
    * **sortAsc/sortDesc** (optional) string - Sort the results along one of these fields: "name", "last_update" (default)
    * **collection_type** (optional) string - Filter results to a particular collection type, e.g. "Subset", "Value Set"
    * **locale** (optional) string - Filter results down to the ones that include a particular locale in their supported locales, e.g. "en", "fr".
    * **contains** (optional) string - Filter results to collections that contain a referenced to the specified relative URL for a concept version or mapping version, e.g. `/orgs/MOH/sources/HMIS-Indicators/concepts/HIV01-01/5a120687f7dccb0064ee8d2f/`
    * **customValidationSchema** (optional) string - Filter results to a given validationSchema, e.g. "OpenMRS"

### Response
```
Status: 200 OK
```
```JSON
[
    {
        "id": "Community-MCH",
        "name": "Community-MCH Core Dataset",
        "url": "/orgs/OCL/collections/Community-MCH/",
        "owner": "OCL",
        "owner_type": "Organization",
        "owner_url": "/orgs/OCL/"
    }
]
```



## Create collection
* Create new collection owned by the authenticated user or an organization
```
POST /user/collections/
POST /orgs/:org/collections/
```
* Notes
    * "id" is automatically set based on the value of "short_code" and these cannot be modified after the collection has been created
    * The collection "id" must be unique within a specific owner's sources and collections - meaning a collection "id" may be reused by other users and organizations
    * An authenticated user must be an administrator of the organization to create a new source within that organization
* Input
    * **short_code** (required) string - mnemonic used to identify the collection in the URL (usually an acronym e.g. Community-MCH)
    * **external_id** (optional) string - external ID used for import/export
    * **name** (required) string - Commonly used name for the collection
    * **full_name** (optional) string - fully specified name of the collection; automatically set to the value of "name" if blank
    * **collection_type** (optional) string - Collection type descriptor
    * **public_access** (optional) string - "View" (default), "Edit", "None"
    * **preferred_source** (optional) string - Preferred Collection Source
    * **supported_locales** (optional) string - comma-separated list of 2-letter language codes supported by the collection (e.g. "en,es,sw"); default is "en"
    * **website** (optional) string - website for more information on the collection
    * **description** (optional) string - description of the collection
    * **extras** (optional) json dictionary - additional metadata for the resource
```JSON
{
    "short_code": "Community-MCH",
    "name": "Community-MCH Core Dataset",
    "full_name": "Community Maternal-Child Health Core Dataset",
    "collection_type": "Core Dataset",
    "preferred_source": "CIEL",
    "public_access": "View",
    "supported_locales": "en,es"
}
```

### Response
* Status: 201 Created
* Location: http://api.openconceptlab.com/OCL/collections/Community-MCH/
```JSON
{
    "type": "Collection",
    "uuid": "8d492ee0-c2cc-11de-8d13-0010c6dffd0f",
    "id": "Community-MCH",
    "external_id": "",

    "short_code": "Community-MCH",
    "name": "Community-MCH Core Dataset",
    "full_name": "Community Maternal-Child Health Core Dataset",
    "collection_type": "Core Dataset",
    "public_access": "View",
    "supported_locales": "en,es",
    "website": "",
    "description": "",
    "preferred_source": "CIEL",

    "extras": {},

    "owner": "OCL",
    "owner_type": "Organization",
    "owner_url": "/orgs/OCL/",

    "url": "/orgs/OCL/collections/Community-MCH/",
    "versions_url": "/orgs/OCL/collections/Community-MCH/versions/",
    "concepts_url": "/orgs/OCL/collections/Community-MCH/concepts/",
    "mappings_url": "/orgs/OCL/collections/Community-MCH/mappings/",

    "versions": 1,

    "created_on": "2008-01-14T04:33:35Z",
    "created_by": "johndoe",
    "updated_on": "2008-01-14T04:33:35Z",
    "updated_by": "johndoe"
}
```



## Edit collection
* Partial update of a collection owned by the authenticated user or an organization
```
POST /user/collections/:collection/
POST /orgs/:org/collections/:collection/
```
* Notes
    * The "id" and "short_code" of the collection cannot be updated after it has been created
* Input
    * **external_id** (optional) string - external ID used for import/export
    * **name** (optional) string - Commonly used name for the collection
    * **full_name** (optional) string - fully specified name of the collection; automatically set to the value of "name" if blank
    * **collection_type** (optional) string - Collection type descriptor
    * **public_access** (optional) string - "View" (default), "Edit", "None"
    * **supported_locales** (optional) string - comma-separated list of 2-letter language codes supported by the collection (e.g. "en,es,sw"); default is "en"
    * **website** (optional) string - website for more information on the collection
    * **description** (optional) string - description of the collection
    * **extras** (optional) json dictionary - additional metadata for the resource
```JSON
{
    "name": "Community-MCH Core Dataset",
    "full_name": "Community Maternal-Child Health Core Dataset",
    "public_access": "View",
    "supported_locales": "en,es"
}
```

### Response
* Status: 200 OK
* Returns the detailed JSON representation of the updated collection - same as `GET /user/collections/:collection/`



## Deactivate a collection
* Deactivate the specified collection owned by the authenticated user or by an organization
```
DELETE /user/collections/:collection/
DELETE /orgs/:org/collections/:collection/
```
* Notes
    * This only **deactivates** the collection using an internal flag. This effectively hides the organization and any data or metadata it owns, but does not delete it from the system.
    * Authentication information for a user with administrative access to the collection must be passed with the request. E.g. `curl -u "username" "/users/johndoe/"`

### Response
* Status: 204 No Content



## Get single version of a collection
* Get a single version of a collection, where `:version` is a collection version ID or "latest"
```
GET /user/collections/:collection/versions/:version/
GET /users/:user/collections/:collection/versions/:version/
GET /orgs/:org/collections/:collection/versions/:version/
```
* Notes
    * "latest" is a magic keyword which automatically refers to the most recent **released** collection version

### Response
* Status: 200 OK
```JSON
{
    "type": "Collection Version",
    "id": "1.2",
    "external_id": "",
    "released": "false",
    "description": "Released a new iteration",

    "url": "/orgs/OCL/collections/Community-MCH/1.2/",
    "collection_url": "/orgs/OCL/collections/Community-MCH/",
    "previous_version_url": "/orgs/OCL/collections/Community-MCH/1.1/",
    "root_version_url": "/orgs/OCL/collections/Community-MCH/1.0/",

    "extras": {},

    "created_on": "2008-01-14T04:33:35Z",
    "created_by": "johndoe",
    "updated_on": "2010-02-14T04:33:35Z",
    "updated_by": "johndoe",

    "collection": {
       
    }
}
```



## List all versions of a collection
* List all versions of a collection
```
GET /user/collections/:collection/versions/
GET /users/:user/collections/:collection/versions/
GET /orgs/:org/collections/:collection/versions/
```
* Parameters
    * `released` (optional) boolean - set to true to include only collection versions with the `released` attribute set to `true`
    * `processing` (optional) boolean - set to true to include only collection versions with the `_ocl_processing` attribute set to `true`

### Response
* Status: 200 OK
```JSON
[
    {
        "id": "1.1",
        "released": "false",
        "url": "/orgs/OCL/collections/Community-MCH/1.1/"
    }
]
```



## Create new version of a collection
* Create new version of a collection
```
POST /user/collections/:collection/versions/
POST /users/:user/collections/:collection/versions/
POST /orgs/:org/collections/:collection/versions/
```
* Notes
    * "id" cannot be changed after a collection version is created
    * "root_version_url" is set automatically by the API to the first user-generated collection version. If the collection version being created is the first collection version, then "root_version_url" is set to itself.
    * "previous_version_url" is set automatically by the API to the prior collection version, or it is set to an empty string if this is the first collection version.
* Inputs
    * **id** (required) string
    * **released** (optional) string - "true" or "false"
    * **description** (optional) string
    * **external_id** (optional) string
    * **extras** (optional) JSON dictionary
```JSON
{
    "id": "1.5",
    "released": "false",
    "description": "new version",
    "external_id": "v1.5",
    "extras": { "myattr": "myvalue" }
}
```

### Response
* Status: 201 Created
* Location: http://api.openconceptlab.com/OCL/collections/Community-MCH/2.5/
```JSON
{
    "type": "Collection Version",
    "id": "1.5",
    "external_id": "",
    "released": "false",
    "description": "new version",

    "url": "/orgs/OCL/collections/Community-MCH/1.5/",
    "collection_url": "/orgs/OCL/collections/Community-MCH/",
    "previous_version_url": "/orgs/OCL/collections/Community-MCH/1.4/",
    "root_version_url": "/orgs/OCL/collections/Community-MCH/1.0/",

    "extras": {},

    "created_on": "2008-01-14T04:33:35Z",
    "created_by": "johndoe",
    "updated_on": "2008-01-14T04:33:35Z",
    "updated_by": "johndoe",

    "collection": {
        
    }
}
```



## Edit a collection version
* Edit a collection version owned by a user or organization
```
POST /user/collections/:collection/versions/:version/
POST /users/:user/collections/:collection/versions/:version/
POST /orgs/:org/collections/:collection/versions/:version/
```
* Notes
    * "id" cannot be changed after a version is created
* Inputs
    * **released** (optional) string - "true" or "false"
    * **description** (optional) string
    * **external_id** (optional) string
    * **extras** (optional) JSON dictionary
```JSON
{
    "released": "true",
    "description": "updated version description"
}
```

### Response
* Status: 200 OK
```JSON
{
    "type": "Collection Version",
    "id": "1.5",
    "external_id": "",
    "released": "false",
    "description": "new version",

    "url": "/orgs/OCL/collections/Community-MCH/1.5/",
    "collection_url": "/orgs/OCL/collections/Community-MCH/",
    "previous_version_url": "/orgs/OCL/collections/Community-MCH/1.4/",
    "root_version_url": "/orgs/OCL/collections/Community-MCH/1.0/",

    "extras": {},

    "created_on": "2008-01-14T04:33:35Z",
    "created_by": "johndoe",
    "updated_on": "2008-01-14T04:33:35Z",
    "updated_by": "johndoe",

    "collection": {
        
    }
}
```



## Deactivate a collection version
* Deactivate a collection version owned by a user or organization
```
DELETE /user/collections/:collection/versions/:version/
DELETE /users/:user/collections/:collection/versions/:version/
DELETE /orgs/:org/collections/:collection/versions/:version/
```
* Notes
    * A deactivated version can no longer be accessed
    * Authentication information for a user with administrative access to the collection must be passed with the request. E.g. `curl -u "username" "/users/johndoe/"`

### Response
* Status: 204 No Content


## Retrieve and clear processing flag on a collection version
### Retrieve processing flag on a collection version
```
GET /orgs/:org/collections/:collection/:version/processing/
```

### Response
* Processing flag value: True or False

### Clear processing flag on a collection version
```
POST /orgs/:org/collections/:collection/:version/processing/
```

### Response
* Status: 200 OK


## List all references in a collection
* List all references in a collection
```
GET /orgs/:org/collections/:collection/[:collectionVersion/]references/
GET /users/:user/collections/:collection/[:collectionVersion/]references/
GET /user/collections/:collection/[:collectionVersion/]references/
```
* Notes
    * "expression" may be any API request that results in a list of concepts or mappings

### Example
```
GET /orgs/KenyaMOH/collections/KenyaEMR/v1.1/references/
```
### Response
* Status: 200 OK
```JSON
[
    {
        "expression": "/orgs/CIEL/sources/CIEL/concepts/"
    },
    {
        "expression": "/orgs/CIEL/sources/CIEL/mappings/"
    },
    {
        "expression": "/orgs/KenyaMOH/sources/LocalConcepts/concepts/123/"
    },
    {
        "expression": "/orgs/KenyaMOH/sources/LocalConcepts/concepts/123/mappings/"
    }
]
```



## Add a reference to a collection
```
PUT /orgs/:org/collections/:collection/references/
PUT /users/:user/collections/:collection/references/
PUT /user/collections/:collection/references/
```
* Input
    * **expressions** (required) string - URLs of the references to add
    * **uri** (optional) string - URL of source/ collection from which to copy concepts
    * **concepts** (optional) lists of strings - URLs of the concepts to add or `*` when used with `uri` to add all concepts from the collection/ source
    * **mappings** (optional) lists of strings - URLs of the mappings to add or `*` when used with `uri` to add all mappings from the collection/ source
    * **search_term** (optional) - Used with `uri` to filter concepts/ mappings to add by search term
* Query Parameters
    * **cascade** (optional) (default=none) string - It takes `none`, `sourcemappings`, or `sourcetoconcepts` as a value
      * `none` (default): Do not cascade to any mappings or concepts
      * `sourecemappings`: Cascade to mappings in the same source, where the concept currently being processed is the `from-concept` of the mapping
      * `sourcetoconcepts`: Cascade to mappings and target concepts in the same source, where the concept currently being processed is the `from-concept` of the mapping
      * Note: Full documentation on cascade parameters can be found here: https://docs.openconceptlab.org/en/latest/oclapi/apireference/cascade.html

### Example
* `PUT /orgs/KenyaMOH/collections/KenyaEMR/references/?cascade=sourcemappings`
```JSON
{
    "data": {
        "expressions": ["/orgs/WHO/sources/ICD-10/concepts/A15.1/"]
    }
}
```

### Response
* Status: 200 OK
* Information, warning or error messages concerning addition of reference(s) are returned in a list
```JSON
[
  {
    "message": "Added the latest versions of concept to the collection. Future updates will not be added automatically.",
    "added": true,
    "expression": "/users/molgun/sources/HSTP-Indicatorsubhea/concepts/C1.1.1.2-gazmuk/58a19a1d46d2b100166d287f/"
  },
  {
    "message": [
      "Concept or Mapping reference name must be unique in a collection."
    ],
    "added": false,
    "expression": "/users/molgun/sources/HSTP-Indicatorsubhea/concepts/C1.1.1.2-gazmuk/"
  }
]
```


## Delete a reference from a collection
```
DELETE /orgs/:org/collections/:collection/references/
DELETE /users/:user/collections/:collection/references/
DELETE /user/collections/:collection/references/
```
* Input
    * **references** (required). One of;
        * string array - Version URLs of the references to remove
        * `*` - Delete all references from the collection
* Query Parameters
    * **cascade** (optional) (default=none) string - It takes `none`, `sourcemappings`, or `sourcetoconcepts` as a value.

### Example
* `DELETE /orgs/KenyaMOH/collections/KenyaEMR/references/`
```JSON
{
    "references": ["/orgs/WHO/sources/ICD-10/concepts/A15.1/589193e44ac5f4030b80bdb0/"]
}
```

### Response
* Status: 200 OK -- indicates that the expression was deleted
* Status: 404 Not Found -- indicates that the expression was not found in the collection



## Get single concept from a collection
* Get a single concept from a collection, where an optional `:collectionVersion` is "latest" or a collection version ID.
```
GET /orgs/:org/collections/:collection/[:collectionVersion/]concepts/:concept/
GET /users/:user/collections/:collection/[:collectionVersion/]concepts/:concept/
GET /user/collections/:collection/[:collectionVersion/]concepts/:concept/
```
* Notes
    * The results are identical to fetching a concept directly from a source (i.e. GET /user/sources/MySource/concepts/123/), with two exceptions:
        * The concept mappings (both direct and inverse) are pulled from the collection, not from the original source.
        * Additional metadata are added to the concept that point back to the collection from which it is referenced (e.g. concept_reference_url, collection_url, collection, collection_owner, collection_owner_type).
    * `:concept` is of the form `concept_id` (e.g. "A15.1") or `owner:source[sourceVersion]:concept_id` (e.g. "WHO:ICD-10:A15.1"). The simple form may only be used if the concept ID is unique within the collection, otherwise it results in an error.
    * The version of the concept returned is determined by the expression that added this concept to the collection

### Example Request
* Get a single concept from a collection with a simple concept ID
```
GET /orgs/OCL/collections/Community-MCH/concepts/1234/
```
* Get a single concept from a collection with a fully specified concept ID
```
GET /orgs/OCL/collections/Community-MCH/concepts/CIEL:CIEL:1234/
```

### Example
```
GET /orgs/OCL/collections/Community-MCH/concepts/WHO:ICD-10:A15.1/
```

### Response
Status: 200 OK
```JSON
{
    "concept_reference_url": "/orgs/OCL/collections/Community-MCH/concepts/WHO:ICD-10:A15.1/",
    "collection_url": "/orgs/OCL/collections/Community-MCH/",
    "collection": "Community-MCH",
    "collection_owner": "OCL",
    "collection_owner_type": "Organization",

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
    ],

    "extras": {
        "parent": "A15"
    },

    "source": "ICD-10",
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
```



## List concepts referenced in a collection
* List concepts referenced in a collection, where an optional `:collectionVersion` is "latest" or a collection version ID
```
GET /orgs/:org/collections/:collection/[:collectionVersion/]concepts/
GET /users/:user/collections/:collection/[:collectionVersion/]concepts/
GET /user/collections/:collection/[:collectionVersion/]concepts/
```
* Parameters
    * **verbose** (optional) string - default is false; set to true to return full concept details instead of the summary
    * **q** (optional) string - Search criteria
    * **sortAsc/sortDesc** (optional) string - sort results according to one of the following attributes: "bestMatch" (deafult), "lastUpdate", "name", "id", "datatype", "conceptClass"
    * **conceptClass** (optional) string - filter results to those within a particular concept class, e.g. "laboratory procedure"
    * **datatype** (optional) string
    * **locale** (optional) string - filter results to those with a name for the given locale, e.g. "en", "fr"
    * **includeRetired** (optional) integer - 1 or 0, default 0
    * **mapcode** (optional) string - e.g. IHTSDO:SNOMED-CT:123455
    * **includeMappings** (optional) string - default is "false" (even if "verbose" is set to "true"); set to "true" to return direct mappings contained in this collection
    * **includeInverseMappings** (optional) string - default is "false" (even if "verbose" is set to "true"); set to "true" to return inverse mappings contained in this collection
* Notes
    * The results are identical to listing concepts directly from a source (i.e. GET /user/sources/MySource/concepts/), with the exception that additional metadata are added that point back to the collection from which they are referenced (e.g. concept_reference_url). Note that "concept_reference_url" always uses the fully specified format (e.g. "WHO:ICD-10:A15.1").
    * If verbose is set to true, results are the same as fetching a single concept

### Examples
* List concepts in the latest version of a collection
```
GET /orgs/CIEL/collections/StarterSet/concepts/
```
* List concepts in a specific version of a collection
```
GET /orgs/CIEL/collections/StarterSet/v1.0/concepts/
```

### Example
```
GET /orgs/CIEL/collections/StarterSet/concepts/
```

### Response
Status: 200 OK
```JSON
[
    {
        "id": "A15.1",
        "concept_class": "Diagnosis",
        "datatype": "None",
        "retired": false,
        "source": "ICD-10",
        "owner": "WHO",
        "owner_type": "Organization",
        "owner_url": "/orgs/WHO/sources/ICD-10/",
        "display_name": "Tuberculosis of lung, confirmed by culture only",
        "display_locale": "en",
        "version": "abc345jf9fj",
        "url": "/orgs/WHO/sources/ICD-10/concepts/A15.1/",
        "version_url": "/orgs/WHO/sources/ICD-10/concepts/A15.1/abc345jf9fj/",
        "concept_reference_url": "/orgs/CIEL/collections/StarterSet/concepts/WHO:ICD-10:A15.1"
    }
]
```



## Concept subresource Requests
* **List concept names** -- just pull from the concept; in the future, collections may allow restricting which names are present -- is `:conceptVersion` dictated by the reference?
```
GET /orgs/:org/collections/:collection/[:collectionVersion/]concepts/:concept/names/
GET /users/:user/collections/:collection/[:collectionVersion/]concepts/:concept/names/
GET /user/collections/:collection/[:collectionVersion/]concepts/:concept/names/
```
* **Get a single concept name** -- just pull from the concept; in the future, collections may allow restricting which names are present -- is `:conceptVersion` dictated by the reference?
```
GET /orgs/:org/collections/:collection/[:collectionVersion/]concepts/:concept/names/:name/
GET /users/:user/collections/:collection/[:collectionVersion/]concepts/:concept/names/:name/
GET /user/collections/:collection/[:collectionVersion/]concepts/:concept/names/:name/
```
* **List concept descriptions** -- just pull from the concept; in the future, collections may allow restricting which descriptions are present -- is `:conceptVersion` dictated by the reference?
```
GET /orgs/:org/collections/:collection/[:collectionVersion/]concepts/:concept/descriptions/
GET /users/:user/collections/:collection/[:collectionVersion/]concepts/:concept/descriptions/
GET /user/collections/:collection/[:collectionVersion/]concepts/:concept/descriptions/
```
* **Get a single concept description** -- just pull from the concept; in the future, collections may allow restricting which descriptions are present -- is `:conceptVersion` dictated by the reference?
```
GET /orgs/:org/collections/:collection/[:collectionVersion/]concepts/:concept/descriptions/:description/
GET /users/:user/collections/:collection/[:collectionVersion/]concepts/:concept/descriptions/:description/
GET /user/collections/:collection/[:collectionVersion/]concepts/:concept/descriptions/:description/
```
* **List extras** -- just pull from the concept; in the future, collections may allow restricting which extras are present -- is `:conceptVersion` dictated by the reference?
```
GET /orgs/:org/collections/:collection/[:collectionVersion/]concepts/:concept/extras/
GET /users/:user/collections/:collection/[:collectionVersion/]concepts/:concept/extras/
GET /user/collections/:collection/[:collectionVersion/]concepts/:concept/extras/
```
* **Get a single extra** -- just pull from the concept; in the future, collections may allow restricting which extras are present -- is `:conceptVersion` dictated by the reference?
```
GET /orgs/:org/collections/:collection/[:collectionVersion/]concepts/:concept/extras/:field_name/
GET /users/:user/collections/:collection/[:collectionVersion/]concepts/:concept/extras/:field_name/
GET /user/collections/:collection/[:collectionVersion/]concepts/:concept/extras/:field_name/
```



## Get a single mapping from a collection
* Get a single mapping from a collection owned by a user or organization
```
GET /orgs/:org/collections/:collection/[:collectionVersion/]mappings/:mapping/
GET /users/:user/collections/:collection/[:collectionVersion/]mappings/:mapping/
GET /user/collections/:collection/[:collectionVersion/]mappings/:mapping/
```
* Notes
    * The results are identical to fetching a mapping directly from a source (i.e. GET /user/sources/MySource/mappings/8d492ee0-c2cc-11de-8d13-0010c6dffd0f/), with one exception:
        * Additional metadata are added to the mapping that point back to the collection from which it is referenced (e.g. mapping_reference_url, collection_url, collection, collection_owner, collection_owner_type).
    * `:mapping` is the same identifier that is used when requesting a mapping from a source, because the mapping "uuid" is guaranteed unique across OCL.

### Example
```
GET /orgs/MyOrg/collections/MyCollection/mappings/8d492ee0-c2cc-11de-8d13-0010c6dffd0f/
```

### Response
* Status: 200 OK
```JSON
{
    "mapping_reference_url": "/orgs/MyOrg/collections/MyCollection/mappings/WHO:ICD-10:A15.1/",
    "collection_url": "/orgs/MyOrg/collections/MyCollection/",
    "collection": "MyCollection",
    "collection_owner": "MyOrg",
    "collection_owner_type": "Organization",

    "type": "Mapping",
    "uuid": "8d492ee0-c2cc-11de-8d13-0010c6dffd0f",
    "external_id": "a9d93ffjjen9dnfekd9",
    "retired": "false",

    "map_type": "Same As",

    "from_source_owner": "Regenstrief",
    "from_source_owner_type": "Organization",
    "from_source_name": "loinc2",
    "from_concept_code": "32700-7",
    "from_concept_name": "Malarial Smear",
    "from_source_url": "/orgs/Regenstrief/sources/loinc2/",
    "from_concept_url": "/orgs/Regenstrief/sources/loinc2/concepts/32700-7/",

    "to_source_owner": "WHO",
    "to_source_owner_type": "Organization",
    "to_source_name": "ICPC-2",
    "to_concept_code": "A73",
    "to_concept_name": "Malaria",
    "to_source_url": "/orgs/WHO/sources/ICPC-2/",

    "source": "loinc2",
    "owner": "Regenstrief",
    "owner_type": "Organization",

    "url": "/orgs/Regenstrief/sources/loinc2/mappings/8d492ee0-c2cc-11de-8d13-0010c6dffd0f/",

    "extras": {},

    "created_on": "2008-01-14T04:33:35Z",
    "created_by": "johndoe",
    "updated_on": "2008-02-18T09:10:16Z",
    "updated_by": "johndoe"
}
```



## List mappings for a single concept that are contained in the collection
* List mappings for a single concept that are contained in the collection
```
GET /orgs/:org/collections/:collection/[:collectionVersion/]concepts/:concept/mappings/
GET /users/:user/collections/:collection/[:collectionVersion/]concepts/:concept/mappings/
GET /user/collections/:collection/[:collectionVersion/]concepts/:concept/mappings/
```
* Notes
    * Only mappings that are referenced in the collection are returned.
    * The results are identical to requesting mappings from a source (e.g. GET /orgs/CIEL/sources/CIEL/mappings/), with the exception that an additional metadata field is added, "mapping_reference_url", to point back to the mapping's URL within the collection.
    * If verbose is set to true, the results look the same as fetching a single concept (refer to documentation above).
* Parameters
    * **verbose** (optional) string - default is "false"; set to "true" to return full mapping details instead of the summary
    * **includeRetired** (optional) string - default - "false"; set to "true" to return retired mappings
    * **includeInverseMappings** (optional) string - default is "false"; set to "true" to return inverse mappings

### Example
```
GET /orgs/MyOrg/collections/MyCollection/concepts/Regenstrief:LOINC2:32700-7/mappings/
```

### Response
* Status: 200 OK
```JSON
[
    {
        "map_type": "Same As",
        "retired": "false",
        "source": "loinc2",
        "owner": "Regenstrief",
        "owner_type": "Organization",
        "from_concept_url": "/orgs/Regenstrief/sources/loinc2/concepts/32700-7/",
        "to_concept_url": "/orgs/WHO/sources/ICPC-2/concepts/A73/",
        "url": "/orgs/Regenstrief/sources/loinc2/mappings/8d492ee0-c2cc-11de-8d13-0010c6dffd0f/",
        "mapping_reference_url": "/orgs/MyOrg/collections/MyCollection/mappings/8d492ee0-c2cc-11de-8d13-0010c6dffd0f/"
    },
    {
        "map_type": "Narrower Than",
        "retired": "false",
        "source": "loinc2",
        "owner": "Regenstrief",
        "owner_type": "Organization",
        "from_concept_url": "/orgs/Regenstrief/sources/loinc2/concepts/32700-7/",
        "to_concept_code": "A73",
        "to_concept_name": "Malaria",
        "to_source_code": "ICPC-2",
        "url": "/orgs/Regenstrief/sources/loinc2/def3fe-c2cc-11de-8d13-asdf9393930/",
        "mapping_reference_url": "/orgs/MyOrg/collections/MyCollection/mappings/def3fe-c2cc-11de-8d13-asdf9393930/"
    }
]
```



## List mappings in a collection
* List mappings in a collection
```
GET /orgs/:org/collections/:collection/[:collectionVersion/]mappings/
GET /users/:user/collections/:collection/[:collectionVersion/]mappings/
GET /user/collections/:collection/[:collectionVersion/]mappings/
```
* Notes
    * The results are identical to requesting mappings from a source (e.g. GET /orgs/CIEL/sources/CIEL/mappings/), with the exception that an additional metadata field is added, "mapping_reference_url", to point back to the mapping's URL within the collection.
    * If verbose is set to true, the results look the same as fetching a single concept (refer to documentation above).
* Parameters
    * **verbose** (optional) string - default is "false"; set to "true" to return full mapping details instead of the summary
    * **includeRetired** (optional) string - default - "false"; set to "true" to return retired mappings
    * **includeInverseMappings** (optional) string - default is "false"; set to "true" to return inverse mappings

### Example
```
GET /orgs/MyOrg/collections/MyCollection/mappings/
```

### Response
* Status: 200 OK
```JSON
[
    {
        "map_type": "Same As",
        "retired": "false",
        "source": "loinc2",
        "owner": "Regenstrief",
        "owner_type": "Organization",
        "from_concept_url": "/orgs/Regenstrief/sources/loinc2/concepts/32700-7/",
        "to_concept_url": "/orgs/WHO/sources/ICPC-2/concepts/A73/",
        "url": "/orgs/Regenstrief/sources/loinc2/mappings/8d492ee0-c2cc-11de-8d13-0010c6dffd0f/",
        "mapping_reference_url": "/orgs/MyOrg/collections/MyCollection/mappings/8d492ee0-c2cc-11de-8d13-0010c6dffd0f/"
    },
    {
        "map_type": "Narrower Than",
        "retired": "false",
        "source": "loinc2",
        "owner": "Regenstrief",
        "owner_type": "Organization",
        "from_concept_url": "/orgs/Regenstrief/sources/loinc2/concepts/32700-7/",
        "to_concept_code": "A73",
        "to_concept_name": "Malaria",
        "to_source_code": "ICPC-2",
        "url": "/orgs/Regenstrief/sources/loinc2/def3fe-c2cc-11de-8d13-asdf9393930/",
        "mapping_reference_url": "/orgs/MyOrg/collections/MyCollection/mappings/def3fe-c2cc-11de-8d13-asdf9393930/"
    }
]
```



## Search and Filter Behavior
* Text Search (e.g. `q=criteria`) - NOTE: Plus-sign (+) indicates relative relevancy weight of the term
    * collection.short_code (++++), collection.name (++++), collection.full_name (+++), collection.description (+)
* Facets
    * **locale** - collection.supported_locales
    * **collection_type** - collection.source_type
    * **owner** - collection.owner
    * **ownerType** - collection.owner_type
* Filters
    * ??
* Sort
    * **bestMatch** (default) - see search fields above
    * **name** (Asc/Desc) - collection.name
    * **lastUpdate** (Asc/Desc) - collection.updated_on
