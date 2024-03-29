# Operation: $cascade

## Overview
The API exposes a `$cascade` operation to retrieve a list of associated concepts and mappings initiated from a specific concept within a source or collection version. Associations may be determined using both hierarchy (parent-child relationships) and/or mappings, and associations may be processed recursively. Note that a repository version is required in order to unambiguously navigate hierarchy and mappings – therefore, it is not possible to cascade interim resource versions. `$cascade` can return a hierarchical response or a flattened response.

## Get a list of resources that are associated with a concept within a specific source or collection version
```
GET /:ownerType/:ownerId/sources/:source/[:sourceVersion/]concepts/:concept/$cascade/
GET /:ownerType/:ownerId/collections/:collection/[:collectionVersion/]concepts/:concept/$cascade/
```

Notes:
* If `:sourceVersion`/`:collectionVersion` is omitted, OCL defaults to the `latest` released repository version (not `HEAD`), or returns an error if that does not exist. Note that it is possible to cascade the `HEAD` version of a repository.
* It is possible for a concept to appear in a result set more than once (i.e. multiple concepts have the same concept as a child). In a hierarchical response (`view=hierarchy`), the same concept may appear more than once, but only the first appearance of a concept will be cascaded. In a flattened response (`view=flat`), duplicates are removed so the concept will appear only once.

**Input Parameters**
* `mapTypes` (0..\*) - Comma-delimited list of map types used to process the cascade, e.g. `*` or `Q-AND-A,CONCEPT-SET`. If set, map types not in this list are ignored.
* `excludeMapTypes` (0..\*) - Comma-delimited list of map types to exclude from processing the cascade. If set, all map types are cascaded except for those listed here. This parameter is ignored if `mapType` is set.
* `returnMapTypes` (0..\*) - Comma-delimited list of map types to include in the resultset. If no value (the default), then this takes on the value of `mapTypes`. `*` returns all of a concept’s mappings; `false`/`0` will not include any mappings in the resultset.
* `method` (0..1) - default=`sourcetoconcepts`; other option is `sourcemappings`. Controls cascade behavior via mappings, target concepts, or both. Note, to cascade everything (mappings, target concepts, and source hierarchy), use `method=sourceToConcepts` and `cascadeHierarchy=true`
  * `sourceMappings`: Cascade only mappings and not target concepts and not source hierarchy
  * `sourceToConcepts`: Cascades mappings and target concepts
* `cascadeHierarchy` (optional; boolean) - default=true; if true (default), cascade a concept’s parent/child hierarchy relationships
* `cascadeMappings` (optional; boolean) - default=true; if true (default), cascade a concept’s mappings
* `cascadeLevels` (optional) - default="\*"; Set the number of levels of cascading to process from the starting concept. The number of levels of recursion for the cascade process, beginning from the requested root concept, where 0=no recursion, so only the root concept and its children/target concepts are processed. 1=one level of recursion, so the root concept, its children/targets, and their children/targets are included in the response. “\*”=infinite recursion, where the process recursively cascades until it is not possible to go further or until it has reached a hard limit (e.g. 1,000 resources, or as set by the system administrator). Note that the `$cascade` operator prevents infinite loops by not cascading a concept that has already been cascaded. Also note that each level of recursion applies the same “mapTypes” or “excludeMapTypes” filters.
* `reverse` (optional; boolean) - default=false. By default, `$cascade` is processed from parent-to-child, from-concept-to-target-concept. Set `reverse=false` to process in the reverse direction, from child-to-parent and target-concept-to-from-concept.
* `view` (string) - `flat` (default), `hierarchy`; Set to `“hierarchy”` to have entries returned inside each concept, beginning with the requested root concept. The default `“flat”` behavior simply returns a flat list of all concepts and mappings.
* `omitIfExistsIn` (string) - Relative or canonical URL of a repository (or repository version) used to terminate the processing of a branch if the current concept already exists in the repository version specified in `omitIfExistsIn`. The matching concept and associated mappings will not be included in the result set, and its children (and associated mapping) will also be omitted from the result set unless they happen to be included via processing of another returned concept.
* `equivalencyMapType` (string) - A map type (e.g. `SAME-AS`) used to terminate the processing of a branch if the current concept has an equivalent mapping in the repository version specified in `omitIfExistsIn`. This attribute has no effect if blank or if `omitIfExistsIn` is empty or does not point to a valid repository.
* `includeMappings` DEPRECATED (optional; boolean) - default=true; if true, all mappings that are cascaded are included in the response; set this to false to exclude the mappings from the response and only include the concepts. This parameter is deprecated as it has been replaced by `returnMapTypes`, which has even more features.

