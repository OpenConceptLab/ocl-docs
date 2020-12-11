# OCL API

## User Documentation

### API Resources

#### Concepts

The API exposes a representation of `concepts`. Concepts are stored in [[Sources]], such as CIEL, ICD-10, or WHO-IMR. [[Collections]] provide versioned containers of references to concepts across sources.

Concepts have 3 fields that act as sub-resources: `names`, `descriptions`, and `extras`. This allows names, descriptions, and extra metadata to be referenced, viewed, edited, or deleted individually rather than as a whole. These fields are returned with the concept when requesting the full concept details (e.g. `GET /orgs/WHO/sources/ICD-10-2010/concepts/A10.9/`)

##### Concept Shorthand
Concepts are sometimes referred to in shorthand, rather than their fully specified URL:
```
# Get the latest version of a concept
org:source:concept

# Get the concept from a specific version of the source
org:source[sourceVersion]:concept

# Get a specific version of a concept
org:source:concept[conceptVersion]
```
For example, each set points to the same three concepts:
* Fully specified:
    * `/orgs/WHO/sources/ICD-10-2010/concepts/A10.9/`
    * `/orgs/Regenstrief/sources/LOINC/2.43/concepts/32700-7/`
    * `/orgs/MySDO/sources/RainMED/concepts/2962/1d35/`
* Short-hand:
    * `WHO:ICD-10-2010:A10.9`
    * `Regenstrief:LOINC[2.43]:32700-7`
    * `MySDO:RainMED:2962[1d35]`

##### Versioning of concepts
All changes to concepts are tracked internally. The latest version of a concept is retrieved if no version identifier (for the source or concept) is specified. If a source version identifier is specified, then the version of the concept at the time the source version was created is used. Alternatively, specific versions of concepts may also be retrieved.

##### Future Considerations
* Should the standard concept shorthand be changed to refer to the concept of the latest "released" source version instead of the latest version of a concept (regardless of source version)? For example, which url should this shorthand refer to:
    * /orgs/WHO/sources/ICD-10/concepts/A10.9/
    * /orgs/WHO/sources/ICD-10/latest/concepts/A10.9/
* Should "deleting a concept" return the "retired" concept?
* Would be great to allow additional annotation when concepts are retired to indicate to the user which concepts they should use instead. E.g. "retire_reason", "alternative_concepts" fields?



##### Get a single concept from a source
* Get a single concept from a source owned by an organization or user, where an optional `:sourceVersion` is "latest" or a source version ID
```
GET /orgs/:org/sources/:source/[:sourceVersion/]concepts/:concept/
GET /users/:user/sources/:source/[:sourceVersion/]concepts/:concept/
GET /user/sources/:source/[:sourceVersion/]concepts/:concept/
```
* Get a specific version of a concept
```
GET /orgs/:org/sources/:source/concepts/:concept/:conceptVersion/
GET /users/:user/sources/:source/concepts/:concept/:conceptVersion/
GET /user/sources/:source/concepts/:concept/:conceptVersion/
```
* Parameters
    * **includeMappings** (optional) string - default is "true", which returns direct `mappings` that are contained in the same source as the concept; set to "false" to not return `mappings`
    * **includeInverseMappings** (optional) string - default is "false"; set to "true" to return inverse mappings that are contained in the same source as the concept
* Notes
    * "names", "descriptions", "extras" and "mappings" (within the same source as the concept) are returned in full along with the concept details.
    * Use the `/mappings/` endpoint to retrieve `mappings` for a concept that are defined in other `sources`

##### Example Request
* Get the current version of a concept from the WHO:ICD-10-2010
```
GET /orgs/WHO/sources/ICD-10-2010/concepts/A15.1/?includeInverseMappings=true
```

