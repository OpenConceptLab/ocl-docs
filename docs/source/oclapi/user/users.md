# Users

## Overview
The API exposes a representation of OCL `users`. Access to most resources via the API requires an authenticated user and the authentication information must be passed with the request.

### Versioning of users
* Version history is not stored for `users` - edits are in-place and only the standard audit fields are maintained ("created_on", "created_by", "updated_on" and "updated_by")

### Future Considerations
* Want to support optional gravatar images for users in the future



## Get a single user
* Get the public version of a user
```
GET /users/:user/
```

### Response
* Status: 200 OK
```JSON
{
    "type": "User",
    "uuid": "8d94f280-c2cc-11de-8d13-0010c6dffd0f",

    "username": "johndoe",
    "name": "John Doe",
    "company": "My Company",
    "location": "Kenya",
    "email": "johndoe@me.com",
    "preferred_locale": "en",
    "website": "http://mydomain.me/",          # NOT CURRENTLY IMPLEMENTED

    "url": "/users/johndoe/",
    "collections_url": "/users/johndoe/collections/",
    "sources_url": "/users/johndoe/sources/",
    "orgs_url": "/users/johndoe/orgs/",

    "extras": null,

    "orgs": 3,
    "public_collections": 3,
    "public_sources": 1,

    "created_on": "2008-01-14T04:33:35Z",
    "created_by": "johndoe",
    "updated_on": "2008-02-18T09:10:16Z",
    "updated_by": "johndoe"
}
```



## Get the authenticated user
* Get the non-public version of the authenticated user
```
GET /user/
```
* Note that authentication information must be passed with the request. E.g. `curl -u "username" "/user/"`

### Response
* Status: 200 OK
* Currently this returns the same JSON response as `GET /users/:user`



## Partial update of a user
* Update selected fields of the currently authenticated user
```
POST /user/
```
* [OCL Admin only] Update a user
```
POST /users/:user/
```
* For the partial update, pass only the fields that will be updated
* Note that authentication information for the user to be updated must be passed with the request. E.g. `curl -u "username" "/user/"`
* Input
    * **name** (optional) string - public name
    * **email** (optional) string - public email address
    * **company** (optional) string - public company name
    * **location** (optional) string - public location (e.g. Boston, MA, USA)
    * **preferred_locale** (optional) string - ordered, comma-separated list of preferred locales (e.g. "en", "es", "en,es")
    * **website** (optional) string - fully-specified URL
    * **extras** (optional) json dictionary - additional metadata
```JSON
{
    "name": "Johnny Doe",
    "email": "jdoe@me.com",
    "company": "My New Company",
    "location": "Eldoret, Kenya",
    "preferred_locale": "en,sw",
    "website": "http://mydomain.me/",
    "extras": { "my-field": "my-value" }
}
```

### Response
* Status: 200 OK
* Returns the full, non-public JSON representation of the updated user object, same as `GET /user/`.



## List all users
* List all users
* Returns the reference JSON representation only
* Default sort is "created_at" ascending - meaning the order in which users were created (???)
```
GET /users/
```
* Parameters
    * **q** (optional) - search criteria to filter users (searches across "username", "full_name", "company", and "location")
    * **sortAsc/sortDesc** (optional) - sort on one of the following attributes: "bestMatch" (default), "dateJoined", "username"

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



## Create new user
### Create new user
* Requires administrative rights - standard users will do this through the web application and will use the web app's user account
```
POST /users/
```
* Input
    * **username** (required) string - username; must be unique
    * **name** (required) string - public name
    * **email** (required) string - public email address
    * **company** (optional) string - public company name
    * **location** (optional) string - public location (e.g. Boston, MA, USA)
    * **preferred_locale** (optional) string - ordered, comma-separated list of preferred locales (e.g. "en", "es", "en,es")
    * **website** (optional) string - fully-specified URL
    * **extras** (optional) json dictionary - additional metadata
```JSON
{
    "username": "johnnydoe",
    "name": "Johnny Doe",
    "email": "jdoe@me.com",
    "company": "My New Company",
    "location": "Eldoret, Kenya",
    "preferred_locale": "en,sw",
    "website": "http://mydomain.me/",
    "extras": { "my-field": "my-value" }
}
```

### Response
* Status: 201 Created
* Location: http://api.openconceptlab.com/users/johnnydoe/
```JSON
{
    "type": "User",

    "uuid": "8d94f280-c2cc-11de-8d13-0010c6dffd0f",
    "username": "johnnydoe",
    "name": "Johnny Doe",
    "company": "My New Company",
    "location": "Eldoret, Kenya",
    "email": "jdoe@me.com",
    "preferred_locale": "en,sw",
    "website": "http://mydomain.me/",         # NOT CURRENTLY IMPLEMENTED

    "extras": { "my-field": "my-value" },

    "url": "/users/johnnydoe/",
    "collections_url": "/users/johnnydoe/collections/",
    "sources_url": "/users/johnnydoe/sources/",
    "orgs_url": "/users/johnnydoe/orgs/",

    "orgs": 0,
    "public_collections": 0,
    "public_sources": 0,

    "created_on": "2008-01-14T04:33:35Z",
    "created_by": "johnnydoe",
    "updated_on": "2008-01-14T04:33:35Z",
    "updated_by": "johnnydoe"
}
```
* Status: 400 Bad Request - if the user already exists, or other errors. (Cannot distinguish between already exists active, or deactivated users?)



