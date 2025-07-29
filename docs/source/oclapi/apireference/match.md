# Operation: $match (Experimental)

## Overview

The `$match` endpoint allows you to find similar or matching concepts across different repositories in OCL. While search returns concepts that match a specific search query, `$match` returns concept candidates that match structured input data, such as a row in a spreadsheet. This could be used to retrieve mapping candidates for an entire spreadsheet.

`$match` API must accept POST (GET is not supported).

### $match Algorithm Fields
- `id` - Exact match on concept ID in the target repository
- `name` - Keyword or semantic search on primary display names
- `synonyms` - Keyword or semantic search on all synonyms
- `description` - String search on concept descriptions
- Properties: String match on defined properties in the target repository
  - `Property: Class` - String match on `concept_class`
  - `Property: Datatype` - String match on `datatype`
- `Mapping: Code` - Matches concepts in the target repo that share a mapping with the input row. For example, the input row and target concept share a mapping to the same LOINC code.
- `Mapping: List` - Matches concepts in the target repo that share a mapping, where the input is a list of mappings for the row.

## Request
```
POST /concepts/$match/
```

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `verbose` | Boolean | No | `false` | More details in results (concept details) |
| `limit` | Integer | No | `1` | Number of results to be returned or page size |
| `offset` | Integer | No | `0` | Number of results to skip |
| `page` | Integer | No | `1` | Page number for paginated results |
| `includeRetired` | Boolean | No | `false` | Match against retired concepts as well |
| `bestMatch` | Boolean | No | `false` | Forces a minimum search score threshold to be applied |
| `semantic` | Boolean | No | `false` | Use LM algo for matching |
| `numCandidates` | Integer | No | `5000` | Only needed when semantic=true. Range: 1 to 5000. For more information: https://www.elastic.co/guide/en/elasticsearch/reference/current/knn-search.html#tune-approximate-knn-for-speed-accuracy |
| `kNearest` | Integer | No | `5` | Only needed when semantic=true. Range: 1 to 10 |

### Request Body Schema

| **Code (Name)**                | **Card.** | **Type**             | **Definition (Description)**                                                                                                                                                                                         |
| ------------------------------ | --------- | -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `target_repo_url`              | 1..1      | string               | Repository URL to match against. Uses `$resolve` to identify the specific repo version. **Example:** `/orgs/CIEL/sources/CIEL/`                                                                                      |
| `rows`                         | 1..\*     | list      | List of concept-like key-value pairs; each row may have different fields. Only fields that are recognized by the matching algorithm are used, all other rows are ignored. **Example:** `[{"s_n":"1","name":"malaria"},{"s_n":"2","name":"blood type"}]`                                                            |
| `rows.id`                      | 0..1      | string               | Exact match against a concept ID. *(May be removed in future versions.)* **Example Input Data:** `12`, `57`, `A01.1`                                                                                                 |
| `rows.name`                    | 0..1      | string               | Semantic or fuzzy search on primary display name. **Example Input Data:** `Anemia due to blood loss`                                                                                                                 |
| `rows.synonyms`                | 0..1     | string               | Semantic or fuzzy search across all names/synonyms. **Example Input Data:** `"Anemia, blood loss", "Anémie secondaire à une hémorragie"`                                                                           |
| `rows.description`             | 0..1      | string               | Text search on concept descriptions. **Example Input Data:** `"Anemia due to bleeding or a hemorrhagic process"`                                                                                                     |
| `rows.concept_class`           | 0..1      | string               | Match on concept class (e.g., diagnosis, symptom). **Example Input Data:** `Diagnosis`, `Symptom`                                                                                                                    |
| `rows.datatype`                | 0..1      | string               | Match on datatype (e.g., numeric, coded). **Example Input Data:** `Numeric`, `Coded`                                                                                                                                 |
| `rows.mapping_code`            | 0..1      | string               | Exact match on a concept ID or mapping in the target repo version. **Example Input Data:** `D50.0`, `Z87.5`, `X59.9`                                                                                                 |
| `rows.mapping_list`            | 0..1      | string               | Exact match on comma‑separated mapping list. *(In development.)* **Example Input Data:** `CIEL:1858, ICD10:DC14.Z, LOINC:5792-7`                                                                                     |
| `rows.same_as_map_codes`       | 0..1      | string               | Search only “same as” mappings. *(Deprecated.)* **Example Input Data:** `CIEL:1858, ICD10:DC14.Z, LOINC:5792-7`                                                                                                      |
| `rows.other_map_codes`         | 0..1      | string               | Search all non‑“same as” mappings. *(Deprecated.)* **Example Input Data:** `CIEL:1858, ICD10:DC14.Z, LOINC:5792-7`                                                                                                   |
| `map_config`                   | 0..\*     | list      | Optional list configuring mapping logic per row. **Example from Request Body:** see below.                                                                                                                           |
| `map_config.type`              | 1..1      | code                 | Type of mapping: `mapping-code` or `mapping-list`. **Example:** `mapping-code`, `mapping-list`                                                                                                                       |
| `map_config.input_column`      | 1..1      | string               | Name of the row‑field to use. **Example:** `loinc-example`, `icd10-example`, `list example`                                                                                                                          |
| `map_config.target_source_url` | 0..1      | string               | Target repo URL for `mapping-code` entries (required if type is `mapping-code`). **Example:** `/orgs/CIEL/sources/CIEL/`                                                                                             |
| `map_config.separator`         | 0..1      | string               | Separator between source name and code in `mapping-list`. **Example:** `:`                                                                                                                                           |
| `map_config.delimiter`         | 0..1      | string               | Delimiter for multiple mappings in `mapping-list`. **Example:** `,`                                                                                                                                                  |
| `map_config.target_urls`       | 0..1      | map                  | URL map of source mnemonics to repositories. Required for `mapping-list`. **Example:** `{"ICD10": "/orgs/WHO/sources/ICD-10-WHO/", "CIEL": "/orgs/CIEL/sources/CIEL/", "LOINC": "/orgs/Regenstrief/sources/LOINC/"}` |