##### Response
* Status: 200 OK
```JSON
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
            "name_type": None
        },
        {
            "type": "ConceptName",
            "uuid": "90jmcna4-lkdhf78",
            "external_id": "12345677",
            "name": "Tuberculose pulmonaire, confirmée par culture seulement",
            "locale": "fr",
            "locale_preferred": "true",
            "name_type": None
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
            "description_type": None
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
            "from_concept_name": "Tuberculosis of lung, confirmed by culture only",
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

    "inverse_mappings": [
        {
            "type": "Mapping",
            "uuid": "8jf8j-993JFKDKDFKJFIE",
            "external_id": "14356",
            "retired": "false",

            "map_type": "Narrower Than",

            "from_source_owner": "COLUMBIA",
            "from_source_owner_type": "Organization",
            "from_source_name": "CIEL",
            "from_concept_code": "113488",
            "from_concept_name": "Pulmonary Tuberculosis",
            "from_concept_url": "/orgs/IHTSDO/sources/CIEL/concepts/113488/",
            "from_source_url": "/orgs/IHTSDO/sources/CIEL/",

            "to_source_owner": "WHO",
            "to_source_owner": "Organization",
            "to_source_name": "ICD-10-2010",
            "to_concept_code": "A15.1",
            "to_concept_name": "Tuberculosis of lung, confirmed by culture only",
            "to_source_url": "/orgs/WHO/sources/ICD-10-2010/",
            "to_concept_url": "/orgs/WHO/sources/ICD-10-2010/concepts/A15.1/",

            "source": "CIEL",
            "owner": "COLUMBIA",
            "owner_type": "Organization",

            "url": "/orgs/WHO/sources/ICD-10-2010/mappings/8jf8j-993JFKDKDFKJFIE/",

            "created_on": "2008-01-14T04:33:35Z",
            "created_by": "johndoe",
            "updated_on": "2008-02-18T09:10:16Z",
            "updated_by": "johndoe"
        }
    ],

    "extras": {
        "parent": "A15"
    },

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
```



##### List concepts in a source
* List all concepts for an organization or user's source, where an optional `:sourceVersion` is "latest" or a source version ID  - NOTE: Retired concepts are excluded by default
```
GET /orgs/:org/sources/:source/[:sourceVersion/]concepts/
GET /users/:user/sources/:source/[:sourceVersion/]concepts/
GET /user/sources/:source/[:sourceVersion/]concepts/
```
* Parameters
    * **verbose** (optional) string - default is false; set to true to return full concept details instead of the summary
    * **q** (optional) string - search criteria (searches across "id", "names" and "descriptions")
    * **sortAsc/sortDesc** (optional) string - sort results according to one of the following attributes: "bestMatch" (default), "lastUpdate", "name", "id", "datatype", "conceptClass"
    * **conceptClass** (optional) string - filter results to those within a particular concept class, e.g. "laboratory procedure"
    * **datatype** (optional) string - filter results to those of a given datatype, e.g. "numeric"
    * **locale** (optional) string - filter results to those with a name for the given locale, e.g. "en", "fr"
    * **includeRetired** (optional) integer - 1 or 0, default 0
    * **includeMappings** (optional) string - default is "false" (even if "verbose" is set to "true"); set to "true" to return direct mappings contained in this source
    * **includeInverseMappings** (optional) string - default is "false" (even if "verbose" is set to "true"); set to "true" to return inverse mappings contained in this source

##### Examples
```
# Retrieve concepts matching free text search
GET /orgs/WHO/sources/ICD-10-2010/concepts/?q=tuberculosis

# Retrieve concepts with a concept_class of "Symptom" or "Diagnosis"
https://api.staging.openconceptlab.org/orgs/PEPFAR-Test7/sources/MER/concepts/?conceptClass="Symptom"+OR+"Diagnosis"
```

##### Response
* Status: 200 OK
```JSON
[
     {
        "id": "A15.1",
        "concept_class": "Diagnosis",
        "datatype": "None",
        "retired": false,
        "source": "ICD-10-2010",
        "owner": "WHO",
        "owner_type": "Organization",
        "owner_url": "/orgs/WHO/",
        "display_name": "Tuberculosis of lung, confirmed by culture only",
        "display_locale": "en",
        "version": "abc345jf9fj",
        "url": "/orgs/WHO/sources/ICD-10-2010/concepts/A15.1/",
        "version_url": "/orgs/WHO/sources/ICD-10-2010/concepts/A15.1/abc345jf9fj/"
    }
]
```



