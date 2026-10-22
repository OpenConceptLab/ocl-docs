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
- If `version` is specified, that specific source version is used. Otherwise, [version locking](#version-locking) picks the source version, by default the latest released version (or HEAD if the source has none). The `system-version` expansion parameter (see [Expansion Parameters](expansions.md)) can name another one, including HEAD via `<canonical_url>|HEAD`.
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
| `system-version` | Specifies a version to use for a system when the reference does not specify one. Multiple systems can be comma-separated. Takes precedence over the expansion's version lock (see [Version Locking](#version-locking)). |

Parameters are applied in a specific order: `activeOnly` (before filter), then `date`, `exclude-system`, and `filter` (after filters).

See [Expansion Parameters](expansions.md) for the full parameter reference.

### Deduplication

After all include and exclude references are processed, the expansion is deduplicated:
- Concepts are deduplicated by `versioned_object_id`, keeping the most recent version (highest ID).
- Mappings are deduplicated the same way.

This prevents the same logical concept from appearing multiple times when referenced by multiple references that resolve to different versions.

## Version Locking

An expansion keeps every unversioned reference to the same source on one source version. This is **version locking**: the first unversioned reference to a source sets the version, and later unversioned references to that source use the same version instead of resolving on their own. Without it, references added at different times could bring in resources from different releases of the same source.

Version locking applies to source versions, the `system` of a reference. Collection versions named in `valueset` aren't locked: OCL resolves each `valueset` URL on its own.

### How OCL picks a source version

For each reference with a `system`, OCL picks the source version as follows.

1. **A reference that names a source version** (a `version`, or a URL such as `/orgs/MyOrg/sources/MySource/v1/concepts/123/`) uses that version, even if the expansion is locked to another one. A reference pinned to `HEAD` counts. OCL lists the version in `explicit_source_versions`. These references don't lock the source: unversioned references to it still follow rule 3.
2. **A reference pinned to a resource version (deprecated)** with no source version, such as `/orgs/MyOrg/sources/MySource/concepts/123/456/`, uses the source's `$resolveReference` default: its latest released version, or HEAD if it has none. Neither the lock nor the `system-version` parameter applies, and OCL lists the version in `explicit_source_versions`. If that source version doesn't contain the pinned resource version, the reference adds nothing to the expansion. Replace these references with references that name a source version.
3. **An unversioned reference** (no source version and no resource version) uses the first of:
   1. the version of this source named in the expansion's `system-version` parameter;
   2. the version the expansion has already evaluated for this source, its **lock** (see [When a lock carries over](#when-a-lock-carries-over));
   3. the `$resolveReference` default: the source's latest released version, or HEAD if it has none.

   OCL lists the version in `evaluated_source_versions`.

Some details:
- OCL compares sources by their URL without the version, so a canonical URL and a relative URL for the same source share one lock.
- Exclude references follow the same rules as include references.
- OCL resolves each `system-version` entry with `$resolveReference` in the global namespace. An entry that doesn't resolve is listed in `unresolved_repo_versions` with type `expansion.parameters.system-version`, and references to that source skip step 1. If the parameter names more than one version of a source, OCL evaluates each unversioned reference to that source against every one of them.

### When a lock carries over

OCL looks for a lock in two places:
- **In the same evaluation run** (creating an expansion, rebuilding one, or one request that adds references), the version picked for the first unversioned reference to a source is used for every later unversioned reference to it. Any version counts, including HEAD.
- **From earlier runs**, OCL reads the expansion's `evaluated_source_versions` and uses the most recently created version of the source that is released, active and not retired. So a lock doesn't carry over to references added later if it is on HEAD (the source had no release when the first reference was added), or on a release since retired, deactivated or marked as not released.

The new reference then gets the `$resolveReference` default, normally the source's latest release. If that differs from the old lock, the expansion lists both versions of the source in `evaluated_source_versions`, since the old one stays. Rebuild the expansion to put every unversioned reference back on one version.

### Creating, rebuilding and adding references

- **Creating an expansion** (`POST .../expansions/`): OCL evaluates every reference with the rules above, starting with no lock.
- **Rebuilding an expansion** ([re-evaluate](expansions.md), or *Rebuild* in TBv3): OCL clears the expansion, including its lists, and evaluates every reference again. The lock starts over, so unversioned references move to the source's current latest release, or to the `system-version` version. This is how an expansion moves to a new release. To compare first, create a similar expansion and look at it before you rebuild.
- **Adding references to a collection** updates only the collection version's default expansion. Its other expansions keep their contents until they're rebuilt. What happens in the default expansion depends on its kind:
  - A **named expansion** (one created with its own mnemonic and set as the default) evaluates the new references, plus the collection's exclude references, with the rules above. New unversioned references follow the lock, where it carries over.
  - An **auto-expansion** (`autoexpand-HEAD`, which OCL keeps up to date for a collection that auto-expands) doesn't apply the lock to new references. It uses the result OCL stored when it saved each reference (from the reference's own version, or the `$resolveReference` default), yet records the locked version in `evaluated_source_versions`. So after a source's new release, a new unversioned reference brings in resources from the new release while the list shows only the old one. Rebuild the auto-expansion to put every reference on one version.
- **Creating a collection version** that auto-expands copies HEAD's references, with the results OCL stored for them, into the new version's auto-expansion without evaluating them again. Its `evaluated_source_versions` shows the versions the rules pick at that moment, normally each source's latest release, even where members came from the release a reference resolved to when it was added. Rebuild it to make the two match.

### Repository version lists

`GET .../expansions/:expansion/` returns the repository versions an expansion used:

| Field | What it lists |
|---|---|
| `explicit_source_versions` | Source versions used by references that name a source version, and by deprecated resource-version references. |
| `evaluated_source_versions` | Source versions OCL picked for unversioned references (see [When a lock carries over](#when-a-lock-carries-over)). |
| `explicit_collection_versions` | Collection versions named by `valueset` URLs that include a version (`url|v1` or `.../collections/MyValueSet/v1/`). |
| `evaluated_collection_versions` | Collection versions resolved for `valueset` URLs without a version. |
| `unresolved_repo_versions` | Repositories OCL couldn't resolve, each with a `url` and a `type`: `reference.system`, `reference.valueset` or `expansion.parameters.system-version`. It holds the entries from the latest evaluation that had any, and a rebuild clears it. |

A version appears in both source lists when some references name it and unversioned references resolve to it as well. For example, if one reference pins `v2` and an unversioned reference resolves to the latest release, also `v2`, then `v2` is in `explicit_source_versions` and `evaluated_source_versions`.

The lists record the versions OCL used, not where the current members come from. Between rebuilds they only grow, and a version stays listed when no member comes from it any more: its references were removed, its resources were excluded, or an expansion parameter such as `date` filtered them all out.

Two other views show related information:
- **A reference on its own.** `GET .../references/?includeResolvedRepoVersions=true` adds `resolved_repo_versions` to each reference: the repository version the reference resolves to by itself, right now, through `$resolveReference`. It ignores the expansion's lock and parameters, so for an unversioned reference it can show a newer release than the one the expansion used. TBv3 shows it as *Resolved Repo* on a collection's References tab.
- **Newer releases.** `GET .../expansions/:expansion/resolved-repo-updates/` lists the expansion's repository versions for which `$resolveReference` now picks a different version (see [Expansions](expansions.md)). TBv3 uses it to show "New releases are available for some resolved repo versions in this expansion".

In TBv3, the expansion details on a collection's Versions tab show the expansion's parameters and two lists, *Explicit Repo Versions* and *Evaluated Repo Versions*. Each list combines source and collection versions. TBv2 shows the same two lists under *Resolved Repo Versions*.

### Expansions built before version locking

Version locking shipped in oclapi2 2.3.201. The upgrade didn't recompute existing expansions. It moved every repository version they had recorded into `explicit_source_versions` and `explicit_collection_versions`, including versions picked for unversioned references, and left the evaluated lists empty. In such an expansion:
- there's no lock to carry over, because OCL reads locks only from `evaluated_source_versions`. The next unversioned reference added resolves to the latest release and becomes the lock;
- members can come from several versions of one source, because each addition resolved on its own before version locking.

Before oclapi2 2.3.215, a rebuild didn't start over either: it kept the existing members, lists and lock and added to them. An expansion last rebuilt before then can also hold several versions of one source.

**How to tell.** In the expansion's details (TBv3's Versions tab, or `GET` the expansion), look for:
- an empty `evaluated_source_versions` in a collection version that has unversioned references;
- a source with more than one version across the two lists, beyond what its version-pinned references explain;
- both lists empty in an expansion that has members, which means it was built before OCL recorded repository versions at all and hasn't been rebuilt since.

The lists can't show the auto-expansion case described above, where members come from a newer release than the one listed. If references were added to an auto-expansion after a source's new release, rebuild it.

**How to fix.** Rebuild the expansion (see above). To stay on an older release instead, create an expansion whose `system-version` names that release, or pin the references to it.

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

**Dynamic references** have none of the above. They resolve through [version locking](#version-locking), so they can return different resources when an expansion moves to a newer source version, for example on a rebuild.

## Practical Examples

### Example 1: Single concept by code
```json
{
    "system": "/orgs/CIEL/sources/CIEL/",
    "code": "1948",
    "resourceType": "Concept"
}
```
**Pipeline:** Resolves the CIEL source version picked by [version locking](#version-locking) (the latest release by default) -> filters to concept with mnemonic "1948" -> returns that concept as it is in that source version.

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
**Pipeline:** Resolves the ICD-10-WHO source version picked by [version locking](#version-locking) -> resolves common-diagnoses ValueSet v1.0 -> intersects resources from both -> removes matching concepts from the expansion.

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