## Response Format

| **Code (Name)**                | **Card.** | **Type**             | **Definition (Description)**   |
| ------------------------------ | --------- | -------------------- | ------- |
| _\<base\>_ | 1 | list | A list of response objects |
| row | 1 | map | The original row submitted, with no alteration |
| results | 1..* | list | Ordered list of concept candidates |
| results.url | 1 | string | |
| results.display_name | 1 | string | |
| results.id | 1 | string | |
| results.retired | 0..1 | bool | |
| results.concept_class | 0..1 | string | |
| results.datatype | 0..1 |  string | |
| results.property | 0..* | list |  |
| results.property.code | 1 | string | The key of the property (e.g. concept_class) |
| results.property.valueCode | 0..1 | string |  |
| results.property.valueCoding | 0..1 | ... |  |
| results.property.valueString | 0..1 | string |  |
| results.property.valueInteger | 0..1 | int |  |
| results.property.valueBoolean | 0..1 | bool |  |
| results.property.valueDateTime | 0..1 | DateTime |  |
| results.property.valueDecimal | 0..1 | decimal |  |
| results.extras | 0..1 | map | |
| results.search_meta.search_score | 1 | decimal | |
| results.search_meta.search_highlight | 0..1 |  map | |
| | | | |
| results.search_meta.match_type | 0..1 | string | |
| results.source | 0..1 | | |
| results.owner | 0..1 | | |
| results.owner_type | 0..1 | | |
| results.owner_url | 0..1 | | |
| results.mappings | 0..1 | | |
| results.names | 0..1 | | |

#### Response
```json
[
    {
        "row": {"local_id":"1396", "name":"malaria"},
        "results": [
            {
                "search_meta": {
                    "search_score": 2.0546277,
                    "match_type": "very_high",
                    "search_highlight": {}
                },
                "id": "49051-6",
                "url": "/orgs/Regenstrief/sources/LOINC/concepts/49051-6/",
                "retired": false,
                "source": "LOINC",
                "owner": "Regenstrief",
                "owner_type": "Organization",
                "owner_url": "/orgs/Regenstrief/",
                "display_name": "Gestational age in weeks",
                "display_locale": "en",
                "property": [
                  {"code": "concept_class", "valueCode": "Coded"},
                  {"code": "datatype", "valueCode": "Numeric"},
                  {"code": "units", "valueString": "parts/microliter"}
                ]
            },
            {
                "search_meta": {
                    "search_score": 2.0455465,
                    "match_type": "very_high",
                    "search_confidence": null,
                    "search_highlight": {}
                },
                "id": "56081-3",
                "url": "/orgs/Regenstrief/sources/LOINC/concepts/56081-3/",
                "retired": false,
                "source": "LOINC",
                "owner": "Regenstrief",
                "owner_type": "Organization",
                "owner_url": "/orgs/Regenstrief/",
                "display_name": "Fetal gestational age in weeks --at most recent delivery",
                "display_locale": "en"
            }
        }
    }
]
```

### Example Request 1: Simple Request
```
POST https://api.openconceptlab.org/concepts/$match/?includeSearchMeta=true&semantic=true&bestMatch=true&limit=1
```
```json
{
    "rows":[
        {"local_id":"1396", "name":"malaria"},
        {"local_id":"2", "name":"a1c"}
    ],
    "target_repo_url": "/orgs/CIEL/sources/CIEL/"
}
```

#### Response
```json
[
    {
        "row": {"local_id":"1396", "name":"malaria"},
        "results": [
            {
                "search_meta": {
                    "search_score": 2.0546277, # required
                    "match_type": "very_high", # optional
                    "search_confidence": null, # optional
                    "search_highlight": {}     # optional
                },
                "id": "49051-6",
                "url": "/orgs/Regenstrief/sources/LOINC/concepts/49051-6/",
                "retired": false,
                "source": "LOINC",
                "owner": "Regenstrief",
                "owner_type": "Organization",
                "owner_url": "/orgs/Regenstrief/",
                "display_name": "Gestational age in weeks",
                "display_locale": "en"
            },
            {
                "search_meta": {
                    "search_score": 2.0455465,
                    "match_type": "very_high",
                    "search_confidence": null,
                    "search_highlight": {}
                },
                "id": "56081-3",
                "url": "/orgs/Regenstrief/sources/LOINC/concepts/56081-3/",
                "retired": false,
                "source": "LOINC",
                "owner": "Regenstrief",
                "owner_type": "Organization",
                "owner_url": "/orgs/Regenstrief/",
                "display_name": "Fetal gestational age in weeks --at most recent delivery",
                "display_locale": "en"
            }
        }
    }
]
```


### Example Request 2

```json
{
    "rows":[
        {"local_id":"1396", "name":"Mother's HIV Status", "loinc_code": "75179-2"},
        {"local_id":"2", "name":"Weeks of gestation", "loinc_code": "11884-4"}
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
