# Mappings
## Table of Contents
* [Overview](mappings#overview)
* [Get a single mapping](mappings#get-a-single-mapping)
* [List mappings for a concept within a single source](mappings#list-mappings-for-a-concept-within-a-single-source)
* [List all mappings within a specific source](mappings#list-all-mappings-within-a-specific-source)
* [List mappings across public sources](mappings#list-mappings-across-public-sources)
* [Create a new mapping](mappings#create-a-new-mapping)
* [Edit a mapping](mappings#edit-a-mapping)
* [Retire mapping](mappings#retire-mapping)



## Overview
The API exposes a representation of `mappings` to represent relationships between 2 concepts. The type of relationship is defined by the `map_type` attribute. Relationships are unidirectional, originating from the `from_concept` to the `to_concept`, even if the inverse mapping is equivalent (e.g. "Same As" relationship). Note that the inverse mappings can be retrieved from the "to_concept" by setting the "includeInverseMappings" to `true`. `mappings` are also used to store hierarchical relationships (such as parent/child), and OpenMRS-specific relationships such as Question/Answer and Concept Sets.

The `from_concept` must be a concept within an OCL source, while the `to_concept` may refer to a concept stored in OCL or to an external code. Regardless, the definition for the source must be in OCL. If the `to_concept` refers to a concept in OCL, it is referenced using the `to_concept_url` field. If the `to_concept` refers to an external concept, it is referenced using the `to_source_url`, `to_concept_code`, and (optionally) `to_concept_name` fields. In other words, there are two types of **to_concepts**:

1. **Internal** - The `to_concept` is stored in OCL
2. **External** - The `to_source` is defined in OCL (as an "external" source), but the `to_concept` is not.

Following are the fields that are required for Internal and External mappings:

| Field | Type | Notes |
| -------------- | ----- | ----- |
| `from_concept_url` | 1, 2 | Required for both types of mappings. |
| `to_concept_url` | 1 | Required for internal mappings. |
| `to_source_url` | 2 | Required for external mappings. URL in OCL of the `to_concept` source (e.g. /orgs/Regenstrief/sources/LOINC/). |
| `to_concept_code` | 2 | Required for external mappings. Identifier for the external `to_concept`. |
| `to_concept_name` | (2) | Optional for external mappings. The human-readable name for the external `to_concept`. |

Editing of mappings is supported, but edits that substantively change the meaning of a mapping is discouraged. For example, instead of changing the "from" or "to" concept of an existing mapping, retire the mapping and create a new one.

`mappings` are owned by `sources`, not by their `from_concept`. Modifications to mappings do not directly effect the concepts to which they are linked. Like `concepts`, `mappings` will be saved as part of source versions. Mappings may point to concepts from any source, meaning that neither the "from" or "to" concept needs to be in the source that owns the mapping. This allows sources to be used as containers of a set of mappings.

### Versioning of mappings
* All changes to mappings are tracked and can be accessed via a mapping's history


## Get a single mapping
* Get a single mapping
```
GET /orgs/:org/sources/:source/[:sourceVersion/]mappings/:mapping/
GET /users/:user/sources/:source/[:sourceVersion/]mappings/:mapping/
GET /user/sources/:source/[:sourceVersion/]mappings/:mapping/
```

### Example
```
GET /orgs/Regenstrief/sources/loinc2/mappings/8d492ee0-c2cc-11de-8d13-0010c6dffd0f/
```

### Response
* Status: 200 OK
```JSON
{
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
    "owner_url": "/orgs/Regenstrief/",

    "url": "/orgs/Regenstrief/sources/loinc2/mappings/8d492ee0-c2cc-11de-8d13-0010c6dffd0f/",

    "extras": {},

    "created_on": "2008-01-14T04:33:35Z",
    "created_by": "johndoe",
    "updated_on": "2008-02-18T09:10:16Z",
    "updated_by": "johndoe"
}
```



## List mappings for a concept within a single source
* List mappings or inverse mappings for a concept that are contained in the same source as the concept
```
GET /user/sources/:source/[:sourceVersion/]concepts/:concept/mappings/
GET /users/:user/sources/:source/[:sourceVersion/]concepts/:concept/mappings/
GET /orgs/:org/sources/:source/[:sourceVersion/]concepts/:concept/mappings/
```
* Notes
    * Use the `/mappings/` endpoint to view mappings across public sources or use `/orgs/:org/sources/:source/mappings/` for private sources.
* Parameters
    * **verbose** (optional) string - default is false; set to true to return full mapping details instead of the summary
    * **includeRetired** (optional) string - default - "false"; set to "true" to return retired mappings
    * **includeInverseMappings** (optional) string - default is "false"; set to "true" to return inverse mappings

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
    },
    {
        "map_type": "Same As",
        "retired": "false",
        "source": "loinc2",
        "owner": "Regenstrief",
        "owner_type": "Organization",
        "to_concept_url": "/orgs/Regenstrief/sources/loinc2/concepts/32700-7/",
        "from_concept_url": "/orgs/WHO/sources/ICPC-2/concepts/A73/",
        "url": "/orgs/Regenstrief/sources/loinc2/mappings/8d492ee0-c2cc-11de-8d13-0010c6dffdea/",
    }
]
```



## List all mappings within a specific source
* List all mappings within an organization or user's source
```
GET /orgs/:org/sources/:source/[:sourceVersion/]mappings/
GET /users/:user/sources/:source/[:sourceVersion/]mappings/
GET /user/sources/:source/[:sourceVersion/]mappings/
```
* Notes
    * Retired mappings are excluded by default
* Parameters
    * **verbose** (optional) string - default is false; set to true to return full mapping details instead of the summary
    * **q** (optional) string - ID of the "from" or "to" concept
    * **includeRetired** (optional) string - default is "false"; set to "true" to return retired mappings
    * **mapType** (optional) string - mapType descriptor, such as "Same As", "Parent", "Child", etc.
    * Sources
        * **conceptSource** (optional) string - comma-separated list of sources for the "from" or "to" source; can be the "code" for the source (e.g. "SNOMED-CT" or "ICPC-2") or the fully specified URL
        * **fromConceptSource** (optional) string - comma-separated list of source IDs for the "from" source (e.g. "SNOMED-CT" or "ICPC-2")
        * **toConceptSource** (optional) string - comma-separated list of source IDs for the "to" source (e.g. "SNOMED-CT" or "ICPC-2")
    * Concepts
        * **concept** (optional) string - comma-separated list of concept IDs for the "from" or "to" concept (e.g. A57)
        * **fromConcept** (optional) string - comma-separated list of concept IDs for the "from" concept (e.g. A57)
        * **toConcept** (optional) string - comma-separated list of concept IDs for the "to" concept (e.g. A57)

### Example
```
GET /orgs/Regenstrief/sources/loinc2/mappings/
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
    }
]
```



## List mappings across public sources
* List mappings across all public sources
```
GET /mappings/
```
* Parameters
    * **q** (optional) string - ID of the "from" or "to" concept
    * **includeRetired** (optional) string - default is "false"; set to "true" to return retired mappings
    * **mapType** (optional) string - map type descriptor, such as "Same As", "Parent", "Child", etc.
    * Sources
        * **conceptSource** (optional) string - comma-separated list of source IDs for the "from" or "to" source (e.g. "SNOMED-CT" or "ICPC-2")
        * **fromConceptSource** (optional) string - comma-separated list of source IDs for the "from" source (e.g. "SNOMED-CT" or "ICPC-2")
        * **toConceptSource** (optional) string - comma-separated list of source IDs for the "to" source (e.g. "SNOMED-CT" or "ICPC-2")
    * Concepts
        * **concept** (optional) string - comma-separated list of concept IDs for the "from" or "to" concept (e.g. A57)
        * **fromConcept** (optional) string - comma-separated list of concept IDs for the "from" concept (e.g. A57)
        * **toConcept** (optional) string - comma-separated list of concept IDs for the "to" concept (e.g. A57)

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
    }
]
```



## Create a new mapping
* Create a new mapping
```
POST /user/sources/:source/mappings/
POST /users/:user/sources/:source/mappings/
POST /orgs/:org/sources/:source/mappings/
```
* Input
    * Internal Mapping: `from_concept` and `to_concept` are stored in OCL
        * **id** (optional) string - ID is auto-generated if not provided
        * **map_type** (required) string - map type
        * **external_id** (optional) string - external unique identifier for import/export
        * **from_concept_url** (required) string - relative URL of the "from" concept
        * **to_concept_url** (required) string - relative URL for the "to" concept
    * External Mapping: `from_concept` is stored in OCL; `to_source` is defined in OCL, but `to_concept` is not
        * **id** (optional) string - ID is auto-generated if not provided
        * **map_type** (required) string - map type
        * **external_id** (optional) string - external unique identifier for import/export
        * **from_concept_url** (required) string - relative URL of the "from" concept
        * **to_source_url** (required) string - relative URL of the "to" source (must be stored in OCL)
        * **to_concept_code** (required) string - code for the external `to_concept`
        * **to_concept_name** (optional) string - name of the external `to_concept`

### Examples
* Internal mapping: `to_concept` is stored in OCL:
```JSON
{
    "map_type": "Same As",
    "from_concept_url": "/orgs/Columbia/sources/CIEL/concepts/161426/",
    "to_concept_url": "/orgs/Regenstrief/sources/loinc2/concepts/32700-7/"
}
```
* External mapping: `to_concept` is not stored in OCL (but `to_source_url` is defined in OCL):
```JSON
{
    "map_type": "Narrower Than",
    "from_concept_url": "/orgs/Columbia/sources/CIEL/concepts/116125/",
    "to_source_url": "/orgs/WHO/sources/ICPC-2/",
    "to_concept_code": "A73",
    "to_concept_name": "Malaria"
}
```

### Response
* Status: 201 Created
* Location: http://api.openconceptlab.com/orgs/Columbia/sources/CIEL/mappings/8d492ee0-c2cc-11de-8d13-0010c6dffd0f/
* Returns the JSON representation of the new mapping in the same format as [Get a single mapping](Mapping#get-a-single-mapping)



## Edit a mapping
* Edit a mapping
```
PUT /user/sources/:source/mappings/:mapping/
PUT /users/:user/sources/:source/mappings/:mapping/
PUT /orgs/:org/sources/:source/mappings/:mapping/
```
* Input
    * Internal Mapping: `to_concept` is stored in OCL
        * **map_type** (optional) string - map type
        * **external_id** (optional) string - external unique identifier for import/export
        * **from_concept_url** (optional) string - relative URL of the "from" concept
        * **to_concept_url** (optional) string - relative URL of the "to" concept
    * External Mapping: `to_concept` is not stored in OCL
        * **map_type** (optional) string - map type
        * **external_id** (optional) string - external unique identifier for import/export
        * **from_concept_url** (optional) string - relative URL of the "from" concept
        * **to_source_url** (optional) string - relative URL of the "to" source - note that the source must be defined in OCL
        * **to_concept_code** (optional) string - code for the external `to_concept`
        * **to_concept_name** (optional) string - name of the external `to_concept`

### Examples
* Internal Mapping: `to_concept` is stored in OCL
```JSON
{
    "map_type": "Same As",
    "to_concept_url": "/orgs/Regenstrief/sources/loinc2/concepts/32700-7/"
}
```
* External Mapping: `to_concept` is not stored in OCL
```JSON
{
    "map_type": "Narrower Than",
    "to_source_code": "ICPC-2",
    "to_concept_code": "A73",
    "to_concept_name": "Malaria",
}
```

### Response
* Status: 200 OK
* Returns the updated JSON representation of the mapping in the same format as [Get a single mapping](Mapping#get-a-single-mapping)



## Retire mapping
* Retire a mapping
```
DELETE /user/sources/:source/mappings/:mapping/
DELETE /users/:user/sources/:source/mappings/:mapping/
DELETE /orgs/:org/sources/:source/mappings/:mapping/
```
* Parameters
    * **purge** (optional) string - default is "false"; set to "true" to actually delete the mapping from all source versions
* Notes
    * DELETE does not actually delete the mapping unless "purge" is set to "true"; rather, it sets its "retired" attribute to "true" so that it does not show up by default in results, and it indicates to users that the mapping should no longer be used.

### Response
* Status: 204 No Content



## Search and Filter Behavior
* Text Search (e.g. `q=criteria`) - NOTE: Number of plus-signs (+) indicates relative relevancy weight of the term
    * Note that search criteria should be surrounded by quotes to appropriately handle whitespace, e.g. `https://api.openconceptlab.org/mappings/?concept=%22Disorder%20of%20Lymphatic%20System`
    * mapping.from_concept_code (++++), mapping.to_concept_code (++++), mapping.from_concept_name (++), mapping.to_concept_name (++)
    * external_id (+)
* Facets
    * **retired** - mapping.retired
    * **mapType** - mapping.map_type
    * **source** - mapping.source
    * **owner** - mapping.owner
    * **ownerType** - mapping.owner_type
    * **conceptSource** - mapping.from_concept.source OR mapping.to_concept.source
    * **fromConceptSource** - mapping.from_concept.source
    * **toConceptSource** - mapping.to_concept.source
    * **conceptOwner** - mapping.from_concept.owner OR mapping.to_concept.owner
    * **fromConceptOwner** - mapping.to_concept.owner
    * **toConceptOwner** - mapping.to_concept.owner
    * **conceptOnwerType** - mapping.from_concept_owner_type OR mapping.to_concept_owner_type
    * **fromConceptOnwerType** - mapping.from_concept_owner_type
    * **toConceptOnwerType** - mapping.to_concept_owner_type
* Filters
    * **concept** - mapping.from_concept_code OR mapping.to_concept_code OR mapping.from_concept_name OR mapping.to_concept_name
    * **fromConcept** - mapping.from_concept_code OR mapping.from_concept_name
    * **toConcept** - mapping.to_concept_code OR mapping.to_concept_name
* Sort
    * **bestMatch** (**default**) - see text search fields above
    * **name** (Asc/Desc) - mapping.name
    * **lastUpdated** (Asc/Desc) - mapping.updated_on



## Issues and Potential Future Features
* Add support for URL parameters for filtering mappings (e.g. "fromConcept=/orgs/CIEL/sources/CIEL/concepts/3/")
