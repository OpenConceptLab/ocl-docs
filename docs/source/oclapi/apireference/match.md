# Operation: $match

## Overview

The `$match` endpoint allows you to find similar or matching concepts across different repositories in OCL. While search returns concepts that match a specific search query, `$match` returns concept candidates that match structured input data, such as a row in a spreadsheet. This could be used to retrieve mapping candidates for an entire spreadsheet.

`$match` API must accept POST (GET is not supported).

### Authentication & Access

- **Authentication required.** The user must be signed in.
- The user must be **mapper-approved** and not on the mapper waitlist. Returns `403 Forbidden` otherwise.
- Plan-based throttling is applied per user.

### $match Algorithm Fields
- `id` - Exact match on `concept.id` in the target repository
- `name` - Keyword or semantic search on concept primary display name
- `synonyms` - Keyword or semantic search on all concept names and synonyms
- `description` - String search on all concept descriptions
- Properties: String match on concept properties
  - Syntax: `property:<property-name>` where `<property-name>` corresponds with `repo.properties.code`
  - Note: A property must also be defined as a filter (i.e. `repo.filters`) in order for it to be indexed and searchable
  - There are two special cases - for historical compatibility, class and datatype are stored as core concept attributes:
    - `property:class` - String match on `concept.concept_class`
    - `property:datatype` - String match on `concept.datatype`
  - Examples:
    - `property:component` - String match on LOINC's "component" property
    - `property:units` - String match on a "units" property
- `mapping:code` - Matches concepts in the target repo that share a mapping with the input row. For example, the input row and target concept share a mapping to the same LOINC code.
- `mapping:list` - Matches concepts in the target repo that share a mapping, where the input is a list of mappings for the row.

## Request
```
POST /concepts/$match/
```

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `verbose` | Boolean | No | `false` | More details in results (full concept details) |
| `brief` | Boolean | No | `false` | Return minimal concept fields in results |
| `limit` | Integer | No | `1` | Number of results to be returned per row (page size) |
| `offset` | Integer | No | `0` | Number of results to skip |
| `page` | Integer | No | `1` | Page number for paginated results |
| `includeRetired` | Boolean | No | `false` | Match against retired concepts as well |
| `bestMatch` | Boolean | No | `false` | Forces a minimum search score threshold to be applied, filtering out low-quality matches |
| `semantic` | Boolean | No | `false` | Use semantic (LM-based) matching algorithm. The target repository must have the semantic match algorithm enabled. |
| `numCandidates` | Integer | No | `3000` | Only needed when semantic=true. Number of approximate nearest neighbor candidates. Range: 1 to 3000. For more information: https://www.elastic.co/guide/en/elasticsearch/reference/current/knn-search.html#tune-approximate-knn-for-speed-accuracy |
| `kNearest` | Integer | No | `100` | Only needed when semantic=true. Number of nearest neighbors to return from vector search. Range: 1 to 100. |
| `encoder_model` | String | No | (none) | Custom encoder model name for semantic vector search |
| `reranker` | Boolean | No | `false` | Enable cross-encoder reranking of results. Only applicable when semantic=true. |

### Request Body Schema

