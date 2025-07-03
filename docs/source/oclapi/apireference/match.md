# Operation: $match (Experimental)

## Overview

The `$match` endpoint allows you to find similar or matching concepts across different repositories in OCL. While search returns concepts that match a specific search query, `$match` returns concept candidates that match structured input data, such as a row in a spreadsheet. This could be used to retrieve mapping candidates for an entire spreadsheet.

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
| `rows`                         | 1..\*     | list      | List of concept-like objects to match; each item may have different fields. **Example:** `[{"s_n":"1","name":"malaria"},{"s_n":"2","name":"blood type"}]`                                                            |
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
| `map_config.target_urls`       | 0..1      | map\<string, string> | URL map of source mnemonics to repositories. Required for `mapping-list`. **Example:** `{"ICD10": "/orgs/WHO/sources/ICD-10-WHO/", "CIEL": "/orgs/CIEL/sources/CIEL/", "LOINC": "/orgs/Regenstrief/sources/LOINC/"}` |


### Example Request
```
POST https://api.openconceptlab.org/concepts/$match/?includeSearchMeta=true&semantic=true&bestMatch=true
```
```json
{
    "rows":[
        {"s_n":"1", "name":"malaria"},
        {"s_n":"2", "name":"blood type"}
    ],
    "target_repo_url": "/orgs/MSF/sources/MSF/20250311/",
    "map_config": [
        {"type": "mapping-code", "input_column": "loinc-example", "target_source_url": "/orgs/CIEL/sources/CIEL/"},
        {"type": "mapping-code", "input_column": "icd10-example", "target_source_url": "/orgs/CIEL/sources/CIEL/"},
        {"type": "mapping-list", "input_column": "list example", "separator": ":", "delimiter": ",", "target_urls": {
            "ICD10": "/orgs/WHO/sources/ICD-10-WHO/",
            "CIEL": "/orgs/CIEL/sources/CIEL/",
            "LOINC": "/orgs/Regenstrief/sources/LOINC/"
        }
    ]
}
```
