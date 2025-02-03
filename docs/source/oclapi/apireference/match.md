# $match Concepts (EXPERIMENTAL)

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