| **Code (Name)**                | **Card.** | **Type**             | **Definition (Description)**                                                                                                                                                                                         |
| ------------------------------ | --------- | -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `target_repo_url`              | 0..1      | string               | Repository URL to match against. Uses `$resolve` to identify the specific repo version. Either `target_repo_url` or `target_repo` must be provided. **Example:** `/orgs/CIEL/sources/CIEL/`                           |
| `target_repo`                  | 0..1      | object               | Alternative to `target_repo_url`. Object with `owner`, `source`, `source_version`, and `owner_type` fields. Either `target_repo_url` or `target_repo` must be provided.                                              |
| `rows`                         | 1..\*     | list      | List of concept-like key-value pairs; each row may have different fields. Only fields that are recognized by the matching algorithm are used, all other fields are ignored. **Example:** `[{"s_n":"1","name":"malaria"},{"s_n":"2","name":"blood type"}]`                                                            |
| `rows.id`                      | 0..1      | string               | Exact match against a concept ID. *(May be removed in future versions.)* **Example Input Data:** `12`, `57`, `A01.1`                                                                                                 |
| `rows.name`                    | 0..1      | string               | Semantic or fuzzy search on primary display name. **Example Input Data:** `Anemia due to blood loss`                                                                                                                 |
| `rows.synonyms`                | 0..1     | string or list       | Semantic or fuzzy search across all names/synonyms. Accepts a single string or a list of strings. **Example Input Data:** `"Anemia, blood loss"` or `["Anemia, blood loss", "Anémie secondaire à une hémorragie"]`    |
| `rows.description`             | 0..1      | string               | Text search on concept descriptions. **Example Input Data:** `"Anemia due to bleeding or a hemorrhagic process"`                                                                                                     |
| `rows.concept_class`           | 0..1      | string               | Match on concept class (e.g., diagnosis, symptom). **Example Input Data:** `Diagnosis`, `Symptom`                                                                                                                    |
| `rows.datatype`                | 0..1      | string               | Match on datatype (e.g., numeric, coded). **Example Input Data:** `Numeric`, `Coded`                                                                                                                                 |
| `rows.mapping_code`            | 0..1      | string               | Exact match on a concept ID or mapping in the target repo version. **Example Input Data:** `D50.0`, `Z87.5`, `X59.9`                                                                                                 |
| `rows.mapping_list`            | 0..1      | string               | Exact match on comma-separated mapping list. *(In development.)* **Example Input Data:** `CIEL:1858, ICD10:DC14.Z, LOINC:5792-7`                                                                                     |
| `rows.same_as_map_codes`       | 0..1      | string               | Search only "same as" mappings. *(Deprecated.)* **Example Input Data:** `CIEL:1858, ICD10:DC14.Z, LOINC:5792-7`                                                                                                      |
| `rows.other_map_codes`         | 0..1      | string               | Search all non-"same as" mappings. *(Deprecated.)* **Example Input Data:** `CIEL:1858, ICD10:DC14.Z, LOINC:5792-7`                                                                                                   |
| `filter`                       | 0..1      | object               | Filtering criteria. Supports `locale` (string, comma-separated locale codes for semantic search) and faceted filters matching the target repository's filter definitions.                                              |
| `map_config`                   | 0..\*     | list      | Optional list configuring mapping logic per row. **Example from Request Body:** see below.                                                                                                                           |
| `map_config.type`              | 1..1      | code                 | Type of mapping: `mapping-code` or `mapping-list`. **Example:** `mapping-code`, `mapping-list`                                                                                                                       |
| `map_config.input_column`      | 1..1      | string               | Name of the row-field to use. **Example:** `loinc-example`, `icd10-example`, `list example`                                                                                                                          |
| `map_config.target_source_url` | 0..1      | string               | Target repo URL for `mapping-code` entries (required if type is `mapping-code`). **Example:** `/orgs/CIEL/sources/CIEL/`                                                                                             |
| `map_config.separator`         | 0..1      | string               | Separator between source name and code in `mapping-list`. **Example:** `:`                                                                                                                                           |
| `map_config.delimiter`         | 0..1      | string               | Delimiter for multiple mappings in `mapping-list`. **Example:** `,`                                                                                                                                                  |
| `map_config.target_urls`       | 0..1      | map                  | URL map of source mnemonics to repositories. Required for `mapping-list`. **Example:** `{"ICD10": "/orgs/WHO/sources/ICD-10-WHO/", "CIEL": "/orgs/CIEL/sources/CIEL/", "LOINC": "/orgs/Regenstrief/sources/LOINC/"}` |


## Response Format

| **Code (Name)**                | **Card.** | **Type**             | **Definition (Description)**   |
| ------------------------------ | --------- | -------------------- | ------- |
| _\<base\>_ | 1 | list | A list of response objects, one per input row |
| row | 1 | map | The original row submitted, with no alteration |
| results | 1..* | list | Ordered list of concept candidates, sorted by score |
| results.url | 1 | string | Concept URL |
| results.display_name | 1 | string | Primary display name |
| results.id | 1 | string | Concept ID |
| results.retired | 0..1 | bool | Whether the concept is retired |
| results.concept_class | 0..1 | string | Concept class |
| results.datatype | 0..1 |  string | Concept datatype |
| results.property | 0..* | list | Concept properties |
| results.property.code | 1 | string | The key of the property (e.g. concept_class) |
| results.property.valueCode | 0..1 | string |  |
| results.property.valueCoding | 0..1 | ... |  |
| results.property.valueString | 0..1 | string |  |
| results.property.valueInteger | 0..1 | int |  |
| results.property.valueBoolean | 0..1 | bool |  |
| results.property.valueDateTime | 0..1 | DateTime |  |
| results.property.valueDecimal | 0..1 | decimal |  |
| results.extras | 0..1 | map | Additional concept attributes |
| results.search_meta | 1 | object | Search metadata (always included for $match) |
| results.search_meta.search_score | 1 | decimal | Raw Elasticsearch score |
| results.search_meta.search_normalized_score | 1 | decimal | Normalized score on a 0-100 scale |
| results.search_meta.search_rerank_score | 0..1 | decimal | Cross-encoder rerank score (only present when `reranker=true`) |
| results.search_meta.match_type | 1 | string | Match confidence: `very_high`, `high`, `medium`, or `low` (see below) |
| results.search_meta.algorithm | 1 | string | Algorithm used: `ocl-search`, `ocl-semantic`, or `ocl-ciel-bridge` |
| results.search_meta.search_highlight | 0..1 | map | Highlighted matching fields (e.g. `name`, `synonyms`) |
| results.search_meta.search_confidence | 0..1 | string | Confidence value (may be null) |
| results.source | 0..1 | string | Source mnemonic |
| results.owner | 0..1 | string | Owner mnemonic |
| results.owner_type | 0..1 | string | Owner type (e.g. Organization) |
| results.owner_url | 0..1 | string | Owner URL |
| results.mappings | 0..1 | list | Concept mappings (when verbose) |
| results.names | 0..1 | list | Concept names (when verbose) |
| map_config | 0..1 | list | Echo of the `map_config` from the request |
| filter | 0..1 | object | Echo of the `filter` from the request |

