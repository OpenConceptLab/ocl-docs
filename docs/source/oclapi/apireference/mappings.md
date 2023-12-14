# Mappings

## Overview
The API exposes a representation of `mappings` to represent relationships between two concepts or codes. The type of relationship is defined by the `map_type` attribute. Relationships are unidirectional, originating from the `from_concept` to the `to_concept`, even if the inverse mapping is equivalent (e.g. "Same As" relationship). Note that the inverse mappings can be retrieved from the `to_concept` by setting the `includeInverseMappings` to `true`. In addition to the `concept.hierarchy` attributes, `mappings` may be used as a flexible approach to model other hierarchical relationships, and OpenMRS-specific relationships such as Question/Answer and Concept Sets. While mappings are generally used define relationships between sources and concepts that are defined in OCL, this is not required, allowing the definition of mappings to concepts and sources that are external to OCL.

Editing of mappings is supported, but edits that substantively change the meaning of a mapping is discouraged. For example, instead of changing the "from" or "to" concept code or updating the `map_type`, consider retiring the mapping and creating a new one.

`mappings` are owned by `sources`, not by their `from_concept`. Modifications to mappings do not directly effect the concepts to which they are linked. Like `concepts`, `mappings` will be saved as part of source versions. Mappings may point to concepts from any source, meaning that neither the "from" or "to" concept needs to be in the source that owns the mapping. This allows sources to be used as containers of a set of mappings.

A mapping's `from_concept` and `to_concept` may be defined using Canonical URLs or Relative URLs. 
1. **Canonical URL** - this is the preferred way of defining mappings that maintain meaning within and outside of OCL; note that repository version, if needed, must be specified in a separate field and cannot use the "pipe" syntax (e.g. this syntax is not supported for mappings: "http://hl7.org/fhir/CodeSystem/my-codeystem|0.8")
2. **Relative URL: Inline** - A single relative URL specified both source and concept, and, optionally, repository version
3. **Relative URL: Expanded** - Source, concept, and, optionally, repository version, are specified in separate fields

| Field                 | Canonical URL                    | Relative URL: Inline                     | Relative URL: Expanded |
| --------------        | -----                            | -----                                    | -----                  |
| `from_source_url`     | "https://CIELterminology.org"    | _(n/a)_                                    | "/orgs/CIEL/sources/CIEL/" |
| `from_source_version` | _(optional)_                       | _(optional- can embed in `from_concept_url`)_ | _(optional- can embed in `from_source_url`)_ |
| `from_concept_code`   | "161426"                         | _(n/a)_                                    | "161426"                 |
| `from_concept_name`   | _(optional)_ "Malarial parasites by smear test" | _(optional)_                    | _(optional)_             |
| `from_concept_url`    | _(n/a)_                            | "/orgs/CIEL/sources/CIEL/concepts/161426/" | _(n/a)_                  |

### Versioning of mappings
* All changes to mappings are tracked and can be accessed via a mapping's history