**Output Parameters**
* `resourceType` - `Bundle`
* `type` - `searchset`
* `requested_url` - The relative URL of the original request
* `repo_version_url` - The relative URL of the repository version used to process the cascade request. If the repository version is specified in the original request, this will always reflect that. If the repository version was not specified
* `total` - In a flattened response, the total number of entries in the result set
* `meta` - Metadata about the request, such as `lastUpdated`
* `entry` - The concept from where the cascade operations was initiated
* `entry.type` - `Concept` or `Mapping`
* `entry.id` - ID/mnemonic of the resource
* `entry.url` - Version-less relative URL to the resource
* `entry.version_url` - Relative URL to the resource within its repository version
* `entry.retired`
* For concepts:
  * `entry.display_name`
  * `entry.terminal` - For a concept in the result set, `terminal` indicates if the cascade operation cut off the tree, if it is possible to continue cascading further given the input parameters, or if it is indeterminate:
    * `true` - The concept was cascaded, and there were no further results for the concept. i.e. The tree, as defined by the input parameters, does not go any further.
    * `false` - The concept was cascaded, and there were results that were included in the response. Note: For a hierarchical response, associated mappings and target concepts will always appear with the first occurrence of a concept.
    * `null` - The concept was not cascaded based on the provided parameters (e.g. cascadeLevels=1), so no information is available about whether the concept is terminal.
  * `entry.entries` - For a hierarchical response, includes the list of Mappings and target Concepts associated with this concept based on the input parameters.
* For mappings:
  * `entry.map_type`
  * `entry.sort_weight`
  * If `reverse=false`:
    * `entry.to_concept_code`
    * `entry.to_concept_url`
  * If `reverse=true`:
    * `entry.from_concept_code`
    * `entry.from_concept_url`
  * `entry.cascade_target_concept_code`
  * `entry.cascade_target_concept_url`
  * `entry.cascade_target_concept_name`
  * `entry.cascade_target_source_owner`
  * `entry.cascade_target_source_name`

### Example requests
* Default: Cascade all map types recursively
```
/users/ocladmin/sources/CascadeTest/v2/concepts/BB/$cascade/
```
* Cascade SAME-AS mappings recursively -- Note: Only SAME-AS mappings are returned in the result set
```
/users/ocladmin/sources/CascadeTest/v2/concepts/BB/$cascade/?mapTypes=SAME-AS
```
* Cascade SAME-AS mappings recursively, but return all of each concept's mappings in the result set
```
/users/ocladmin/sources/CascadeTest/v2/concepts/BB/$cascade/?mapTypes=SAME-AS&returnMapTypes=*
```
* Cascade 2-levels in the reverse direction
```
/users/ocladmin/sources/CascadeTest/v2/concepts/CC/$cascade/?reverse=true&cascadeLevels=1
```
* OpenMRS-compatible cascade - Includes associated Mappings and target Concepts from the same source, and recursively adds any of their answer and set member concepts and mappings (e.g. CONCEPT-SET and Q-AND-A mappings)
```
/users/ocladmin/sources/CascadeTest/v2/concepts/BB/$cascade/?mapType=CONCEPT-SET,Q-AND-A&returnMapType=*&view=hierarchy
```


