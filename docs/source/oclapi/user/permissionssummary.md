# Permissions Summary

Note: This document is currently a DRAFT version

## Syntax
* This document uses short-hand to keep it concise:
	* `users` and `orgs` are joinly referred to as `owners` and their identifiers as `:owner`. `:ownerType` may have the value "users" or "orgs".
	* `sources` and `collections` are jointly referred to as `repos` (short for repositories) and their identifiers as `:repo`. `:repoType` may have the value "sources" or "collections".

## Notes
* A "private organization" is an organization that has `public_access` set to "None". Note that the `user` resource does not have a `public_access` field and the user profile is always public.
* A "public repository" is a source or collection that has `public_access` set to "View" or "Edit". A "public resource" refers to any user profile and any public organization, public repository and any sub-resource of a public repository (i.e. concept, mapping, version).
* A "private repository" is a source or collection that has `public_access` set to "None". A "private resource" refers to any private organization, private repository or any sub-resource (i.e. concept, mapping, version) of a private repository.
* All POST, PUT, DELETE, and PATCH requests require authentication. Many GET and HEAD requests may be performed without authentication, however GET/HEAD requests still requires authentication for private and other protected resources.


## Permissions

### Permissions for anonymous users
* An unauthenticated user is not permitted to make any requests using the `/user/` endpoint.
* An unauthenticated user is not permitted to perform any POST, PUT, PATCH, or DELETE requests on any endpoint.
* An unauthenticated user is not permitted to perform GET or HEAD requests on a private organization, a private repository, or any sub-resources of a private organization or private repository.
* An unauthenticated user is permitted to search using the `/users/` endpoint and to view user profiles (e.g. `GET /users/` and `GET /users/:user/`), since all user profiles are public.
* An unauthenticated user is permitted to search using the top-level search endpoints. Note that only **public** resources are available through these endpoints.
	* `GET /collections/`
	* `GET /sources/`
	* `GET /concepts/`
	* `GET /mappings/`
	* `GET /orgs/`
* An unauthenticated user is permitted to search **public** repositories and any sub-resources of **public** repositories, including versions, concepts, and mappings. For example, these requests are all permitted:
	* `GET /:ownerType/:owner/sources/`
	* `GET /:ownerType/:owner/collections/`
	* `GET /:ownerType/:owner/sources/versions/`
	* `GET /:ownerType/:owner/collections/versions/`
	* `GET /:ownerType/:owner/sources/:source/[/:sourceVersion/]concepts/`
	* `GET /:ownerType/:owner/sources/:source/[/:sourceVersion/]mappings/`
	* `GET /:ownerType/:owner/collections/:collection/[:collectionVersion/]concepts/`
	* `GET /:ownerType/:owner/collections/:collection/[:collectionVersion/]mappings/`
* An unauthenticated user is permitted to view details of any **public** repository and any sub-resources of **public** repositories, including versions, concepts, and mappings. For example, these requests are all permitted:
    * `GET /:ownerType/:owner/sources/:source/`
    * `GET /:ownerType/:owner/sources/:source/versions/:version/`
    * `GET /:ownerType/:owner/sources/:source/[:sourceVersion/]/concepts/:concept/[:conceptVersion/]`
    * `GET /:ownerType/:owner/sources/:source/[:sourceVersion/]/mappings/:mapping/`
    * `GET /:ownerType/:owner/collections/:collection/`
    * `GET /:ownerType/:owner/collections/:collection/versions/:version/`
    * `GET /:ownerType/:owner/collections/:collection/[:collectionVersion/]concepts/:concept/`
    * `GET /:ownerType/:owner/collections/:collection/[:collectionVersion/]mappings/:mapping/`

### Permissions for authenticated users
* An authenticated user is permitted to perform any operation that an unauthenticated user is permitted to perform (see above).
* An authenticated user is permitted to perform any operation on resources and sub-resources available through the `/user/` endpoint. For example:
	* `GET /user/`
	* `POST /user/sources/`
	* `POST /user/collections/:collection/references/`
	* `DELETE /user/sources/:source/mappings/:mapping/`
* An authenticated user is permitted to create an organization:
	* `POST /orgs/`
* If an authenticated user is a **member** of an organization, it is permitted to view organization details, membership, and repositories:
	* `GET /orgs/:org/`
	* `GET /orgs/:org/members/`
	* `GET /orgs/:org/members/:user/`
	* `GET /orgs/:org/sources/`
	* `GET /orgs/:org/collections/`
* If an authenticated user is the **owner** of an organization, it is permitted to edit/delete the organization, edit membership, and create new repositories in addition to everything that a **member** of an organization can do:
	* `POST /orgs/:org/`
	* `DELETE /orgs/:org/`
	* `PUT /orgs/:org/members/:user/`
	* `DELETE /orgs/:org/members/:user/`
	* `POST /orgs/:org/sources/`
	* `POST /orgs/:org/collections/`
* If an authenticated user is a **contributor** to a repository, it is permitted to:
	* `GET /orgs/:org/:repoType/:repo/`
* If an authenticated user is the **owner** of a repository, it is permitted to edit/delete the repository and to create/edit/delete versions of the repository:
	* `POST /orgs/:org/:repoType/:repo/`
	* `DELETE /orgs/:org/:repoType/:repo/`
	* `POST /orgs/:org/:repoType/:repo/versions/`
	* `POST /orgs/:org/:repoType/:repo/versions/:version/`
	* `DELETE /orgs/:org/:repoType/:repo/versions/:version/`

### Sysadmin permissions
* Only the sysadmin may create new users