##### List concepts across public sources
* List concepts across public sources from all users and organizations - NOTE: Retired concepts are excluded by default; To list concepts from private sources, use the `/orgs/[org]/sources/[source]/concepts/` endpoint for the specific source.
```
GET /concepts/
```
* Parameters
    * **verbose** (optional) string - default is "false"; set to "true" to return full concept results instead of the concept summary
    * **q** (optional) string - search criteria
    * **sortAsc/sortDesc** (optional) string - sort results according to one of the following attributes: "lastUpdate" (default), "name", "datatype", "conceptClass"
    * **conceptClass** (optional) string - filter results to those within a particular concept class, e.g. "laboratory procedure"
    * **datatype** (optional) string - filter results to those of a given datatype, e.g. "numeric"
    * **locale** (optional) string - filter results to those with a name for the given locale, e.g. "en", "fr"
    * **includeRetired** (optional) integer - 1 or 0, default 0
    * **mapcode** (optional) string - e.g. IHTSDO:SNOMED-CT:123455
    * **source** (optional) string - ID of source that contains this
    * **user** (optional) string - filter by the username of the owner of the concept's source; cannot be used with org
    * **org** (optional) string - filter by the organization name of the owner of the concept's source; cannot be used with user
    * **includeMappings** (optional) string - default is "false" (even if "verbose" is set to "true"); set to "true" to return mappings
    * **includeInverseMappings** (optional) string - default is "false" (even if "verbose" is set to "true"); set to "true" to return inverse mappings

##### Example
```
GET /concepts/?q=tuberculosis
```

##### Response
* Status: 200 OK
```JSON
[
    {
        "id": "A15.1",
        "concept_class": "Diagnosis",
        "datatype": "None",
        "retired": false,
        "source": "ICD-10-2010",
        "owner": "WHO",
        "owner_type": "Organization",
        "owner_url": "/orgs/WHO/",
        "display_name": "Tuberculosis of lung, confirmed by culture only",
        "display_locale": "en",
        "version": "abc345jf9fj",
        "url": "/orgs/WHO/sources/ICD-10-2010/concepts/A15.1/",
        "version_url": "/orgs/WHO/sources/ICD-10-2010/concepts/A15.1/abc345jf9fj/"
    }
]
```



##### Create new concept
* Create a new concept in a source
```
POST /orgs/:org/sources/:source/concepts/
POST /users/:user/sources/:source/concepts/
POST /user/sources/:source/concepts/
```
* Input
    * **id** (required) string - unique identifier for concept, e.g. 145939019, A15.1, etc.
    * **retired** (optional) boolean - default is false; set to true to create a retired concept
    * **external_id** (optional) string - optional UUID from an external source
    * **concept_class** (required) string - classification of the concept (e.g. "Diagnosis", "Procedure", etc.)
    * **datatype** (optional) string - datatype for the concept (e.g. "Numeric", "String", "Coded")
    * **names** (required) list - at least one name is required
        * **name** (required) string - concept name
        * **external_id** (optional) string - optional UUID from an external source
        * **locale** (required) string - 2-character language code, e.g. "en", "es"
        * **locale_preferred** (optional) - "true" or "false", default is "false",
        * **name_type** (optional) - additional name descriptor, such as those used in SNOMED CT
    * **descriptions** (optional) list
        * **description** (required)
        * **external_id** (optional) string - optional UUID from an external source
        * **locale** (required)
        * **locale_preferred** (optional) - default is "false"
        * **description_type** (optional) - additional descriptor, such as those used in SNOMED CT
    * **extras** (optional) JSON dictionary - additional metadata for the resource
* Notes
    * At least one name must be submitted when the concept is created in order to set the "display_name" and "display_locale" fields
    * "descriptions", "extras", and additional "names" may be submitted with the concept details, in addition to being created after the fact by posting to the sub-resource (e.g. POST /.../concepts/12845003/names/)

##### Example
* `POST /orgs/IHTSDO/sources/SNOMED-CT/concepts/`
```JSON
{
    "id": "12845003",
    "external_id": "12845003",
    "concept_class": "Laboratory Procedure",
    "datatype": "N/A",
    "names": [
        {
            "name": "Malaria smear",
            "external_id": "14",
            "locale": "en",
            "locale_preferred": "true",
            "name_type": "Designated Preferred Name"
        },
        {
            "name": "Malaria smear (procedure)",
            "external_id": "176",
            "locale": "en",
            "name_type": "Full Form of Descriptor"
        }
    ],

    "extras": {
        "UMLS_CUI": "C0200703",
        "ISPRIMITIVE": "1"
    }
}
```

