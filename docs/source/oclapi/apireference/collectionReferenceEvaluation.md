# Collection Reference Evaluation Logic

## Overview

When a reference is added to an OCL collection, it goes through an **evaluation pipeline** to resolve the reference expression into a concrete set of concepts and/or mappings. This document describes how that pipeline works end-to-end.

A collection reference is a declarative instruction — it describes *what* resources should be in a collection, not the resources themselves. The evaluation pipeline translates these instructions into actual concept and mapping resources that are stored in the collection's **expansion**.

### Key Concepts

- **Reference**: A declarative rule stored on a collection that describes which concepts/mappings to include or exclude. References persist as part of the collection definition.
- **Expansion**: The evaluated result of processing all references. An expansion contains the actual concepts and mappings. A collection version can have multiple expansions, each evaluated with different parameters.
- **System**: The source repository (CodeSystem) from which resources are drawn. Can be a relative URL (e.g. `/orgs/CIEL/sources/CIEL/`) or a canonical URL (e.g. `https://CIELterminology.org`).
- **ValueSet**: One or more collections whose expansions constrain which resources are included. When both `system` and `valueset` are specified, the result is their intersection.

## Reference Fields

Each `CollectionReference` has the following fields that control evaluation:

| Field | Type | Description |
|---|---|---|
| `expression` | string | The reference expression — typically a relative or canonical URL. Can be auto-built from other fields. |
| `reference_type` | string | `"concepts"` or `"mappings"` — determines which resource type is fetched. |
| `include` | boolean | `true` (default) to include resources; `false` to exclude them from the expansion. |
| `system` | string | Relative or canonical URL of the source repository (CodeSystem). |
| `version` | string | Version of the source repository to use. |
| `valueset` | string[] | List of collection (ValueSet) URLs whose expansions constrain the result. |
| `namespace` | string | Namespace context for resolving canonical URLs (see `$resolveReference`). |
| `code` | string | A specific concept mnemonic or mapping ID. |
| `resource_version` | string | Pin to a specific version of the concept or mapping. |
| `filter` | JSON[] | List of property-based filter criteria (see [Filters](#filters)). |
| `cascade` | string/JSON | Cascade configuration for graph traversal (see [Cascade](#cascade)). |
| `transform` | string | Version resolution strategy: `"resourceversions"` or `"extensional"` (see [Transforms](#transforms)). |
| `display` | string | Overrides the default display name for a concept (only with `code`). |

## Evaluation Pipeline

When a reference is evaluated, it passes through these stages in order:

```
Reference
    |
    v
1. System/ValueSet Resolution  -->  Base queryset of resources
    |
    v
2. Code Filter                 -->  Filter to specific concept/mapping by code
    |
    v
3. Search Filter               -->  Apply property-based search filters via Elasticsearch
    |
    v
4. Cascade                     -->  Traverse graph to include related mappings/concepts
    |
    v
5. Transform                   -->  Apply version resolution strategy
    |
    v
Evaluated concepts + mappings
```

### Stage 1: System/ValueSet Resolution

The first stage resolves the `system` and `valueset` fields into a base queryset of resources.

**System resolution:**
- If `system` is a relative URL (e.g. `/orgs/CIEL/sources/CIEL/`), it resolves directly to the source repository.
- If `system` is a canonical URL (e.g. `https://CIELterminology.org`), it is resolved using the `$resolveReference` operation, which considers namespace-specific and global Canonical URL Registries.
- If `version` is specified, that specific source version is used. Otherwise, HEAD is used.
- The base queryset is the set of all concepts (or mappings) in the resolved source version.
- For HEAD without a pinned resource version: if `transform` is `"resourceversions"`, the latest version of each resource is selected; otherwise, the HEAD (versioned_object) is selected.

**ValueSet resolution:**
- Each URL in the `valueset` array is resolved to a collection version.
- The resources from each resolved collection's expansion are intersected.

**Combined:**
- If both `system` and `valueset` are specified, the result is their intersection — only resources that appear in both the source and all valueset expansions.

**Permissions:**
- The reference creator must have view access to the resolved source/collection. If access is denied, the queryset is empty.

### Stage 2: Code Filter

If the `code` field is set, the base queryset is filtered to match:
- For concepts: `mnemonic = code`
- For mappings: `mnemonic = code`

If `resource_version` is also set, the result is pinned to that exact version. Otherwise, if multiple versions match, the latest version (highest ID) is selected.

**Note:** The `code` field and `filter` field are mutually exclusive — if `code` is set, filters are skipped.

### Stage 3: Search Filters {#filters}

If no `code` is specified and the `filter` field is populated, property-based filters are applied using Elasticsearch.

**Filter schema:**
Each filter in the array must be a JSON object with three string fields:
```json
{
    "property": "<field name>",
    "op": "<operator>",
    "value": "<value>"
}
```

**Supported operators:** `=` (equal), `in`

**Supported properties:**
- For concepts: All Elasticsearch-indexed concept fields (e.g. `concept_class`, `datatype`, `locale`, `retired`, `source`, `owner`, `name`, `description`, etc.) plus `extras.*` fields
- For mappings: All Elasticsearch-indexed mapping fields (e.g. `map_type`, `from_concept_code`, `to_concept_code`, `source`, `owner`, etc.) plus `extras.*` fields
- Special properties:
  - `q` — full-text search across display names, codes, and descriptions
  - `exact_match` — exact phrase matching
  - `exclude_wildcard` — set to a falsey value to enable wildcard matching
  - `exclude_fuzzy` — set to a falsey value to enable fuzzy matching
  - `search_map_codes` — set to a falsey value to exclude map codes from search
  - `include_search_meta` — include search metadata

**Filter evaluation:**
1. An Elasticsearch search is built from the filter criteria.
2. The search is intersected with the base queryset (processed in batches of 500 to respect Elasticsearch clause limits).
3. If version is specified or transform is set, the versioned resources are returned directly. Otherwise, the HEAD resources for the matching versioned_object_ids are returned.

### Stage 4: Cascade {#cascade}

If the `cascade` field is set, OCL performs graph traversal starting from each concept in the current queryset, adding related mappings and optionally their target concepts.

**Prerequisite:** `cascade` requires either `code` or `filter` to be specified — a cascade without an initial set of concepts would attempt to traverse the entire repository.

**Simple cascade (string value):**
- `"sourcemappings"` — Include all mappings in the same source where each concept is the from-concept.
- `"sourcetoconcepts"` — Include mappings (as above) AND their target concepts (full closure).

When cascade is a simple string, `cascade_levels` defaults to `1` (single hop).

**Extended cascade (JSON object):**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `method` | string | required | `"sourcemappings"` or `"sourcetoconcepts"` |
| `cascade_levels` | int/string | `"*"` (all) | Number of levels to traverse. `"*"` means traverse until no new resources are found. |
| `source_version` | string | — | Specific source version for cascade context. Falls back to the reference's resolved system version. |
| `cascade_mappings` | boolean | `true` | Whether to include mappings in the cascade result. |
| `cascade_hierarchy` | boolean | `true` | Whether to traverse hierarchical relationships. |
| `reverse` | boolean | `false` | Reverse the cascade direction (traverse from to-concept to from-concept). |
| `map_types` | string[] | — | Only include mappings of these types (e.g. `["SAME-AS", "NARROWER-THAN"]`). |
| `exclude_map_types` | string[] | — | Exclude mappings of these types. |
| `return_map_types` | string[] | `"*"` (all) | Which map types to include in the returned mappings. |
| `max_results` | int | `1000` | Maximum number of resources to return from cascade. Set to `null` for unlimited (used in async mode). |
| `include_retired` | boolean | `false` | Whether to include retired concepts in the cascade. |
| `omit_if_exists_in` | string | — | Omit concepts that already exist in the specified collection/source version. |

**Cascade traversal:**
1. For each concept in the current queryset (deduplicated by URI to avoid redundant traversal):
2. Call `concept.cascade()` with the computed cascade parameters.
3. The cascade performs iterative graph traversal:
   - At each level, find mappings matching the criteria for each not-yet-cascaded concept.
   - If `source_to_concepts` is enabled, also include the target concepts of those mappings.
   - If `cascade_hierarchy` is enabled, also traverse hierarchical relationships.
   - Continue iterating until `cascade_levels` is reached or no new resources are found.
4. Merge all discovered concepts and mappings into the result sets.

### Stage 5: Transforms {#transforms}

Transforms control how resource versions are resolved in the final result:

- **`"resourceversions"`** (static/intensional): Converts HEAD resources to their latest released version. This means the reference will always resolve to a specific, immutable version of each resource. If the concept/mapping was already versioned, it keeps that version.
- **`"extensional"`**: For non-HEAD source versions, converts versioned resources to their HEAD (versioned_object) equivalent. This makes the reference track the latest state of each resource.

If neither transform is specified, the `transform` field is set to `null`.

**Transform interactions with other fields:**
- When `transform` is `"resourceversions"` and `code` is set: After transforming, the reference's `version` field is updated to the source version containing the latest resource version, and the `expression` is updated to the resource's URI.
- When `transform` is `"extensional"` and the source version is not HEAD: Resources are resolved to their HEAD equivalents.

## Expansion Processing

When references are added to an expansion (via `Expansion.add_references()`), the following process occurs:

### Include/Exclude Processing Order

1. **Include references** are processed first: Each include reference is evaluated, and its resulting concepts/mappings are added to the expansion.
2. **Exclude references** are processed second: Each exclude reference is evaluated, and matching concepts/mappings are removed from the expansion.
3. When processing excludes, all existing exclude references on the collection are also applied — not just the newly added ones.

### Expansion Parameters

After each reference is evaluated, **expansion parameters** are applied to filter the results before adding them to the expansion. The implemented parameters are:

| Parameter | Description |
|---|---|
| `activeOnly` | If `true`, exclude retired resources (applied as a database filter). |
| `filter` | Text filter to restrict resources by display name or code (applied via Elasticsearch). |
| `date` | Only include resources that existed as of this date (applied via date comparison). |
| `exclude-system` | Exclude resources from the specified code system (applied via source URI matching). |
| `system-version` | Specifies a version to use for a system when the reference does not specify one. Multiple systems can be comma-separated. |

Parameters are applied in a specific order: `activeOnly` (before filter), then `date`, `exclude-system`, and `filter` (after filters).

See [Expansion Parameters](expansions.md) for the full parameter reference.

### Version Tracking

The expansion tracks which source and collection versions were used during evaluation:

| Tracking Field | Description |
|---|---|
| `explicit_source_versions` | Source versions that were explicitly specified in references (e.g. version pinned). |
| `evaluated_source_versions` | Source versions that were resolved at evaluation time (versionless references). |
| `explicit_collection_versions` | Collection (ValueSet) versions explicitly specified in references. |
| `evaluated_collection_versions` | Collection versions resolved at evaluation time. |
| `unresolved_repo_versions` | References that could not be resolved to any repository version, stored as JSON with `url`, `namespace`, and `type` fields. |

This tracking enables:
- Auditing which exact versions were used in an expansion
- Re-evaluation of versionless references against updated source versions
- Identification of broken references

### Deduplication

After all include and exclude references are processed, the expansion is deduplicated:
- Concepts are deduplicated by `versioned_object_id`, keeping the most recent version (highest ID).
- Mappings are deduplicated the same way.

This prevents the same logical concept from appearing multiple times when referenced by multiple references that resolve to different versions.

### Re-evaluation

When `add_references()` is called, references may be re-evaluated depending on the context:
- **Auto-generated expansions** (created automatically for HEAD) are not re-evaluated when adding new references — they use cached results.
- **Named expansions** (manually created) trigger re-evaluation of all references using the expansion's parameters.
- **Force re-evaluation** can be triggered by the `force_reevaluate` flag, which always re-evaluates regardless of expansion type.

## Human-Readable Translation

Each reference can be translated into a human-readable English description via the `translation` field. Examples:

| Reference | Translation |
|---|---|
| Include concept "1948" from CIEL | `Include latest concept "1948" from CIEL/CIEL` |
| Include concept "1948" version "v1" from CIEL | `Include version "v1" of concept "1948" from CIEL/CIEL` |
| Include concepts with concept_class = "Diagnosis" | `Include latest concepts having concept_class equal to "Diagnosis"` |
| Include concepts from CIEL with cascade to concepts | `Include latest concepts from CIEL/CIEL PLUS its mappings and their target concepts` |
| Exclude concepts from CIEL containing "malaria" | `Exclude latest concepts from CIEL/CIEL containing "malaria"` |
| Include concepts from system intersected with valueset | `Include latest concepts from https://example.org intersection with MyOrg/MyValueSet` |

## Static vs. Dynamic References

References can be classified as **static** (always resolve to the same resources) or **dynamic** (may resolve differently over time):

**Static references** have one of:
- A pinned `resource_version` (e.g. concept version "abc123")
- A `transform` set (locks to a specific version strategy)
- Both `code` and `version` set (specific concept in a specific source version)

**Dynamic references** have none of the above. They resolve against HEAD and may return different resources as the source is updated.

## Practical Examples

### Example 1: Single concept by code
```json
{
    "system": "/orgs/CIEL/sources/CIEL/",
    "code": "1948",
    "resourceType": "Concept"
}
```
**Pipeline:** Resolves CIEL source HEAD -> filters to concept with mnemonic "1948" -> returns latest version of that concept.

### Example 2: Filtered concepts with cascade
```json
{
    "system": "https://CIELterminology.org",
    "version": "v2023-03-01",
    "filter": [
        {"property": "concept_class", "op": "=", "value": "Diagnosis"}
    ],
    "cascade": {
        "method": "sourcetoconcepts",
        "cascade_levels": 1,
        "map_types": ["SAME-AS"]
    },
    "resourceType": "Concept"
}
```
**Pipeline:** Resolves CIEL source version v2023-03-01 -> Elasticsearch query for concept_class=Diagnosis -> for each matching concept, find SAME-AS mappings and their target concepts (1 level) -> returns all discovered concepts and mappings.

### Example 3: Exclude with valueset intersection
```json
{
    "system": "/orgs/WHO/sources/ICD-10-WHO/",
    "valueset": ["https://example.org/ValueSet/common-diagnoses|v1.0"],
    "include": false,
    "resourceType": "Concept"
}
```
**Pipeline:** Resolves ICD-10-WHO source HEAD -> resolves common-diagnoses ValueSet v1.0 -> intersects resources from both -> removes matching concepts from the expansion.

### Example 4: Transform to latest released versions
```json
{
    "system": "/orgs/CIEL/sources/CIEL/",
    "code": "1948",
    "cascade": "sourcetoconcepts",
    "transform": "resourceversions",
    "resourceType": "Concept"
}
```
**Pipeline:** Resolves CIEL source HEAD -> filters to concept "1948" -> cascades to mappings and target concepts -> transforms all HEAD resources to their latest released versions -> generates individual static references for each discovered resource.

## Related Documentation

- [$resolveReference](resolveReference.md) — How canonical and relative URLs are resolved to repository versions
- [Collections API Reference](collections.md) — Collection CRUD operations and add/remove references API
- [Expansions](expansions.md) — Expansion CRUD operations and parameter reference
- [Cascade](cascade.md) — Additional cascade documentation
