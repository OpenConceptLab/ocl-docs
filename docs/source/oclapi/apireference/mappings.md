# Mappings

## Overview
The API exposes a representation of `mappings` to represent relationships between two concepts or codes. The type of relationship is defined by the `map_type` attribute. Relationships are unidirectional, originating from the `from_concept` to the `to_concept`, even if the inverse mapping is equivalent (e.g. "Same As" relationship). Note that the inverse mappings can be retrieved from the `to_concept` by setting the `includeInverseMappings` to `true`. In addition to the `concept.hierarchy` attributes, `mappings` may be used as a flexible approach to model other hierarchical relationships, and OpenMRS-specific relationships such as Question/Answer and Concept Sets. While mappings are generally used define relationships between sources and concepts that are defined in OCL, this is not required, allowing the definition of mappings to concepts and sources that are external to OCL.

Editing of mappings is supported, but edits that substantively change the meaning of a mapping is discouraged. For example, instead of changing the "from" or "to" concept code or updating the `map_type`, consider retiring the mapping and creating a new one.

`mappings` are owned by `sources`, not by their `from_concept`. Modifications to mappings do not directly effect the concepts to which they are linked. Like `concepts`, `mappings` will be saved as part of source versions. Mappings may point to concepts from any source, meaning that neither the "from" or "to" concept needs to be in the source that owns the mapping. This allows sources to be used as containers of a set of mappings.

A mapping's `from_concept` and `to_concept` may be defined using Canonical URLs or Relative URLs. The same approaches apply symmetrically for both `from_*` and `to_*` fields.

1. **Canonical URL** (preferred) - Uses the HL7 FHIR canonical URL of a source (e.g. `https://CIELterminology.org`) to identify the code system. This is the preferred approach because canonical URLs maintain meaning both within and outside of OCL. Note that repository version, if needed, must be specified in a separate field — the "pipe" syntax (e.g. `http://hl7.org/fhir/CodeSystem/my-codesystem|0.8`) is not supported for mappings.
2. **Relative URL: Inline** - A single relative URL specifies both source and concept, and, optionally, repository version.
3. **Relative URL: Expanded** - Source, concept, and, optionally, repository version, are specified in separate fields.

The table below shows how the `from_*` fields are used in each approach. The `to_*` fields work identically (replace `from_` with `to_`).

| Field                 | Canonical URL                    | Relative URL: Inline                     | Relative URL: Expanded |
| --------------        | -----                            | -----                                    | -----                  |
| `from_source_url`     | `"https://CIELterminology.org"` (canonical URL) | _(n/a)_                       | `"/orgs/CIEL/sources/CIEL/"` (relative URL) |
| `from_source_version` | _(optional)_                       | _(optional — can embed in `from_concept_url`)_ | _(optional — can embed in `from_source_url`)_ |
| `from_concept_code`   | `"161426"`                       | _(n/a)_                                    | `"161426"`               |
| `from_concept_name`   | _(optional)_ `"Malarial parasites by smear test"` | _(optional)_                  | _(optional)_             |
| `from_concept_url`    | _(n/a)_                            | `"/orgs/CIEL/sources/CIEL/concepts/161426/"` | _(n/a)_                  |

### Versioning of mappings
All changes to mappings are tracked and can be accessed via a mapping's history. The latest version of a concept is retrieved if no version identifier (for the repository or mapping) is specified. If a repository version identifier is specified, then the version of the concept at the time the repository version was created is used. Altenratively, specific versions of mappigns may be retrieved directly, though this is designed as an administrative function and not intended for external use.

OCL does not create a new version of a mapping in the HEAD of a repo if the submitted mapping details would result in the creation of an identical mapping version as the latest mapping version already in HEAD, as determined by its standard checksum. In this case, OCL responds with a `208 Already reported` status code. Because edits only take place in the HEAD version of a repository, this behavior has no effect on the content returned to a user and it helps to streamline resource management because it reduces the number of interim mapping versions that must be maintained. Please refer to the Checksum documentation for more information.


