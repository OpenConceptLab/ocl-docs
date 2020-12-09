# Formatted CSV Responses
## Overview
Most API requests allow setting the format to CSV. CSV responses only return a subset of fields formatted for readability and sharing. For a full export of data, you should use JSON Exports. This page describes the syntax of CSV exports for each resource type.

* Users
* Organizations
* Sources
* Collections
* Repository Versions
* Concepts
* Mappings
* References

## Content below is out of date!

### Users
**Columns** - excludes extras and navigation URLs
* type
* uuid
* username
* company
* location
* email
* preferred_locale
* website
* url
* orgs
* public_collections
* public_sources
* created_on
* created_by
* updated_on
* updated_by

### Organizations
**Columns** - excludes extras, short_code, and navigation URLs
* type
* id
* uuid
* external_id
* url
* name
* company
* website
* location
* public_access
* members - CSV - "username"
* public_collecions
* public_sources
* created_on
* created_by
* updated_on
* updated_by

### Organization Members
**Columns** - excludes ??

### Sources
**Columns** - excludes extras, short_code, and navigation URLs
* type
* id
* uuid
* external_id
* url
* name
* full_name
* source_type
* public_access
* default_locale
* supported_locales
* website
* description
* owner
* owner_type
* owner_url
* versions
* active_concepts
* active_mappings
* created_on
* created_by
* updated_on
* updated_by

### Collections
**Columns** - excludes extras, short_code, and navigation URLs
* type
* id
* uuid
* external_id
* url
* name
* full_name
* collection_type
* public_access
* default_locale
* supported_locales
* website
* description
* owner
* owner_type
* owner_url
* versions
* active_concepts
* active_mappings
* created_on
* created_by
* updated_on
* updated_by

### Repository Versions
**Columns** - excludes extras, root/previous/parent urls
* type - e.g. "Source Version" or "Collection Version"
* id
* external_id
* url
* released
* retired (if implemented)
* description
* created_on
* created_by
* updated_on
* updated_by

### Mappings
**Simplified**
* from_concept - [:owner]:[:source]:[:fromConceptCode]
* map_type
* to_concept - [:owner]:[:source]:[:toConceptCode]
* to_concept_name (if available)
* internal_external - "Internal" or "External" mapping?
* owner
* owner_type
* owner_url
* url (for the mapping)

**Columns** - excludes extras
* type
* uuid
* external_id
* url
* retired
* map_type
* from_source_owner
* from_source_owner_type
* from_source_name
* from_concept_code
* from_concept_name
* from_concept_url
* to_source_owner
* to_source_owner_type
* to_source_name
* to_concept_code
* to_concept_name (optional)
* to_concept_url (only present if "internal" mapping)
* internal_external - "Internal" or "External" mapping?
* owner
* owner_type
* owner_url
* source
* created_on
* created_by
* updated_on
* updated_by

### Concepts
**Columns** - excludes extras, navigation URLs
* type
* id
* uuid
* external_id
* url
* concept_class
* datatype
* retired
* display_name
* display_locale
* names: CSV - "name [locale] [name type]"
* descriptions: CSV - "description [locale] [description type]"
* mappings: CSV of direct mappings stored in same source with map_type, to_concept ([:owner]:[:source]:[:toConceptCode]), to_concept_name (if available), and internal/external flag
* owner
* owner_type
* owner_url
* source
* concept_version = version
* versions
* created_on
* created_by
* updated_on
* updated_by