##### Response
* Status: 201 Created
* Location: http://api.openconceptlab.com/orgs/IHTSDO/sources/SNOMED-CT/concepts/12845003/
```JSON
{
    "type": "Concept",
    "uuid": "8d492ee0-c2cc-11de-8d13-0010c6dffd02",
    "id": "12845003",
    "external_id": "12845003",

    "concept_class": "Laboratory Procedure",
    "retired": false,

    "names": [
        {
            "type": "ConceptName",
            "uuid": "akdiejf93jf939f9",
            "external_id": "14",
            "name": "Malaria smear",
            "locale": "en",
            "locale_preferred": "true",
            "name_type": "Designated Preferred Name"
        },
        {
            "type": "ConceptName",
            "uuid": "akdiejf93jf939f9",
            "external_id": "176",
            "name": "Malaria smear (procedure)",
            "locale": "en",
            "name_type": "Full Form of Descriptor"
        }
    ],

    "source": "SNOMED-CT",
    "owner": "IHTSDO",
    "owner_type": "Organization",

    "version": "73jifjibL83",

    "url": "/orgs/IHTSDO/sources/SNOMED-CT/concepts/12845003/",
    "version_url": "/orgs/IHTSDO/sources/SNOMED-CT/concepts/12845003/73jifjibL83/",
    "source_url": "/orgs/IHTSDO/sources/SNOMED-CT/",
    "owner_url": "/orgs/IHTSDO/",
    "mappings_url": "/orgs/IHTSDO/sources/SNOMED-CT/concepts/12845003/mappings/",

    "versions": 1,

    "created_on": "2008-01-14T04:33:35Z",
    "created_by": "johndoe",
    "updated_on": "2008-01-14T04:33:35Z",
    "updated_by": "johndoe",

    "extras": {
        "UMLS_CUI": "C0200703",
        "ISPRIMITIVE": "1"
    }
}
```



##### Edit concept
* Edit a concept - note that this creates a new version of the concept
```
POST /orgs/:org/sources/:source/concepts/:concept/
POST /users/:user/sources/:source/concepts/:concept/
PUT /user/sources/:source/concepts/:concept/
```
* Input
    * **external_id** (optional) string - optional UUID from an external source
    * **concept_class** (optional) string - classification of the concept (e.g. Diagnosis, Procedure, etc.)
    * **datatype** (optional) string - datatype for the concept (e.g. Numeric, String, Coded)
    * **names** (optional)
        * **name** (optional) string -
        * **external_id** (optional) string - optional UUID from an external source
        * **locale** (optional) string - 2-character language code, e.g. "en", "es"
        * **locale_preferred** (optional) - "true" or "false", default is "false",
        * **name_type** (optional) - additional name descriptor, such as those used in SNOMED CT
    * **descriptions** (optional)
        * **description** (optional)
        * **external_id** (optional) string - optional UUID from an external source
        * **locale** (optional)
        * **locale_preferred** (optional) - default is "false"
        * **description_type** (optional) - additional description descriptor, such as those used in SNOMED CT
    * **update_comment** (optional) string - text describing the reason for the update; this is stored in the "concept_version" resource
* Notes
    * "names", "descriptions", and "extras" may be submitted with the concept details to update, but note that this approach will replace all of the names, descriptions, or extras with the newly submitted values. Use the sub-resources to create, edit, or delete individual values (e.g. POST /.../concepts/12845003/names/19jfjf9j3j/)

##### Response
* Status: 200 OK
* Returns the full JSON representation of the updated concept (same as above)



##### Retire concept
* Retire a concept
```
DELETE /user/sources/:source/concepts/:concept/
DELETE /users/:user/sources/:source/concepts/:concept/
DELETE /orgs/:org/sources/:source/concepts/:concept/
```
* Parameters
    * **update_comment** (optional) string - text describing the reason for retiring the concept; this field is saved to the concept version

To be implemented
***
* **purge** (optional) string - default is "false"; set to "true" to actually delete the concept from all source versions
* Notes
    * DELETE does not actually delete the concept unless "purge" is set to "true"; a retired concept is a new version of the concept with "retired" set to true, so that by default it is not returned in searches unless specifically requested. The "retired" status also indicates to users that the concept should not be used.
***


##### Response
* Status: 204 No Content



