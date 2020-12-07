# Orgs
## Table of Contents
**Overview**
* [Overview](orgs#overview)

**Organizations**
* [Get a single organization](orgs#get-a-single-organization)
* [List organizations](orgs#list-organizations)
* [Create new organization](orgs#create-new-organization)
* [Update organization](orgs#update-organization)
* [Deactivate an organization](orgs#deactivate-an-organization)
* [Delete an organization](orgs#delete-an-organization)

**Members**
* [List members of an organization](orgs#list-members-of-an-organization)
* [Get organization member status](orgs#get-organization-member-status)
* [Add new member to organization](orgs#add-new-member-to-organization)
* [Remove member from organization](orgs#remove-member-from-organization)



## Overview
The API exposes a representation of OCL `orgs`. `users` are members of `orgs`. `orgs` may "own" `collections` and `sources` in the same way that `users` may. However, `orgs` do not actually perform any edits since they cannot authenticate; instead, `users` which are members with appropriate permissions may create new `sources` and `collections` on behalf of an organization.

### Versioning of organizations
* Version history is not stored for `orgs` - edits are in-place and only standard audit information is maintained ("created_on", "created_by", "updated_on", "updated_by")



## Get a single organization
* Get a single organization - private organizations can only be retrieved with appropriate privileges
```
GET /orgs/:org/
```

### Response
* Status: 200 OK
```JSON
{
    "type": "Organization",
    "uuid": "8d94f280-c2cc-11de-8d13-0010c6dffd0f",
    "id": "My-Organization",

    "name": "My Organization",
    "company": "Company Name",
    "website": "http://myorganization.com/",
    "location": "Boston, MA, USA",
    "public_access": "View",

    "extras": { "extra-meta-data": "my-value" },

    "url": "/orgs/My-Organization/",
    "members_url": "/orgs/My-Organization/members/",
    "sources_url": "/orgs/My-Organization/sources/",
    "collections_url": "/orgs/My-Organization/collections/",

    "members": 2,
    "public_collections": 1,
    "public_sources": 1,

    "created_on": "2008-01-14T04:33:35Z",
    "created_by": "johndoe",
    "updated_on": "2008-02-18T09:10:16Z",
    "updated_by": "johndoe"
}
```



## List organizations
* List all public organizations
```
GET /orgs/
```
* List all organizations for a user
```
GET /users/:user/orgs/
```
* List all public and private organizations for the authenticated user
```
GET /user/orgs/
```
* Parameters
    * **q** (optional) - search criteria (across these fields: "name" and "company")
    * **verbose** (optional) - returns detailed results (same as getting a single organization)
    * **sortAsc/sortDesc** (optional) - sort the results (ascending or descending) on one of the following attributes: "bestMatch" (default), "lastUpdated", "name"

### Response
* Status: 200 OK
```JSON
[
    {
        "id": "My-Organization",
        "name": "My Organization",
        "url": "/orgs/My-Organization/"
    }
]
```



## Create new organization
* Creates a new organization with the currently authenticated user as the owner and as a member
```
POST /orgs/
```
* Input
    * **id** (required) string
    * **name** (required) string
    * **company** (optional) string
    * **website** (optional) string
    * **location** (optional) string
    * **extras** (optional) json dictionary
    * **public_access** (optional) string "View" (default), "Edit", or "None"
```JSON
{
    "id": "My-New-Org",
    "name": "My New Org",
    "company": "My Company Name",
    "website": "http://mycompany.com/",
    "location": "Paris, France",
    "extras": { "extra-meta-data": "my-value" },
    "public_access": "View"
}
```

### Response
* Status: 201 Created
* Returns the full, non-public JSON representation of the new organization object



## Update organization
* Update metadata for an organization. Authenticated user must be a member of the organization.
```
POST /orgs/:org/
```
* Note that modifying "name" does not change the value of "id"
* Input
    * **name** (optional) string
    * **company** (optional) string
    * **website** (optional) string
    * **location** (optional) string
    * **extras** (optional) json dictionary
    * **public_access** (optional) string "View" (default), "Edit", or "None"
```JSON
{
    "name": "My New Organization Name",
    "company": "My Company Name",
    "website": "http://mycompany.com/",
    "location": "Mumbai, India",
    "extras": { "extra-meta-data": "my-value" }
}
```

### Response
* Status: 200 OK
* Returns the updated version of the organization - same as `GET /orgs/:org/`



## Deactivate an organization
* To be implemented (https://github.com/OpenConceptLab/ocl_issues/issues/67)

## Delete an organization
* Authenticated user must be a superuser in order to delete an organization
```
DELETE /orgs/:org/
```

### Response
* Status: 204 No Content


## List members of an organization
* Get the list of members for an organization - Organization must be public or the authenticated user is also a member
```
GET /orgs/:org/members/
```

### Response
* Status: 200 OK
```JSON
[
    {
        "username": "johndoe",
        "name": "John Doe",
        "url": "/users/johndoe/"
    }
]
```



## Get organization member status
* Get whether a specific user is a member of an organization - authenticated user must be a member of the organization to do this
```
GET /orgs/:org/members/:user/
```

### Response if user is a member
* Status: 204 No Content

### Response if user is not a member
* Status: 404 Not Found



## Add new member to organization
* Add a new member to an organization - in order to add a new member, the authenticated user must be an administrative member of the organization
```
PUT /orgs/:org/members/:user/
```

### Response
* Status: 204 No Content



## Remove member from organization
* Authenticated user must be an administrative member of the organization in order to remove an organization member
```
DELETE /orgs/:org/members/:user/
```
* Note that this does not delete the user account, it only removes them from the organization

### Response
* Status: 204 No Content - Member successfully removed
* Status: 404 Not found OR 409 Conflict - User is not a member of the organization
* Status: 404 Not Found - User does not exist
* Status: 401 Unauthorized - No authentication credentials provided
* Status: 403 Forbidden - Authentication credentials provided, but user does not have permission to remove a member



## Search and Filter Behavior
* Text Search (e.g. `q=criteria`) - NOTE: Plus-sign (+) indicates relative relevancy weight of the term
    * org.name (++++), org.company (+)
* Facets
    * ??
* Filters
    * **company** - user.company
    * **location** - user.location
* Sort
    * **bestMatch** (default) - see search fields above
    * **name** (Asc/Desc) - org.name
    * **lastUpdated** (Asc/Desc) - org.last_updated