### Other notes and attributes of mappings
* Mapping IDs (both the OCL ID and External ID) can be automatically generated upon resource creation using the auto-id assignment scheme outlined in the [Create Source page](https://docs.openconceptlab.org/en/latest/oclapi/apireference/sources.html#create-source)
* Mappings can be given a sort weight using the numeric `sort_weight` attribute, which is used in OCL's TermBrowser application to visually sort mapped concepts within a particular map type. A sort weight can be applied using OCL's Bulk Import, API, or TermBrowser's Edit Mapping form or in the Associations section of a concept.

### Future work
* Implement support for `context` (as specified in FHIR)
* How to interact with OCL mappings via the OCL FHIR Core (and vice versa)

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
    "id": "8d492ee0-c2cc-11de-8d13-0010c6dffd0f",
    "external_id": "a9d93ffjjen9dnfekd9",
    "retired": false,
    "map_type": "Same As",

    "from_source_owner": "Regenstrief",
    "from_source_owner_type": "Organization",
    "from_source_name": "loinc2",
    "from_source_url": "/orgs/Regenstrief/sources/loinc2/",
    "from_source_version": null,
    "from_concept_code": "32700-7",
    "from_concept_name": "Malarial Smear",
    "from_concept_url": "/orgs/Regenstrief/sources/loinc2/concepts/32700-7/",
    "from_concept_name_resolved": "Malarial Smear",

    "to_source_owner": "WHO",
    "to_source_owner_type": "Organization",
    "to_source_name": "ICPC-2",
    "to_source_url": "/orgs/WHO/sources/ICPC-2/",
    "to_source_version": null,
    "to_concept_code": "A73",
    "to_concept_name": "Malaria",
    "to_concept_url": null,
    "to_concept_name_resolved": null,

    "source": "loinc2",
    "owner": "Regenstrief",
    "owner_type": "Organization",

    "url": "/orgs/Regenstrief/sources/loinc2/mappings/8d492ee0-c2cc-11de-8d13-0010c6dffd0f/",
    "version": "8d492ee0-c2cc-11de-8d13-0010c6dffd0f",
    "version_url": "/orgs/Regenstrief/sources/loinc2/mappings/8d492ee0-c2cc-11de-8d13-0010c6dffd0f/12345/",
    "versioned_object_id": 12345,
    "versioned_object_url": "/orgs/Regenstrief/sources/loinc2/mappings/8d492ee0-c2cc-11de-8d13-0010c6dffd0f/",
    "is_latest_version": true,
    "update_comment": null,
    "sort_weight": null,

    "extras": {},
    "checksums": {
        "smart": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6",
        "standard": "f6e5d4c3b2a1f6e5d4c3b2a1f6e5d4c3"
    },

    "version_created_on": "2008-01-14T04:33:35Z",
    "version_updated_on": "2008-02-18T09:10:16Z",
    "version_updated_by": "johndoe",
    "created_on": "2008-01-14T04:33:35Z",
    "created_by": "johndoe",
    "updated_on": "2008-02-18T09:10:16Z",
    "updated_by": "johndoe",
    "public_can_view": true,
    "latest_source_version": "v2023-09-11"
}
```

Note: When a mapping uses canonical URLs (e.g. `from_source_url` is `"https://CIELterminology.org"` instead of a relative URL), the response will show the canonical URL in `from_source_url` / `to_source_url`. If the canonical URL resolves to a source in OCL, the `from_source_owner`, `from_source_name`, and related fields will be populated; otherwise, they may be `null`.



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
        "type": "Mapping",
        "id": "8d492ee0-c2cc-11de-8d13-0010c6dffd0f",
        "map_type": "Same As",
        "retired": false,
        "source": "loinc2",
        "owner": "Regenstrief",
        "owner_type": "Organization",
        "from_concept_code": "32700-7",
        "from_concept_name": null,
        "from_concept_url": "/orgs/Regenstrief/sources/loinc2/concepts/32700-7/",
        "from_source_url": "/orgs/Regenstrief/sources/loinc2/",
        "from_source_name": "loinc2",
        "from_source_version": null,
        "to_concept_code": "A73",
        "to_concept_name": "Malaria",
        "to_concept_url": "/orgs/WHO/sources/ICPC-2/concepts/A73/",
        "to_source_url": "/orgs/WHO/sources/ICPC-2/",
        "to_source_name": "ICPC-2",
        "to_source_version": null,
        "url": "/orgs/Regenstrief/sources/loinc2/mappings/8d492ee0-c2cc-11de-8d13-0010c6dffd0f/",
        "sort_weight": null
    },
    {
        "type": "Mapping",
        "id": "def3fe-c2cc-11de-8d13-asdf9393930",
        "map_type": "Narrower Than",
        "retired": false,
        "source": "loinc2",
        "owner": "Regenstrief",
        "owner_type": "Organization",
        "from_concept_code": "32700-7",
        "from_concept_name": null,
        "from_concept_url": "/orgs/Regenstrief/sources/loinc2/concepts/32700-7/",
        "from_source_url": "/orgs/Regenstrief/sources/loinc2/",
        "from_source_name": "loinc2",
        "from_source_version": null,
        "to_concept_code": "A73",
        "to_concept_name": "Malaria",
        "to_concept_url": null,
        "to_source_url": "https://who.int/classifications/icpc-2",
        "to_source_name": null,
        "to_source_version": null,
        "url": "/orgs/Regenstrief/sources/loinc2/mappings/def3fe-c2cc-11de-8d13-asdf9393930/",
        "sort_weight": null
    },
    {
        "type": "Mapping",
        "id": "8d492ee0-c2cc-11de-8d13-0010c6dffdea",
        "map_type": "Same As",
        "retired": false,
        "source": "loinc2",
        "owner": "Regenstrief",
        "owner_type": "Organization",
        "from_concept_code": "A73",
        "from_concept_name": null,
        "from_concept_url": "/orgs/WHO/sources/ICPC-2/concepts/A73/",
        "from_source_url": "/orgs/WHO/sources/ICPC-2/",
        "from_source_name": "ICPC-2",
        "from_source_version": null,
        "to_concept_code": "32700-7",
        "to_concept_name": null,
        "to_concept_url": "/orgs/Regenstrief/sources/loinc2/concepts/32700-7/",
        "to_source_url": "/orgs/Regenstrief/sources/loinc2/",
        "to_source_name": "loinc2",
        "to_source_version": null,
        "url": "/orgs/Regenstrief/sources/loinc2/mappings/8d492ee0-c2cc-11de-8d13-0010c6dffdea/",
        "sort_weight": null
    }
]
```