### Other notes and attributes of mappings
* Mapping IDs (both the OCL ID and External ID) can be automatically generated upon resource creation using the auto-id assignment scheme outlined in the [Create Source page]([url](https://docs.openconceptlab.org/en/latest/oclapi/apireference/sources.html#create-source))
* Mappings can be given a sort weight using the numeric `sort_weight` attribute, which is used in OCL's TermBrowser application to visually sort mapped concepts within a particular map type. A sort weight can be applied using OCL's Bulk Import, API, or TermBrowser's Edit Mapping form or in the Associations section of a concept.    



### Changes Needed to this Documentation:
- Add to the Overview:
  - Support canonical URLs
  - How to interact with OCL mappings via the OCL FHIR Core (and vice versa)
- Confirm whether repo version can be specified separately or inline for each of the 3 approaches
- Update all examples to use canonical URLs
- Update response examples with all of the new fields
- Add OpenMRS specific examples

### Future work
- Implement support for `context` (as specified in FHIR)

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
    * **id** (optional) string - ID is auto-generated if not provided
    * **map_type** (required) string - map type, e.g. "SAME-AS, "NARROWER-THAN"
    * **external_id** (optional) string - external unique identifier for import/export
    * **retired** (optional) bool - default: `false`; set to `true` to mark that this mapping is not recommended for use
    * **sort_weight** (optional) decimal - a numeric value indicating where you want the mapping sorted relative to other mappings of the same `map_type`
    * **extras** (optional) JSON dictionary - additional metadata for the resource
    * `from_concept`:
        * **from_concept_url** (optional) string - relative URL of the `from_concept`
        * **from_concept_code** (optional) string - code for the `from_concept`; required if `from_concept_url` is not provided, otherwise omitted
        * **from_concept_name** (optional) string - optional name of the `from_concept` within the context of the mapping; this need not be the same as one of the concept's names or synonyms
        * **from_source_url** (optional) string - canonical or relative URL of the `from_source`; if `from_concept_url` is not provided, this field is required, otherwise it is omitted
        * **from_source_version** (optional) string - version identifier for the source; Note that best practice is to only define this field if absolutely necessary
    * `to_concept`:
        * **to_concept_url** (optional) string - relative URL of the `to_concept`
        * **to_concept_code** (optional) string - code for the `to_concept`; required if `to_concept_url` is not provided, otherwise omitted
        * **to_concept_name** (optional) string - optional name of the `to_concept` within the context of the mapping; this need not be the same as one of the concept's names or synonyms
        * **to_source_url** (optional) string - canonical or relative URL of the `to_source`; if `to_concept_url` is not provided, this field is required, otherwise it is omitted
        * **to_source_version** (optional) string - version identifier for the source; Note that best practice is to only define this field if absolutely necessary

### Examples
* Example defining a mapping using canonical URLs:
```JSON
{
    "map_type": "NARROWER-THAN",
    "from_source_url": "https://CIELterminology.org",
    "from_concept_code": "168094",
    "from_concept_name": "Mother pregnant or currently breastfeeding",
    "to_source_url": "http://hl7.org/fhir/ValueSet/medicationdispense-status-reason",
    "to_concept_code": "preg",
    "to_concept_name": "Pregnant or breastfeeding"
}
```
* Simple example using relative URLs for both from and to concepts
```JSON
{
    "map_type": "SAME-AS",
    "from_concept_url": "/orgs/CIEL/sources/CIEL/concepts/161426/",
    "to_concept_url": "/orgs/Regenstrief/sources/LOINC/concepts/32700-7/"
}
```
* Example where the `to_concept` is not stored in OCL, but the `to_source` is
```JSON
{
    "map_type": "NARROWER-THAN",
    "from_concept_url": "/orgs/CIEL/sources/CIEL/concepts/116125/",
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
    * **map_type** (optional) string - map type, e.g. "SAME-AS, "NARROWER-THAN"
    * **external_id** (optional) string - external unique identifier for import/export
    * **retired** (optional) bool - default: `false`; set to `true` to mark that this mapping is not recommended for use
    * **sort_weight** (optional) decimal - a numeric value indicating where you want the mapping sorted relative to other mappings of the same `map_type`
    * **extras** (optional) JSON dictionary - additional metadata for the resource
    * `from_concept`:
        * **from_concept_url** (optional) string - relative URL of the `from_concept`
        * **from_concept_code** (optional) string - code for the `from_concept`; required if `from_concept_url` is not provided, otherwise omitted
        * **from_concept_name** (optional) string - optional name of the `from_concept` within the context of the mapping; this need not be the same as one of the concept's names or synonyms
        * **from_source_url** (optional) string - canonical or relative URL of the `from_source`; if `from_concept_url` is not provided, this field is required, otherwise it is omitted
        * **from_source_version** (optional) string - version identifier for the source; Note that best practice is to only define this field if absolutely necessary
    * `to_concept`:
        * **to_concept_url** (optional) string - relative URL of the `to_concept`
        * **to_concept_code** (optional) string - code for the `to_concept`; required if `to_concept_url` is not provided, otherwise omitted
        * **to_concept_name** (optional) string - optional name of the `to_concept` within the context of the mapping; this need not be the same as one of the concept's names or synonyms
        * **to_source_url** (optional) string - canonical or relative URL of the `to_source`; if `to_concept_url` is not provided, this field is required, otherwise it is omitted
        * **to_source_version** (optional) string - version identifier for the source; Note that best practice is to only define this field if absolutely necessary
    * **update_comment** (optional) string - Brief description of the update

### Examples
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