### Signup
Signup a user. Will require email confirmation before the user can login.

```
POST /users/signup/
```
* Input
    * **username** (required) string - username; must be unique
    * **name** (required) string - public name
    * **email** (required) string - public email address
    * **company** (optional) string - public company name
    * **location** (optional) string - public location (e.g. Boston, MA, USA)
    * **preferred_locale** (optional) string - ordered, comma-separated list of preferred locales (e.g. "en", "es", "en,es")
    * **website** (optional) string - fully-specified URL
    * **extras** (optional) json dictionary - additional metadata
    * **email_verify_success_url** - url to redirect to after email confirmation signup success
    * **email_verify_failure_url** - url to redirect to after email confirmation signup failure
```JSON
{
    "username": "johnnydoe",
    "name": "Johnny Doe",
    "email": "jdoe@me.com",
    "company": "My New Company",
    "location": "Eldoret, Kenya",
    "preferred_locale": "en,sw",
    "website": "http://mydomain.me/",
    "extras": { "my-field": "my-value" }
}
```

### Response
* Status: 201 Created
* Location: http://api.openconceptlab.com/users/johnnydoe/
```JSON
{
    "type": "User",

    "uuid": "8d94f280-c2cc-11de-8d13-0010c6dffd0f",
    "username": "johnnydoe",
    "name": "Johnny Doe",
    "company": "My New Company",
    "location": "Eldoret, Kenya",
    "email": "jdoe@me.com",
    "preferred_locale": "en,sw",
    "website": "http://mydomain.me/",         # NOT CURRENTLY IMPLEMENTED

    "extras": { "my-field": "my-value" },

    "url": "/users/johnnydoe/",
    "collections_url": "/users/johnnydoe/collections/",
    "sources_url": "/users/johnnydoe/sources/",
    "orgs_url": "/users/johnnydoe/orgs/",

    "orgs": 0,
    "public_collections": 0,
    "public_sources": 0,

    "created_on": "2008-01-14T04:33:35Z",
    "created_by": "johnnydoe",
    "updated_on": "2008-01-14T04:33:35Z",
    "updated_by": "johnnydoe"
}
```
* Status: 400 Bad Request - if the user already exists, or other errors. (Cannot distinguish between already exists active, or deactivated users?)



## List organizations that user is a member of
* List organizations for the currently authenticated user
```
GET /user/orgs/
```
* List organizations for the specified user
```
GET /users/:user/orgs/
```
* Parameters
    * "q" (optional) - search criteria

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



## Deactivate a user account
* A user account is deactivated through the web application and will use the web app's administrative user account. To reactivate a deactivated user, see below.
* This operation requires administrative rights
```
DELETE /users/:user/
```
* Note that this only **deactivates** the account using an internal flag. This effectively hides the user account and any data or metadata it owns, but does not delete it from the system.
* Note that authentication information for a user with administrative access must be passed with the request. E.g. `curl -u "username" "/users/johndoe/"`

### Response
* Status: 204 No Content - if the user is deactivated successfully.
* Status: 404 Not Found - if the user specified does not exist, or is already deactivated.



## Reactivate a deactivated user account
* Reactivating a deactivated user account requires administrative permissions
* Note that user accounts can only be reactivated if they have not been deleted from the system.
* User is reactivated, as are any resources belonging to the user.
```
PUT /users/:user/reactivate/
```

### Response
* Status: 204 No Content
* Status: 404 Not Found - if the user specified does not exist



## Retrieve auth token
* This is intended to be called by the application user.
* We assume that the end user has authenticated with the application with a username & password.
* We also assume that there is a corresponding API user with the same username (mnemonic), and that the application user supplied a hashed password when it created this API user.
* If the user created their account via the signup route, they need to have confirmed their email to do this.
```
POST /users/login/
```

* Input
    * **username** (required) string - username that exists both in the application and in the API
    * **password** (required) string - hashed version of the end user's application password
    * **email_verify_success_url** - url to redirect to after email confirmation signup success
    * **email_verify_failure_url** - url to redirect to after email confirmation signup failure

### Response
* Status: 200 OK
```JSON
{ "token": "akEg4ybtattVqANXbQLlm2e67PmW1" }
```



## Proposed Search and Filter Behavior
* Text Search (e.g. `q=criteria`) - NOTE: Plus-sign (+) indicates relative relevancy weight of the term
    * user.username (++++), user.name (++), user.company (+), user.location (+)
* Facets
    * ??
* Filters
    * **company** - user.company
    * **location** - user.location
* Sort
    * **bestMatch** (default) - see search fields above
    * **dateJoined** (Asc/Desc) - user.created_on
    * **username** (Asc/Desc) - user.username



## Issues
* Compare search behavior to GitHub
* These attributes are not yet implemented in the user details: website, starred_collections, starred_concepts, starred_sources, followers