Note: The second mapping above demonstrates the use of a canonical URL (`"https://who.int/classifications/icpc-2"`) in `to_source_url`. When a canonical URL is used for an external source not in OCL, `to_concept_url` and `to_source_name` will be `null`.



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
    * **q** (optional) string - full-text search across from/to concept codes and names
    * **includeRetired** (optional) string - default is "false"; set to "true" to return retired mappings
    * **mapType** (optional) string - mapType descriptor, such as "Same As", "Parent", "Child", etc.
    * **updatedSince** (optional) string - filter results to those updated after the specified date/time, format: YYYY-MM-DD HH:MM:SS
    * **updatedBy** (optional) string - filter by username of the user who last updated the mapping
    * Sources
        * **conceptSource** (optional) string - comma-separated list of sources for the "from" or "to" source (e.g. "SNOMED-CT" or "ICPC-2")
        * **fromConceptSource** (optional) string - comma-separated list of source IDs for the "from" source (e.g. "SNOMED-CT" or "ICPC-2")
        * **toConceptSource** (optional) string - comma-separated list of source IDs for the "to" source (e.g. "SNOMED-CT" or "ICPC-2")
    * Concepts
        * **concept** (optional) string - comma-separated list of concept IDs for the "from" or "to" concept (e.g. A57). **Note:** this parameter may not work on repo-scoped endpoints; use ``fromConcept`` and ``toConcept`` instead (see `ocl_issues#2425 <https://github.com/OpenConceptLab/ocl_issues/issues/2425>`_)
        * **fromConcept** (optional) string - comma-separated list of concept IDs for the "from" concept (e.g. A57)
        * **toConcept** (optional) string - comma-separated list of concept IDs for the "to" concept (e.g. A57)
    * Concept Owners
        * **fromConceptOwner** (optional) string - filter by the owner of the "from" concept's source
        * **toConceptOwner** (optional) string - filter by the owner of the "to" concept's source
        * **fromConceptOwnerType** (optional) string - filter by the owner type of the "from" concept's source (Organization or User)
        * **toConceptOwnerType** (optional) string - filter by the owner type of the "to" concept's source (Organization or User)
    * Mapping Owner
        * **owner** (optional) string - filter by the owner of the source that contains the mapping (distinct from the from/to concept owners)
        * **ownerType** (optional) string - filter by owner type (Organization or User)

### Example
```
GET /orgs/Regenstrief/sources/loinc2/mappings/
```

### Response
* Status: 200 OK
```JSON
[
    {
        "type": "Mapping",
        "id": "8d492ee0-c2cc-11de-8d13-0010c6dffd0f",
        "map_type": "Same As",
        "retired": false,
        "source": "loinc2",
        "owner": "Regenstrief",
        "owner_type": "Organization",
        "from_concept_code": "32700-7",
        "from_concept_name": null,
        "from_concept_url": "/orgs/Regenstrief/sources/loinc2/concepts/32700-7/",
        "from_source_url": "/orgs/Regenstrief/sources/loinc2/",
        "from_source_name": "loinc2",
        "from_source_version": null,
        "to_concept_code": "A73",
        "to_concept_name": "Malaria",
        "to_concept_url": "/orgs/WHO/sources/ICPC-2/concepts/A73/",
        "to_source_url": "/orgs/WHO/sources/ICPC-2/",
        "to_source_name": "ICPC-2",
        "to_source_version": null,
        "url": "/orgs/Regenstrief/sources/loinc2/mappings/8d492ee0-c2cc-11de-8d13-0010c6dffd0f/",
        "sort_weight": null
    }
]
```