### Match Type

The `match_type` field indicates the confidence level of each result. How it is determined depends on the search mode:

**When `limit=1`:** Always `very_high` (single best result is returned).

**When `limit > 1` with semantic search + reranker:**
- `very_high` - Normalized score >= 0.9
- `high` - Normalized score >= 0.65
- `medium` - Normalized score >= 0.5
- `low` - Normalized score < 0.5

**When `limit > 1` with semantic search (no reranker):**
- `very_high` - Name appears in highlights, or normalized score >= threshold (0.9)
- `high` - Synonyms appear in highlights
- `medium` - Any field appears in highlights
- `low` - No highlights

**When `limit > 1` with token-based (default) search:**
- `very_high` - Name appears in highlights
- `high` - Synonyms appear in highlights
- `medium` - Any field appears in highlights
- `low` - No highlights

When `bestMatch=true`, results with `low` match type are excluded.

### Notes

- **`search_meta` is always included** in `$match` responses. There is no need to pass `includeSearchMeta=true`.
- **CIEL bridge algorithm:** When the target repo is CIEL and a `filter.target_repo` points to a non-CIEL repo, the `ocl-ciel-bridge` algorithm is used.
- **Pagination** via `offset`/`limit` or `page`/`limit` applies to each row's result set independently.

### Example Response
```json
[
    {
        "row": {"local_id": "1396", "name": "malaria"},
        "results": [
            {
                "search_meta": {
                    "search_score": 18.534,
                    "search_normalized_score": 95.2,
                    "match_type": "very_high",
                    "algorithm": "ocl-semantic",
                    "search_confidence": null,
                    "search_highlight": {
                        "name": ["<em>Malaria</em>"]
                    }
                },
                "id": "116128",
                "url": "/orgs/CIEL/sources/CIEL/concepts/116128/",
                "retired": false,
                "source": "CIEL",
                "owner": "CIEL",
                "owner_type": "Organization",
                "owner_url": "/orgs/CIEL/",
                "display_name": "Malaria",
                "display_locale": "en"
            }
        ],
        "map_config": [],
        "filter": {}
    }
]
```

### Example Request 1: Simple Request
```
POST https://api.openconceptlab.org/concepts/$match/?semantic=true&bestMatch=true&limit=1
```
```json
{
    "rows": [
        {"local_id": "1396", "name": "malaria"},
        {"local_id": "2", "name": "a1c"}
    ],
    "target_repo_url": "/orgs/CIEL/sources/CIEL/"
}
```

### Example Request 2: With mapping configuration
```
POST https://api.openconceptlab.org/concepts/$match/?limit=3
```
```json
{
    "rows": [
        {"local_id": "1396", "name": "Mother's HIV Status", "loinc_code": "75179-2"},
        {"local_id": "2", "name": "Weeks of gestation", "loinc_code": "11884-4"}
    ],
    "target_repo_url": "/orgs/Regenstrief/sources/LOINC/2.71.21AA/",
    "map_config": [
        {"type": "mapping-code", "input_column": "loinc_code", "target_source_url": "/orgs/CIEL/sources/CIEL/"},
        {"type": "mapping-list", "input_column": "maps", "separator": ":", "delimiter": ",", "target_urls": {
            "ICD10": "/orgs/WHO/sources/ICD-10-WHO/",
            "CIEL": "/orgs/CIEL/sources/CIEL/",
            "LOINC": "/orgs/Regenstrief/sources/LOINC/"
        }}
    ]
}
```

### Example Request 3: Semantic search with reranker and locale filter
```
POST https://api.openconceptlab.org/concepts/$match/?semantic=true&reranker=true&limit=5
```
```json
{
    "rows": [
        {"name": "Type 2 diabetes mellitus", "synonyms": ["Adult-onset diabetes", "Non-insulin dependent diabetes"]}
    ],
    "target_repo_url": "/orgs/CIEL/sources/CIEL/",
    "filter": {
        "locale": "en,fr"
    }
}
```
