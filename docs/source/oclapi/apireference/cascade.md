# Operation: $cascade

## Overview
The API exposes a `$cascade` operation to retrieve a list of associated concepts and mappings initiated from a specific concept code for a source or collection version. Associations may be determined using both hierarchy and mappings and may be processed recursively. Note that a repository version is required in order to unambiguously navigate hierarchy and mappings – therefore, it is not possible to cascade interim resource versions. `$cascade` can return a hierarchical response or a flattened response.

## Get a list of resources that are associated with a concept within a specific source or collection version
If `:sourceVersion`/`:collectionVersion` is omitted, OCL defaults to the `latest` released repository version (not `HEAD`), or returns an error if that does not exist. Note that it is possible to cascade the `HEAD` version of a repository.
```
GET /:ownerType/:ownerId/sources/:source/[:sourceVersion/]concepts/:concept/$cascade/
GET /:ownerType/:ownerId/collections/:collection/[:collectionVersion/]concepts/:concept/$cascade/
```

**Parameters**
* `mapTypes` (0..\*) - Comma-delimited list of map types used to process the cascade, e.g. `*` or `Q-AND-A,CONCEPT-SET`. If set, map types not in this list are ignored.
* `excludeMapTypes` (0..\*) - Comma-delimited list of map types to exclude from processing the cascade. If set, all map types are cascaded except for those listed here. This parameter is ignored if `mapType` is set.
* `returnMapTypes` (0..\*) - Comma-delimited list of map types to include in the resultset. If no value (the default), then this takes on the value of `mapTypes`. `*` returns all of a concept’s mappings; `false`/`0` will not include any mappings in the resultset.
* `method` (0..1) - default=`sourcetoconcepts`; other option is `sourcemappings`. Controls cascade behavior via mappings, target concepts, or both. Note, to cascade everything (mappings, target concepts, and source hierarchy), use `method=sourceToConcepts` and `cascadeHierarchy=true`
  * `sourceMappings`: Cascade only mappings and not target concepts and not source hierarchy
  * `sourceToConcepts`: Cascades mappings and target concepts
* `cascadeHierarchy` (optional; boolean) - default=true; if true (default), cascade a concept’s parent/child hierarchy relationships
* `cascadeMappings` (optional; boolean) - default=true; if true (default), cascade a concept’s mappings
* `includeMappings` (optional; boolean) - default=true; if true, all mappings that are cascaded are included in the response; set this to false to exclude the mappings from the response and only include the concepts
* `cascadeLevels` (optional) - default="\*"; Set the number of levels of cascading to process from the starting concept. The number of levels of recursion for the cascade process, beginning from the requested root concept, where 0=no recursion, so only the root concept and its children/target concepts are processed. 1=one level of recursion, so the root concept, its children/targets, and their children/targets are included in the response. “\*”=infinite recursion, where the process recursively cascades until it is not possible to go further or until it has reached a hard limit (e.g. 1,000 resources, or as set by the system administrator). Note that the `$cascade` operator prevents infinite loops by not cascading a concept that has already been cascaded. Also note that each level of recursion applies the same “mapTypes” or “excludeMapTypes” filters.
* `reverse` (optional; boolean) - default=false. By default, `$cascade` is processed from parent-to-child, from-concept-to-target-concept. Set `reverse=false` to process in the reverse direction, from child-to-parent and target-concept-to-from-concept.
* `view` (string) - `flat` (default), `hierarchy`; Set to `“hierarchy”` to have entries returned inside each concept, beginning with the requested root concept. The default `“flat”` behavior simply returns a flat list of all concepts and mappings.


Example Request