## List mappings across public sources
* List mappings across all public sources
```
GET /mappings/
```
* Parameters
    * **q** (optional) string - full-text search across from/to concept codes and names
    * **includeRetired** (optional) string - default is "false"; set to "true" to return retired mappings
    * **mapType** (optional) string - map type descriptor, such as "Same As", "Parent", "Child", etc.
    * **updatedSince** (optional) string - filter results to those updated after the specified date/time, format: YYYY-MM-DD HH:MM:SS
    * **updatedBy** (optional) string - filter by username of the user who last updated the mapping
    * Sources
        * **conceptSource** (optional) string - comma-separated list of source IDs for the "from" or "to" source (e.g. "SNOMED-CT" or "ICPC-2")
        * **fromConceptSource** (optional) string - comma-separated list of source IDs for the "from" source (e.g. "SNOMED-CT" or "ICPC-2")
        * **toConceptSource** (optional) string - comma-separated list of source IDs for the "to" source (e.g. "SNOMED-CT" or "ICPC-2")
    * Concepts
        * **concept** (optional) string - comma-separated list of concept IDs for the "from" or "to" concept (e.g. A57). **Note:** this parameter may not work on repo-scoped endpoints; use ``fromConcept`` and ``toConcept`` instead (see `ocl_issues#2425 <https://github.com/OpenConceptLab/ocl_issues/issues/2425>`_)
        * **fromConcept** (optional) string - comma-separated list of concept IDs for the "from" concept (e.g. A57)
        * **toConcept** (optional) string - comma-separated list of concept IDs for the "to" concept (e.g. A57)
    * Concept Owners
        * **fromConceptOwner** (optional) string - filter by the owner of the "from" concept's source
        * **toConceptOwner** (optional) string - filter by the owner of the "to" concept's source
        * **fromConceptOwnerType** (optional) string - filter by the owner type of the "from" concept's source (Organization or User)
        * **toConceptOwnerType** (optional) string - filter by the owner type of the "to" concept's source (Organization or User)
    * Mapping Owner
        * **owner** (optional) string - filter by the owner of the source that contains the mapping (distinct from the from/to concept owners)
        * **ownerType** (optional) string - filter by owner type (Organization or User)

### Response
* Status: 200 OK
```JSON
[
    {
        "type": "Mapping",
        "id": "8d492ee0-c2cc-11de-8d13-0010c6dffd0f",
        "map_type": "Same As",
        "retired": false,
        "source": "loinc2",
        "owner": "Regenstrief",
        "owner_type": "Organization",
        "from_concept_code": "32700-7",
        "from_concept_name": null,
        "from_concept_url": "/orgs/Regenstrief/sources/loinc2/concepts/32700-7/",
        "from_source_url": "/orgs/Regenstrief/sources/loinc2/",
        "from_source_name": "loinc2",
        "from_source_version": null,
        "to_concept_code": "A73",
        "to_concept_name": "Malaria",
        "to_concept_url": "/orgs/WHO/sources/ICPC-2/concepts/A73/",
        "to_source_url": "/orgs/WHO/sources/ICPC-2/",
        "to_source_name": "ICPC-2",
        "to_source_version": null,
        "url": "/orgs/Regenstrief/sources/loinc2/mappings/8d492ee0-c2cc-11de-8d13-0010c6dffd0f/",
        "sort_weight": null
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
* Edit a mapping - Creates a new version of a mapping in the HEAD of a repository
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
* Notes
    * OCL does not create a new version of a mapping in the HEAD of a repo if the submitted mapping details would result in the creation of an identical mapping version as the latest mappign version already in HEAD, as determined by its standard checksum. In this case, OCL responds with a 208 Already reported status code. Refer to the Checksum documentation for more information.


### Examples
* Edit a mapping to update the `to_concept` using a canonical URL:
```JSON
{
    "map_type": "Narrower Than",
    "to_source_url": "https://who.int/classifications/icpc-2",
    "to_concept_code": "A73",
    "to_concept_name": "Malaria"
}
```
* Edit using a relative URL:
```JSON
{
    "map_type": "Narrower Than",
    "to_source_url": "/orgs/WHO/sources/ICPC-2/",
    "to_concept_code": "A73",
    "to_concept_name": "Malaria"
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