##### List versions of a concept
* List all versions of a specific concept
```
GET /user/sources/:source/concepts/:concept/versions/
GET /users/:user/sources/:source/concepts/:concept/versions/
GET /orgs/:org/sources/:source/concepts/:concept/versions/
```
* Notes
    * "is_latest_concept_version" is set to `true` if the version is the latest version of the concept
    * "is_root_concept_version" is set to `true` if the version is the initial submission (the "root") of the concept
    * "update_comment" is user-generated text describing the reason for the new version, similar to a GitHub commit comment
* Parameters
    * **verbose** (optional) string - set to true to include the full concept details along with the version metadata

##### Response
* If verbose is "false" (the default), then only the version metadata is returned:
```
Request: GET /orgs/WHO/sources/ICD-10/concepts/A15.1/versions/
Status: 200 OK
```
```JSON
[
    {
        "version": "a93-3j8083-d993mf",
        "previous_version": "abc345jf9fj",
        "url": "/orgs/WHO/sources/ICD-10/concepts/A15.1/a93-3j8083-d993mf/",
        "previous_version_url": "/orgs/WHO/sources/ICD-10/concepts/A15.1/abc345jf9fj/",
        "version_created_on": "2009-01-14T04:33:35Z",
        "version_created_by": "johndoe",
        "is_latest_concept_version": "true",
        "update_comment": "Updated the descriptions"
    },
    {
        "version": "abc345jf9fj",
        "is_root_concept_version": "true",
        "url": "/orgs/WHO/sources/ICD-10/concepts/A15.1/abc345jf9fj/",
        "version_created_on": "2008-01-14T04:33:35Z",
        "version_replaced_on": "2009-01-14T04:33:35Z",
        "version_created_by": "johndoe",
        "update_comment": "Initial submission"
    }
]
```
* If verbose is "true", then the full concept details are returned with version metadata:
```
Request: GET /orgs/WHO/sources/ICD-10/concepts/A15.1/versions/?verbose=true
Status: 200 OK
```
```JSON
[
    {
        "type": "ConceptVersion",
        "version": "a93-3j8083-d993mf",
        "previous_version": "abc345jf9fj",
        "url": "/orgs/WHO/sources/ICD-10/concepts/A15.1/a93-3j8083-d993mf/",
        "previous_version_url": "/orgs/WHO/sources/ICD-10/concepts/A15.1/abc345jf9fj/",
        "version_created_on": "2009-01-14T04:33:35Z",
        "version_created_by": "johndoe",
        "is_latest_concept_version": "true",
        "update_comment": "Updated the descriptions",
        "concept": {
            # Full concept details are included here, the same as if requesting:
            # GET /orgs/WHO/sources/ICD-10/concepts/A15.1/a93-3j8083-d993mf/
        }
    },
    {
        "type": "ConceptVersion",
        "version": "abc345jf9fj",
        "is_root_concept_version": "true",
        "url": "/orgs/WHO/sources/ICD-10/concepts/A15.1/abc345jf9fj/",
        "version_created_on": "2008-01-14T04:33:35Z",
        "version_replaced_on": "2009-01-14T04:33:35Z",
        "version_created_by": "johndoe",
        "update_comment": "Initial submission",
        "concept": {
            # Full concept details are included here, the same as if requesting:
            # GET /orgs/WHO/sources/ICD-10/concepts/A15.1/abc345jf9fj/
        }
    }
]
```



##### Get a single concept name
* Get a single concept name
```
GET /orgs/:org/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]names/:name_id/
GET /users/:user/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]names/:name_id/
GET /user/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]names/:name_id/
```

##### Example
* `GET /orgs/IHTSDO/sources/SNOMED-CT/concepts/12845003/names/akdiejf93jf939f9/`

##### Response
* Status: 200 OK
```JSON
{
    "type": "ConceptName",
    "uuid": "akdiejf93jf939f9",
    "external_id": "14",
    "name": "Malaria smear",
    "locale": "en",
    "locale_preferred": "true",
    "name_type": "Designated Preferred Name"
}
```



##### List concept names
* List concept names
```
GET /orgs/:org/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]names/
GET /users/:user/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]names/
GET /user/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]names/
```
* Parameters
    * **q** (optional) string - search criteria (based on the name field)
    * **locale** (optional) string - filter by locale
    * **name_type** (optional) - filter by name_type

##### Example
* `GET /orgs/IHTSDO/sources/SNOMED-CT/concepts/12845003/names/`