```
POST /orgs/CIEL/sources/CIEL/v2021-03-12/concepts/116128/$cascade/
```
```
{
    "resourceType": "Bundle",
    "type": "searchset",
    "meta": {
        "lastUpdated": "2022-04-29T04:28:27.626537Z"
    },
    "total": 11,
    "entry": [
        {
            "uuid": "31566",
            "id": "116128",
            "type": "Concept",
            "url": "/orgs/CIEL/sources/CIEL/concepts/116128/",
            "version_url": "/orgs/CIEL/sources/CIEL/concepts/116128/1582048/"
        },
        {
            "uuid": "53703",
            "id": "53703",
            "type": "Mapping",
            "map_type": "NARROWER-THAN",
            "url": "/orgs/CIEL/sources/CIEL/mappings/53703/",
            "version_url": "/orgs/CIEL/sources/CIEL/mappings/53703/2365752/",
            "to_concept_code": "1F4Z",
            "to_concept_url": null,
            "target_concept_code": "1F4Z",
            "target_concept_url": null,
            "target_source_owner": "WHO",
            "target_source_name": "ICD-11-WHO"
        },
        {
            "uuid": "65591",
            "id": "65591",
            "type": "Mapping",
            "map_type": "NARROWER-THAN",
            "url": "/orgs/CIEL/sources/CIEL/mappings/65591/",
            "version_url": "/orgs/CIEL/sources/CIEL/mappings/65591/2365778/",
            "to_concept_code": "A73",
            "to_concept_url": null,
            "target_concept_code": "A73",
            "target_concept_url": null,
            "target_source_owner": "WICC",
            "target_source_name": "ICPC2"
        },
        {
            "uuid": "54994",
            "id": "54994",
            "type": "Mapping",
            "map_type": "SAME-AS",
            "url": "/orgs/CIEL/sources/CIEL/mappings/54994/",
            "version_url": "/orgs/CIEL/sources/CIEL/mappings/54994/2365747/",
            "to_concept_code": "906",
            "to_concept_url": null,
            "target_concept_code": "906",
            "target_concept_url": null,
            "target_source_owner": "AMPATH",
            "target_source_name": "AMPATH"
        },
        {
            "uuid": "62352",
            "id": "62352",
            "type": "Mapping",
            "map_type": "SAME-AS",
            "url": "/orgs/CIEL/sources/CIEL/mappings/62352/",
            "version_url": "/orgs/CIEL/sources/CIEL/mappings/62352/2365750/",
            "to_concept_code": "28660",
            "to_concept_url": null,
            "target_concept_code": "28660",
            "target_concept_url": null,
            "target_source_owner": "IMO",
            "target_source_name": "IMO-ProblemIT"
        },
        {
            "uuid": "65611",
            "id": "65611",
            "type": "Mapping",
            "map_type": "SAME-AS",
            "url": "/orgs/CIEL/sources/CIEL/mappings/65611/",
            "version_url": "/orgs/CIEL/sources/CIEL/mappings/65611/2365749/",
            "to_concept_code": "123",
            "to_concept_url": null,
            "target_concept_code": "123",
            "target_concept_url": null,
            "target_source_owner": "PIH",
            "target_source_name": "PIH-Malawi"
        },
        {
            "uuid": "68495",
            "id": "68495",
            "type": "Mapping",
            "map_type": "SAME-AS",
            "url": "/orgs/CIEL/sources/CIEL/mappings/68495/",
            "version_url": "/orgs/CIEL/sources/CIEL/mappings/68495/2365774/",
            "to_concept_code": "123",
            "to_concept_url": null,
            "target_concept_code": "123",
            "target_concept_url": null,
            "target_source_owner": "PIH",
            "target_source_name": "PIH"
        },
        {
            "uuid": "65980",
            "id": "65980",
            "type": "Mapping",
            "map_type": "SAME-AS",
            "url": "/orgs/CIEL/sources/CIEL/mappings/65980/",
            "version_url": "/orgs/CIEL/sources/CIEL/mappings/65980/2365772/",
            "to_concept_code": "123",
            "to_concept_url": null,
            "target_concept_code": "123",
            "target_concept_url": null,
            "target_source_owner": "AMPATH",
            "target_source_name": "AMPATH"
        },
        {
            "uuid": "1058857",
            "id": "1058857",
            "type": "Mapping",
            "map_type": "SAME-AS",
            "url": "/orgs/CIEL/sources/CIEL/mappings/1058857/",
            "version_url": "/orgs/CIEL/sources/CIEL/mappings/1058857/2365776/",
            "to_concept_code": "116128",
            "to_concept_url": "/orgs/CIEL/sources/CIEL/concepts/116128/",
            "target_concept_code": "116128",
            "target_concept_url": "/orgs/CIEL/sources/CIEL/concepts/116128/",
            "target_source_owner": "CIEL",
            "target_source_name": "CIEL"
        },
        {
            "uuid": "54798",
            "id": "54798",
            "type": "Mapping",
            "map_type": "SAME-AS",
            "url": "/orgs/CIEL/sources/CIEL/mappings/54798/",
            "version_url": "/orgs/CIEL/sources/CIEL/mappings/54798/1058845/",
            "to_concept_code": "B54",
            "to_concept_url": "/orgs/WHO/sources/ICD-10-WHO/concepts/B54/",
            "target_concept_code": "B54",
            "target_concept_url": "/orgs/WHO/sources/ICD-10-WHO/concepts/B54/",
            "target_source_owner": "WHO",
            "target_source_name": "ICD-10-WHO"
        },
        {
            "uuid": "59244",
            "id": "59244",
            "type": "Mapping",
            "map_type": "SAME-AS",
            "url": "/orgs/CIEL/sources/CIEL/mappings/59244/",
            "version_url": "/orgs/CIEL/sources/CIEL/mappings/59244/2365770/",
            "to_concept_code": "61462000",
            "to_concept_url": null,
            "target_concept_code": "61462000",
            "target_concept_url": null,
            "target_source_owner": "IHTSDO",
            "target_source_name": "SNOMED-CT"
        }
    ]
}
```