### Example request and response
```
GET /users/ocladmin/sources/CascadeTest/v2/concepts/BB/$cascade/?mapType=CONCEPT-SET,Q-AND-A&returnMapType=*&view=hierarchy
```
```
{
  "resourceType": "Bundle",
  "type": "searchset",
  "meta": {
    "lastUpdated": "2022-10-27T08:11:07.965651Z"
  },
  "total": null,
  "entry": {
    "uuid": "325662",
    "id": "BB",
    "type": "Concept",
    "url": "/users/ocladmin/sources/CascadeTest/concepts/BB/325662/",
    "version_url": "/users/ocladmin/sources/CascadeTest/concepts/BB/325662/",
    "terminal": false,
    "entries": [
      {
        "uuid": "325669",
        "id": "03",
        "type": "Concept",
        "url": "/users/ocladmin/sources/CascadeTest/concepts/03/325669/",
        "version_url": "/users/ocladmin/sources/CascadeTest/concepts/03/325669/",
        "terminal": true,
        "entries": [],
        "display_name": "03",
        "retired": false
      },
      {
        "uuid": "325662",
        "id": "BB",
        "type": "Concept",
        "url": "/users/ocladmin/sources/CascadeTest/concepts/BB/325662/",
        "version_url": "/users/ocladmin/sources/CascadeTest/concepts/BB/325662/",
        "terminal": false,
        "entries": [],
        "display_name": "BB",
        "retired": false
      },
      {
        "id": "04",
        "type": "Concept",
        "url": "/users/ocladmin/sources/CascadeTest/concepts/04/325671/",
        "version_url": "/users/ocladmin/sources/CascadeTest/concepts/04/325671/",
        "terminal": true,
        "entries": [],
        "display_name": "04",
        "retired": false
      },
      {
        "id": "11",
        "type": "Mapping",
        "map_type": "Q-AND-A",
        "url": "/users/ocladmin/sources/CascadeTest/mappings/11/693094/",
        "version_url": "/users/ocladmin/sources/CascadeTest/mappings/11/693094/",
        "to_concept_code": "04",
        "to_concept_url": "/users/ocladmin/sources/CascadeTest/concepts/04/",
        "target_concept_code": "04",
        "target_concept_url": "/users/ocladmin/sources/CascadeTest/concepts/04/",
        "target_source_owner": "ocladmin",
        "target_source_name": "CascadeTest",
        "target_concept_name": null,
        "retired": false
      },
      {
        "id": "10",
        "type": "Mapping",
        "map_type": "Q-AND-A",
        "url": "/users/ocladmin/sources/CascadeTest/mappings/10/693092/",
        "version_url": "/users/ocladmin/sources/CascadeTest/mappings/10/693092/",
        "to_concept_code": "03",
        "to_concept_url": "/users/ocladmin/sources/CascadeTest/concepts/03/",
        "target_concept_code": "03",
        "target_concept_url": "/users/ocladmin/sources/CascadeTest/concepts/03/",
        "target_source_owner": "ocladmin",
        "target_source_name": "CascadeTest",
        "target_concept_name": null,
        "retired": false
      },
      {
        "id": "2",
        "type": "Mapping",
        "map_type": "SAME-AS",
        "url": "/users/ocladmin/sources/CascadeTest/mappings/2/693076/",
        "version_url": "/users/ocladmin/sources/CascadeTest/mappings/2/693076/",
        "to_concept_code": "BB",
        "to_concept_url": "/users/ocladmin/sources/CascadeTest/concepts/BB/",
        "target_concept_code": "BB",
        "target_concept_url": "/users/ocladmin/sources/CascadeTest/concepts/BB/",
        "target_source_owner": "ocladmin",
        "target_source_name": "CascadeTest",
        "target_concept_name": null,
        "retired": false
      },
      {
        "id": "16",
        "type": "Mapping",
        "map_type": "SAME-AS",
        "url": "/users/ocladmin/sources/CascadeTest/mappings/16/693104/",
        "version_url": "/users/ocladmin/sources/CascadeTest/mappings/16/693104/",
        "to_concept_code": "166370",
        "to_concept_url": "/orgs/CIEL/sources/CIEL/concepts/166370/",
        "target_concept_code": "166370",
        "target_concept_url": "/orgs/CIEL/sources/CIEL/concepts/166370/",
        "target_source_owner": "CIEL",
        "target_source_name": "CIEL",
        "target_concept_name": "Lateral condyle of humerus",
        "retired": false
      }
    ],
    "display_name": "BB",
    "retired": false
  }
}
```