##### Response
* Status: 200 OK
```JSON
[
    {
        "type": "ConceptName",
        "uuid": "akdiejf93jf939f9",
        "external_id": "14",
        "name": "Malaria smear",
        "locale": "en",
        "locale_preferred": "true",
        "name_type": "Designated Preferred Name"
    },
    {
        "type": "ConceptName",
        "uuid": "a93-3j8083-d993mf",
        "external_id": "179",
        "name": "Malaria smear (procedure)",
        "locale": "en",
        "name_type": "Full Form of Descriptor"
    }
]
```


##### Create new concept name
* Create a new concept name
```
POST /orgs/:org/sources/:source/concepts/:concept/names/
POST /users/:user/sources/:source/concepts/:concept/names/
POST /user/sources/:source/concepts/:concept/names/
```
* Input
    * **name** (required) string - name
    * **external_id** (optional) string - optional UUID from an external source
    * **locale** (required) string - 2-character language code, e.g. "en", "es"
    * **locale_preferred** (optional) - "true" or "false", default is "false",
    * **name_type** (optional) - additional name descriptor, such as those used in SNOMED CT
* Notes
    * Concept names are part of a concept, therefore creating a concept name will generate a new version of the underlying concept

##### Example
* `POST /orgs/IHTSDO/sources/SNOMED-CT/concepts/12845003/names/`
```JSON
{
    "name": "Malaria smear",
    "external_id": "14",
    "locale": "en",
    "locale_preferred": "true",
    "name_type": "Designated Preferred Name"
}
```

##### Response
* Status: 201 Created
* Location: http://api.openconceptlab.com/orgs/IHTSDO/sources/SNOMED-CT/concepts/12845003/names/akdiejf93jf939f9/
```JSON
{
    "type": "ConceptName",
    "uuid": "akdiejf93jf939f9",
    "external_id": "14",
    "name": "Malaria smear",
    "locale": "en",
    "locale_preferred": "true",
    "name_type": "Designated Preferred Name"
}
```



##### Edit concept name
* Edit concept name
```
POST /orgs/:org/sources/:source/concepts/:concept/names/:name_id/
POST /users/:user/sources/:source/concepts/:concept/names/:name_id/
POST /user/sources/:source/concepts/:concept/names/:name_id/
```
* Input
    * **name** (required) string - name
    * **external_id** (optional) string - optional UUID from an external source
    * **locale** (required) string - 2-character language code, e.g. "en", "es"
    * **locale_preferred** (optional) - "true" or "false", default is "false",
    * **name_type** (optional) - additional name descriptor, such as those used in SNOMED CT
* Notes
    * Concept names are part of a concept, therefore updating a concept name will generate a new version of the underlying concept

##### Example
* `POST /orgs/IHTSDO/sources/SNOMED-CT/concepts/12845003/names/akdiejf93jf939f9/`
```JSON
{
    "name": "Malaria smear and banana sundae"
}
```

##### Response
* Status: 200 OK
```JSON
{
    "type": "ConceptName",
    "uuid": "akdiejf93jf939f9",
    "external_id": "14",
    "name": "Malaria smear and banana sundae",
    "locale": "en",
    "locale_preferred": "true",
    "name_type": "Designated Preferred Name"
}
```



##### Delete concept name
* Delete concept name
```
DELETE /orgs/:org/sources/:source/concepts/:concept/names/:name_id/
DELETE /users/:user/sources/:source/concepts/:concept/names/:name_id/
DELETE /user/sources/:source/concepts/:concept/names/:name_id/
```
* Notes
    * Concept names are part of a concept, therefore deleting a concept name will generate a new version of the underlying concept

##### Example
* `DELETE /orgs/IHTSDO/sources/SNOMED-CT/concepts/12845003/names/akdiejf93jf939f9/`

##### Response
* Status: 204 No Content



##### Get a single concept description
* Get a single concept description
```
GET /orgs/:org/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]descriptions/:description_id/
GET /users/:user/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]descriptions/:description_id/
GET /user/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]descriptions/:description_id/
```

##### Example
* `GET /orgs/IHTSDO/sources/SNOMED-CT/concepts/12845003/descriptions/aY873Hbmkdi09jeh/`

