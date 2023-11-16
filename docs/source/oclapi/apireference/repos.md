# Repositories

## Overview
The API exposes a consolidated view across all repository types from a single `repos` endpoint.
The primary benefit of the `repos` endpoint is that it allows a user to find a repository without having to consider a repository's specific configuration in OCL.
The `repos` endpoint only supports listing and searching and is in addition to the dedicated `sources` and `collections` endpoints exposed by the OCL API, and the `CodeSystem`, `ValueSet` and `ConceptMap` endpoints exposed by the OCL FHIR Core.

Example uses:
* `GET /repos/?q=ATC` - List public repos that match the search criteria
* `GET /orgs/MyOrg/repos/` - List all repos in an org
* `GET /users/username/repos/` - List all repos owned by a user

## List all repositories globally or for a specific user or organization
* List repositories globally or owned by an organization or user
```
GET /repos/
GET /orgs/:org/repos/
GET /users/:user/repos/
GET /user/repos/
```
* Notes
    * Private sources owned by the organization are only returned for users that are members of the organization
* Parameters
    * **q** (optional) string - Search criteria (search across: "name", "full_name" and "description")
    * **sortAsc/sortDesc** (optional) string - Sort results on one of the following fields: "best_match" (default), "last_update", "name"
    * **sourceType** (optional) string - Filter results to a given source type, e.g. "dictionary", "reference"
    * **locale** (optional) string - Filter results to those with a given locale in their supported_locales, e.g. "en", "fr"
    * **customValidationSchema** (optional) string - Filter results to a given validationSchema, e.g. "OpenMRS"
    * **canonicalUrl** (optional) string - Filter results by canonical URL








