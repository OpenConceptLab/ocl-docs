# Canonical URL Registries

## Overview
The API exposes a `url-registry` endpoint to view and manage a canonical URL registry for the global namespace and for each owner namespace (i.e. a user or organization). Canonical URL registries provide users and organizations with an approach to unambiguously control how a canonical URL is resolved to a published terminology repository. Because OCL is a multi-tenant application, it is possible for multiple repositories in different namespaces to be defined with the same canonical URL. For example, OCL might have an official publication of LOINC and other organizations may have published their own subsets, fragments, or supplements of LOINC based on their own project requirements that share the same canonical URL.

Canonical URLs defined in the global registry serve as "defaults" that can be overwritten in an owner's namespace. In the happy path workflow, canonical URLs referenced by a user will be defined in a repository in their own namespace or already defined in the global registry, with the owner-specific URL registry only used to override the default behavior (e.g. pointing to a repository in another namespace that is not registered globally).

A canonical URL is resolved (e.g. using the $resolveReference operation) according to this algorithm:

* If namespace is set (and not global) and a namespace-specific canonical URL registry is defined, attempt to resolve with the namespace-specific canonical URL registry:
  * If the canonical URL is defined in the registry and it resolved, then return success
  * If the canonical URL is defined in the registry, but cannot be resolved, then fail
  * If otherwise unresolved (with no matching entry in the registry), then continue
* If namespace is set (and not global), attempt to resolve the canonical URL with the repos defined in the namespace:
  * If the canonical URL resolved in the repo, then return success
  * If unresolved, then continue
* If namespace could not be resolved in the repo, namespace is explicitly set to global, or namespace is undefined, attempt to resolve the canonical URL with the Global Canonical URL Registry:
  * If the canonical URL is defined in the global registry and it resolved, then return success
  * If the canonical URL is defined in the global registry, but cannot be resolved, then fail
  * If otherwise unresolved (with no matching entry in the global registry), then continue
* If still unresolved, then fail (even if the canonical URL is defined somewhere else in OCL):
  * Resolution to canonicals in other namespaces must be explicitly defined in a canonical URL registry

Example uses:
* `GET /url-registry/` - Get a paged list of entries in the global canonical URL registry
* `GET /url-registry/?q=ATC` - List entries in the global canonical URL registry that match the search criteria
* `GET /orgs/MyOrg/url-registry/` List URL registry entries for a specific owner's namespace
* `POST /orgs/MyOrg/url-registry/$lookup/` - Lookup a canonical URL in a specific registry


## Attributes for an entry in the canonical URL registry
* `id` <int> - internally generated sequential ID
* `url` (required) <url> - canonical URL for the entry
* `namespace` (optional) <string> - namespace to be used to resolve a URL, e.g. `/orgs/:org/` or `/users/:username/`; URL cannot be resolved to a repo unless a namespace is provided
* `name` (optional) <string> - name of the repo
* `extras` (optional) <json> - key-value pairs intended to store additional index terms to make it easy for users to find
* `owner_url` <url> - URL for the owner of the URL registry (e.g. "/orgs/MyOrg/"), _not_ the owner of the namespace to be used to resolve a URL
* `owner_type` string - Owner type, e.g. "Organization" or "User"
* `owner` string - Owner ID

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
[{ “id”: “...”,
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
{ “id”: “...”,
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

## Create new registry entry
```
POST /url-registry/
POST /:ownerType/:owner/url-registry/
```
```
{ “namespace”: “...”,  # e.g. “/orgs/CIEL/”
  “url”: “”,
  “name”: “”,
  “extras”: {}
}
```

## Update registry entry
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

## Delete registry entry
```
DELETE /url-registry/:id/
DELETE /:ownerType/:owner/url-registry/:id/
```

## Operation: $lookup
* Lookup a repository by URL -- returns the repo or returns no result/not found
```
POST /url-registry/$lookup/?url=:url
POST /:ownerType/:owner/url-registry/$lookup/?url=:url
```

## Future Requirements
* System-specific Canonical URL Formats: OCL does not need to know anything about system-specific canonical URL formats for now – a canonical URL is either a match to something defined in OCL, or its not (e.g. see SNOMED-specific handling of canonicals with FHIR)
* Ability to batch load/etc. – out of scope for MVP