##### Response
* Status: 200 OK
```JSON
{
    "type": "ConceptDescription",
    "uuid": "aY873Hbmkdi09jeh",
    "external_id": "9390",
    "description": "Tuberculous bronchiectasis, fibrosis of lung, pneumonia, pneumothorax, confirmed by sputum microscopy with culture only",
    "locale": "en",
    "locale_preferred": "true",
    "description_type": None
}
```



##### List concept descriptions
* List concept descriptions
```
GET /orgs/:org/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]descriptions/
GET /users/:user/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]descriptions/
GET /user/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]descriptions/
```
* Parameters
    * **q** (optional) string - search criteria (based on the description field)
    * **locale** (optional) string - filter by locale
    * **description_type** (optional) - filter by description_type

##### Example
* `GET /orgs/IHTSDO/sources/SNOMED-CT/concepts/12845003/descriptions/`

##### Response
* Status: 200 OK
```JSON
[
    {
        "type": "ConceptDescription",
        "uuid": "aY873Hbmkdi09jeh",
        "external_id": "9390",
        "description": "Tuberculous bronchiectasis, fibrosis of lung, pneumonia, pneumothorax, confirmed by sputum microscopy with culture only",
        "locale": "en",
        "locale_preferred": "true",
        "description_type": None
    },
    {
        "type": "ConceptDescription",
        "uuid": "99fhfhufeh-9ehAAB",
        "external_id": "393930",
        "description": "Another less relevant and more lengthy description here",
        "locale": "en",
        "locale_preferred": "false",
        "description_type": None
    }
]
```


##### Create new concept description
* Create a new concept description
```
POST /orgs/:org/sources/:source/concepts/:concept/descriptions/
POST /users/:user/sources/:source/concepts/:concept/descriptions/
POST /user/sources/:source/concepts/:concept/descriptions/
```
* Input
    * **description** (required) string - description
    * **external_id** (optional) string - optional UUID from an external source
    * **locale** (required) string - 2-character language code, e.g. "en", "es"
    * **locale_preferred** (optional) - "true" or "false", default is "false",
    * **description_type** (optional) - additional description descriptor, such as those used in SNOMED CT
* Notes
    * Concept descriptions are part of a concept, therefore creating a concept description will generate a new version of the underlying concept

##### Example
* `POST /orgs/IHTSDO/sources/SNOMED-CT/concepts/12845003/descriptions/`
```JSON
{
    "description": "Tuberculous bronchiectasis, fibrosis of lung, pneumonia, pneumothorax, confirmed by sputum microscopy with culture only",
    "locale": "en",
    "locale_preferred": "true"
}
```

##### Response
* Status: 201 Created
* Location: http://api.openconceptlab.com/orgs/IHTSDO/sources/SNOMED-CT/concepts/12845003/descriptions/aY873Hbmkdi09jeh/
```JSON
{
    "type": "ConceptDescription",
    "uuid": "aY873Hbmkdi09jeh",
    "external_id": "9390",
    "description": "Tuberculous bronchiectasis, fibrosis of lung, pneumonia, pneumothorax, confirmed by sputum microscopy with culture only",
    "locale": "en",
    "locale_preferred": "true",
    "description_type": None
}
```



##### Edit concept description
* Edit concept description
```
PUT /orgs/:org/sources/:source/concepts/:concept/descriptions/:description_id/
PUT /users/:user/sources/:source/concepts/:concept/descriptions/:description_id/
PUT /user/sources/:source/concepts/:concept/descriptions/:description_id/
```
* Input
    * **description** (required) string -
    * **external_id** (optional) string - optional UUID from an external source
    * **locale** (required) string - 2-character language code, e.g. "en", "es"
    * **locale_preferred** (optional) - "true" or "false", default is "false",
    * **description_type** (optional) - additional description descriptor, such as those used in SNOMED CT
* Notes
    * Concept descriptions are part of a concept, therefore updating a concept description will generate a new version of the underlying concept

##### Example
* `POST /orgs/IHTSDO/sources/SNOMED-CT/concepts/12845003/descriptions/aY873Hbmkdi09jeh/`
```JSON
{
    "description": "Culture-only confirmation of tuberculous bronchiectasis, fibrosis of lung, pneumonia, or pneumothorax"
}
```

##### Response
* Status: 200 OK
```JSON
{
    "type": "ConceptDescription",
    "uuid": "aY873Hbmkdi09jeh",
    "external_id": "9390",
    "description": "Culture-only confirmation of tuberculous bronchiectasis, fibrosis of lung, pneumonia, or pneumothorax",
    "locale": "en",
    "locale_preferred": "true",
    "description_type": None
}
```



