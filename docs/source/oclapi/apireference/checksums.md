# Checksums

## Overview
OCL computes checksums for resources (concepts, mappings, sources, collections, organizations, and users) to support duplicate version detection, change tracking, and data integrity. Each resource stores two checksum types — **standard** and **smart** — as an MD5 hash in a `checksums` JSON field:

```json
{
  "standard": "d41d8cd98f00b204e9800998ecf8427e",
  "smart": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6"
}
```

### Standard vs Smart Checksums
* **Standard checksum** — A comprehensive hash that includes all semantically meaningful fields plus metadata. Any change to the resource, including to extras, external IDs, or ancillary attributes, will produce a different standard checksum.
* **Smart checksum** — A focused hash that includes only the most semantically significant fields. Minor metadata changes (e.g. adding an extra attribute or changing an external ID) do not affect the smart checksum. This is useful for detecting whether the *meaning* of a resource has changed.

When comparing repository versions, changes detected by the smart checksum are classified as **major changes**, while changes detected only by the standard checksum are classified as **minor changes**.


## Fields Included in Checksums

### Concept Checksums

| Field | Standard | Smart |
|---|:---:|:---:|
| `concept_class` | Yes | Yes |
| `datatype` | Yes | Yes |
| `retired` | Yes | Yes |
| `external_id` | Yes | |
| `extras` | Yes | |
| `names` (all) | Yes | |
| `names` (Fully Specified only) | | Yes |
| `descriptions` (all) | Yes | |
| `parent_concept_urls` | Yes | |
| `child_concept_urls` | Yes | |

**Name fields included:** `locale`, `locale_preferred`, `name`, `name_type`, `external_id`

**Description fields included:** `locale`, `locale_preferred`, `description`, `description_type`, `external_id`

For smart checksums, only names where `name_type` is "FULLY_SPECIFIED" (or "Fully Specified") are included.

### Mapping Checksums

| Field | Standard | Smart |
|---|:---:|:---:|
| `map_type` | Yes | Yes |
| `from_concept_code` | Yes | Yes |
| `to_concept_code` | Yes | Yes |
| `from_concept_name` | Yes | Yes |
| `to_concept_name` | Yes | Yes |
| `retired` | Yes | Yes |
| `sort_weight` | Yes | |
| `extras` | Yes | |
| `external_id` | Yes | |
| `from_source_url` | Yes | |
| `from_source_version` | Yes | |
| `to_source_url` | Yes | |
| `to_source_version` | Yes | |

### Concept Names and Descriptions
Individual concept names and descriptions also have their own checksums (standard only, no smart checksum).


## Algorithm

Checksum generation follows these steps:

### 1. Field Extraction
The relevant fields for the resource type and checksum type are extracted (see tables above).

### 2. Cleanup
* `null` values are removed
* Empty arrays and falsy values for certain fields (`retired`, `names`, `descriptions`, `extras`, `parent_concept_urls`, `child_concept_urls`, `locale_preferred`, `name_type`, `description_type`) are omitted
* Keys in `extras` that start with `__` (internal/system keys) are stripped
* `is_active` is omitted if `true` (the default)
* Numeric values are normalized (integer vs float)

### 3. Canonical Serialization
* Object keys are sorted alphabetically
* Arrays are sorted using a generic comparator
* Strings are unicode-escaped
* UUIDs are converted to their string representation

### 4. MD5 Hash
The serialized string is encoded to UTF-8 and hashed with MD5 to produce a 32-character hex digest.

### 5. Aggregate Checksums (Multiple Resources)
When computing a checksum over multiple resources (e.g. all concepts in a page of results), each individual resource's checksum is computed first, and then the list of checksums is itself hashed to produce a single aggregate checksum.


## How Checksums Are Used

### Duplicate Version Prevention
When creating or editing a concept or mapping, OCL computes the standard checksum of the submitted resource. If it matches the standard checksum of the current HEAD version, OCL does **not** create a new version and responds with HTTP status `208 Already Reported`. This reduces unnecessary version proliferation when identical updates are submitted.

### Response Headers
List endpoints include aggregate checksums in response headers:
* `X-OCL-API-STANDARD-CHECKSUM` — aggregate standard checksum for the current page of results
* `X-OCL-API-SMART-CHECKSUM` — aggregate smart checksum for the current page of results

These headers allow clients to quickly determine whether the content of a result set has changed without comparing individual resources.

### Resource Detail Responses
When retrieving a single resource's details, the `checksums` field is included in the response:
```json
{
  "id": "A15.0",
  "concept_class": "Diagnosis",
  "checksums": {
    "standard": "d41d8cd98f00b204e9800998ecf8427e",
    "smart": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6"
  }
}
```

### Checksums Query Parameter
Add `?checksums=true` to list endpoints to include checksums for each resource in the response.

### Repository Version Comparison (ChecksumDiff)
OCL uses checksums to compare two versions of a repository (source or collection) and classify resources into:
* **New** — present in the newer version but not the older
* **Removed** — present in the older version but not the newer (and not retired)
* **Retired** — newly retired in the newer version
* **Changed (major)** — smart checksum differs (semantically significant change)
* **Changed (minor)** — only the standard checksum differs (metadata-only change)
* **Same** — both checksums match


## Checksum API Endpoints

### Generate a Checksum
Generate a checksum for one or more resources without persisting anything:
```
POST /$checksum/standard/
POST /$checksum/smart/
```

**Query parameters:**
* `resource` — the resource type (`concept` or `mapping`)

**Request body:** A JSON object or array of objects representing the resource data.

**Response:** The computed checksum as a string.

**Example:**
```
POST /$checksum/standard/?resource=concept
Content-Type: application/json

{
  "concept_class": "Diagnosis",
  "datatype": "None",
  "names": [
    {"name": "Tuberculosis of lung", "locale": "en", "name_type": "FULLY_SPECIFIED"}
  ]
}
```


## Repository-Level Checksums
Sources and collections also store checksums. These are computed from the repository's own attributes and serve as a version fingerprint. Like resource checksums, they include both standard and smart variants.
