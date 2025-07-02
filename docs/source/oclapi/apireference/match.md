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

| Field | Type | Required | Description | Example |
|-------|------|----------|-------------|---------|
| `target_repo_url` | String | Yes | Repository URL to match against. Uses $resolve operation to get the repo version | `/orgs/CIEL/sources/CIEL/` |
| `rows` | Array | Yes | List of concept-like objects to match. Each object can have different structure | ... |

#### Schema for `rows`

| Display | Current $match field | Description | Example Input Data |
| :---- | :---- | :---- | :---- |
| ID | `id` | Exact match on concept ID *Note: This is the same as Mapping: Code with the target repo selected. We may remove this option.* | 12, 57, A01.1 |
| Name | `name` | Semantic or fuzzy string search (based on selected algorithm) on primary display name | Anemia due to blood loss |
| Synonyms | `synonyms` | Semantic or fuzzy string search (based on selected algorithm) on all concept names and synonyms | “Anemia, blood loss”, “Anémie secondaire à une hémorragie” |
| Description | `description` | Basic string search on concept descriptions | “Anemia due to bleeding or a hemorrhagic process” |
| Property: Class | `concept_class` | String matching on concept class (e.g. diagnosis, symptom) | Diagnosis, Symptom |
| Property: Datatype | `datatype` | String matching on concept datatype (e.g. numeric, coded) | Numeric, Coded |
| Mapping: Code | `mapping_code` | Exact match of the concept ID or the mappings for the selected target repo version, where a value in the input data may have only a single code. | D50.0, Z87.5, X59.9 |
| Mapping: List | `mapping_list` | Exact match of the concept ID or the mappings for the selected target repo version. A value in the input data may have a comma-separated list of key-value pairs, where the key is the mnemonic for the map repository (e.g. ICD10) and the value is the code. *Note: This feature is currently under development.* | CIEL:1858, ICD10:DC14.Z, LOINC:5792-7 |
| Same As Mappings | `same_as_map_codes` | Searches only “same as” mappings *Note: Deprecated. This option will not be supported moving forward.* | CIEL:1858, ICD10:DC14.Z, LOINC:5792-7 |
| Concept Set | `other_map_codes` | Searches all map codes except “same as” *Note: Deprecated. This option will not be supported moving forward.* | CIEL:1858, ICD10:DC14.Z, LOINC:5792-7 |

### Example
```
POST https://api.openconceptlab.org/concepts/$match/?includeSearchMeta=true&semantic=true&bestMatch=true
{
    "rows":[
        {"s_n":"1", "name":"malaria"},
        {"s_n":"2", "name":"blood type"}
    ],
    "target_repo_url": "/orgs/MSF/sources/MSF/20250311/"
}
```
