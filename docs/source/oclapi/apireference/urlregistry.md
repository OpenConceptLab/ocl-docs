# Canonical URL Registries

## Overview
The API exposes a `url-registry` endpoint to view and manage a canonical URL registry for the global namespace and for each owner namespace (i.e. a user or organization). Canonical URL registries provide users and organizations with an approach to unambiguously control how a canonical URL is resolved to a published terminology repository. Because OCL is a multi-tenant application, it is possible for multiple repositories in different namespaces to be defined with the same canonical URL. For example, OCL might have an official publication of LOINC and other organizations may have published their own subsets, fragments, or supplements of LOINC based on their own project requirements that share the same canonical URL.

Canonical URLs defined in the global registry serve as "defaults" that can be overridden in an owner's namespace. In the happy path workflow, canonical URLs referenced by a user will be defined in a repository in their own namespace or already defined in the global registry, with the owner-specific URL registry only used to override the default behavior (e.g. pointing to a repository in another namespace that is not registered globally).

The `$resolveReference` operation implements the business logic to leverage namespace-specific and the global Canonical URL Registries to resolve a canonical URL within a specific context. Refer to the `Operation: $resolveReference` documentation for more information.

Example uses:
* `GET /url-registry/` - Get a paged list of entries in the global canonical URL registry
* `GET /url-registry/?q=ATC` - List entries in the global canonical URL registry that match the search criteria
* `POST /$lookup/?url=https://loinc.org` - Lookup a canonical URL in the global URL registry
* `GET /orgs/MyOrg/url-registry/` - List URL registry entries for a specific owner
* `POST /orgs/MyOrg/url-registry/$lookup/?https://loinc.org` - Lookup a canonical URL in an owner-specific URL registry

## Attributes for an entry in the canonical URL registry
* `id` <int> - internally generated sequential ID
* `url` (required) <url> - canonical URL for the entry
* `namespace` (optional) <string> - namespace to be used to resolve a URL, e.g. `/orgs/:org/` or `/users/:username/`; URL cannot be resolved to a repo unless a namespace is provided
* `name` (optional) <string> - name of the repo
* `extras` (optional) <json> - key-value pairs intended to store additional index terms to make it easy for users to find
* `owner_url` <url> - URL for the owner of the URL registry (e.g. "/orgs/MyOrg/"), _not_ the owner of the namespace to be used to resolve a URL
* `owner_type` string - Owner type, e.g. "Organization" or "User"
* `owner` string - Owner ID
* `type` string - "URLRegistryEntry"

## Get/search list of URL registry entries, with paging
```
GET /url-registry/
GET /:ownerType/:owner/url-registry/
```
* Parameters
  * **q** (optional) string - Full-text search criteria looks at URL, name, namespace, and extras
  * **url** (optional) url - Match a specific url
  * **namespace** (optional) string - Match a specific namespace, e.g. "/orgs/MyOrg/", "/users/johndoe/"

### Response
```
[{
  "type": "URLRegistryEntry",
  “id”: “...”,
  “namespace”: “...”,  # e.g. “/orgs/CIEL/”
  “url”: <url>,
  “name”: <str>,
  “extras”: <json>,
  “created_at”: <datetime>,
  “updated_at”: <datetime>,
  “created_by”: <user>,
  “updated_by”: <user>,
  “is_active”: <bool>,
  "owner_type": "Organization",
  "owner": "MyOrg",
  "owner_url": "/orgs/MyOrg/"
}, ...]
```

## Get registry entry
```
GET /url-registry/:id/
GET /:ownerType/:owner/url-registry/:id/
```

### Response
```
{
  "type": "URLRegistryEntry",
  “id”: “...”,
  “namespace”: “...”,  # e.g. “/orgs/CIEL/”
  “url”: “”,
  “name”: “”,
  “extras”: {},
  “created_at”: <datetime>,
  “updated_at”: <datetime>,
  “created_by”: <user>,
  “updated_by”: <user>,
  “is_active”: <bool>,
  "owner_type": "Organization",
  "owner": "MyOrg",
  "owner_url": "/orgs/MyOrg/"
}
```

## Create a new URL registry entry
```
POST /url-registry/
POST /:ownerType/:owner/url-registry/
```
```
{ “namespace”: “...”,  # e.g. “/orgs/CIEL/”
  “url”: “https://loinc.org/”,
  “name”: “LOINC”,
  “extras”: {}
}
```

## Update a URL registry entry
```
PUT/PATCH /url-registry/:id/
PUT/PATCH /:ownerType/:owner/url-registry/:id/
```
```
{ “namespace”: “...”,  # e.g. “/orgs/CIEL/”
  “url”: “”,
  “name”: “”,
  “extras”: {}
}
```

## Delete a URL registry entry
```
DELETE /url-registry/:id/
DELETE /:ownerType/:owner/url-registry/:id/
```

## Operation: $lookup
* Lookup a repository by canonical URL within a specific URL Registry -- returns the a repository summary and 200 or returns 404 for not found
```
POST /url-registry/$lookup/?url=:url
POST /:ownerType/:owner/url-registry/$lookup/?url=:url
```

## Future Requirements
* System-specific Canonical URL Formats: OCL does not need to know anything about system-specific canonical URL formats for now – a canonical URL is either a match to something defined in OCL, or its not (e.g. see SNOMED-specific handling of canonicals with FHIR)
* Ability to batch load/etc. – out of scope for MVP