##### Delete concept description
* Delete concept description
```
DELETE /orgs/:org/sources/:source/concepts/:concept/descriptions/:description_id/
DELETE /users/:user/sources/:source/concepts/:concept/descriptions/:description_id/
DELETE /user/sources/:source/concepts/:concept/descriptions/:description_id/
```
* Notes
    * Concept descriptions are part of a concept, therefore deleting a concept description will generate a new version of the underlying concept

##### Example
* `DELETE /orgs/IHTSDO/sources/SNOMED-CT/concepts/12845003/descriptions/aY873Hbmkdi09jeh/`

##### Response
* Status: 204 No Content






##### Get a single extra
* Get a single extra
```
GET /orgs/:org/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]extras/:field_name/
GET /users/:user/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]extras/:field_name/
GET /user/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]extras/:field_name/
```

##### Example
* `GET /orgs/IHTSDO/sources/SNOMED-CT/concepts/12845003/extras/isprimitive/`


##### Response
* Status: 200 OK
```JSON
{
    "isprimitive": "true"
}
```



##### List all extras
* List all extras
```
GET /orgs/:org/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]extras/
GET /users/:user/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]extras/
GET /user/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]extras/
```

##### Response
* Status: 200 OK
```JSON
{
    "isprimitive": "true",
    "UMLS_CUI": "C0200703",
    "list-data": [ "1", "ab" ],
    "dictionary-data": { "more": "data" }
}
```



##### Create or Update an extra
* Create or update an extra
```
PUT /orgs/:org/sources/:source/concepts/:concept/extras/:field_name/
PUT /users/:user/sources/:source/concepts/:concept/extras/:field_name/
PUT /user/sources/:source/concepts/:concept/extras/:field_name/
(was POST)
```
* Notes
    * Extras are part of a concept, therefore creating or updating an extra will generate a new version of the underlying concept

##### Example
* `PUT /orgs/IHTSDO/sources/SNOMED-CT/concepts/12845003/extras/isprimitive/`
```JSON
{
    "isprimitive": "false"
}
```

##### Response
* Status: 200 OK
```JSON
{
    "isprimitive": "false"
}
```



##### Delete an extra
* Delete an extra
```
DELETE /orgs/:org/sources/:source/concepts/:concept/extras/:field_name/
DELETE /users/:user/sources/:source/concepts/:concept/extras/:field_name/
DELETE /user/sources/:source/concepts/:concept/extras/:field_name/
```
* Notes
    * Extras are part of a concept, therefore deleting an extra will generate a new version of the underlying concept

##### Example
* `DELETE /orgs/IHTSDO/sources/SNOMED-CT/concepts/12845003/extras/isprimitive/`

##### Response
* Status: 204 No Content



##### Search and Filter Behavior
* Text Search (e.g. `q=criteria`) - NOTE: Plus-sign (+) indicates relative relevancy weight of the term
    * concept.id (++++), concept.names.name (++++), concept.descriptions.description (+)
* Facets
    * **conceptClass** - concept.concept_class
    * **datatype** - concept.datatype
    * **locale** - concept.names.locale
    * **retired** - concept.retired (default=false)
    * **source** - concept.source
    * **owner** - concept.owner
    * **ownerType** - concept.owner_type
* Filters
    * **mapCode** - mapping.to_concept_code, mapping.from_concept_code
* Sort
    * **bestMatch** (default) - see text search fields above
    * **name** (Asc/Desc) - concept.display_name
    * **lastUpdate** (Asc/Desc) - concept.updated_on
    * **id** (Asc/Desc) - concept.id



##### Issues
* "purge" behavior is not yet implemented
* Ability to filter concept search by "mapcode" is not yet implemented.
* Need to update mappings in the concept details to the latest mappings spec
* Currently "display" and "display_locale" are hard coded rather than dynamic attributes set according to the current user's preferences and the concept values.
* Concept details spec has attributes of the count of "stars" and "collections" -- these are not currently implemented.
* Need to confirm implementation of "sortAsc" and "sortDesc"
* Confirm URL parameters
* Need to add method to actually delete a concept, e.g. `DELETE /.../?purge=true`

#### Mappings

#### Sources

#### Collection

#### Orgs

#### Users

