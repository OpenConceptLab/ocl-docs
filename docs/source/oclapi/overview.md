OCL API Overview
=======================
## Overview
The **OCL-API v2.0** is an open definition for collaborative management of data dictionaries using a RESTful API. Data dictionaries consist of data definitions, terminology or indicators, which are collectively referred to as `concepts`. Relationships between concepts are defined in OCL as `mappings`. The API supports searching and editing concepts and mappings, building `sources` (a.k.a. data dictionaries), and logically grouping concepts and mappings into `collections`. Social features such as sharing, following, etc. are planned for subsequent phases. This is the Technical Documentation for the **OCL-API v1.0**.

A cloud-based instance of OCL is available at <https://openconceptlab.org>. The OCL User Documentation is available at <https://github.com/OpenConceptLab/ocl_web/wiki/>.

## Technical Overview
These slides provide a useful technical overview of OCL and will be updated regularly:
* [OCL Technical Overview](https://docs.google.com/presentation/d/1B0N_TlhBt54JdgqLXcH_NANnhQ-H39vm1IeFnf1kOI4/pub?start=false&loop=false&delayms=3000)
* [Overview of Versioning in OCL](https://docs.google.com/presentation/d/e/2PACX-1vROcmEkRnBGBXLOiznMwhfA6qhiRwWuc5-yoOZrc7XMhrxZ1Kvbuy4B_BEyfHmybl-SnoWrWrznPz58/pub?start=false&loop=false&delayms=10000)

## Implementation Notes
* All dates should follow ISO 8601 and be in UTC. Ex: `2011-11-16T14:26:15Z`
* All field/properties should follow the `underscore_spacing` convention
* All URL parameters follow are in `camelCase`
* All URL references within JSON are relative. For example, use `/orgs/WHO/sources/ICD-10-2010/` instead of `http://api.openconceptlab.com/orgs/WHO/sources/ICD-10-2010/`
* Use UTF-8 encoding

## API Endpoint
* `https://api.openconceptlab.org/`

## API Collections
These collections allow you to get up and running with the API within these clients.
### Postman  
[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/677ae58a3995c38e63af#?env%5BOpenMRS%20QA%5D=W3sia2V5IjoiVVJMIiwidmFsdWUiOiJodHRwczovL2FwaS5xYS5vcGVuY29uY2VwdGxhYi5vcmciLCJlbmFibGVkIjp0cnVlfV0=)  
This collection includes a `pre-request` script that sets a token before every request to save you from having to manually include this. Note, however, that this is set after any of your headers, so setting the `Authorization` header will be overridden by it. You can remove or adjust the token in this script under the collection `Pre-Request` Scripts tab.


## Authentication and Authorization
Clients must send an authentication token as a request header with every request. For example:
```
Authorization: Token 1234567890abcdeabcde1234567890abcdeabcde
```
The token is specific to each API user and is 40 characters long by default.

One can fetch the token by posting username and password to `https://api.openconceptlab.org/users/login/`. For example:

**Request:**
```
POST /users/login/
{
  "username": "MyUsername",
  "password": "1234567890",
}
```
**Response:**
```
Status: 200 OK
{
  "token": "1234567890abcdeabcde1234567890abcdeabcde"
}
```

## OCL REST Resources
The API exposes the following resources:
* **Owners**
    * `users` - An OCL user
    * `orgs` - An organization is a group of one or more users that collaborate in the management of sources and collections
* **Repositories**
    * `sources` - A source is where concepts and mappings are created. For example: Columbia eHealth International Laboratory Interface Terminology, WHO ICD-10, IHTSDO SNOMED-CT, or the WHO Indicator and Measurement Registry
    * `collections` - A collection is a logical grouping of concepts and mappings defined in sources
* **Content**
    * `concepts` - A concept is a term, definition, indicator, quality measure, etc.
    * `mappings` - A mapping is a relationship between 2 concepts. A mapping is "internal" if it points to a concept that is also defined in OCL. A mapping is "external" if it points to a concept that is not defined in OCL.

## Pagination and Limiting the Number of Returned Results
Which results are returned can be controlled using the `limit` and `page` parameters as follows:
* `limit` - default=10; set the number of results returned in a request. Set limit=0 to return all results.
* `page` - default=1; set the page number of the results returned

Additionally, headers containing pagination meta information are returned with the response, as follows:
* `next` - url to next page of results
* `num_found` - total number of results found
* `offset` - number of results on all previous pages combined
* `num_returned` - results returned from the current request
* `previous` - url to previous page of results

Examples:
```
# Fetch the first 15 results
GET /concepts/?q=malaria&limit=15

# Fetch the next 15 results
GET /concepts/?q=malaria&limit=15&page=2
```


## Sorting
Results may be sorted based on a single field using the `sortAsc` and `sortDesc` parameters. Note that not all fields support sorting. The documentation for each resource indicates which fields are supported.
* `sortAsc` - sort ascending
* `sortDesc` - sort descending

Examples:
```
GET /sources/?sortAsc={property1}
GET /sources/?sortDesc={property1}
```


## Filters
* Results may be filtered by many fields using the {field_name}={value}[,{value}[,{value}[...]] parameter. Filters may be combined. A comma-separated list may be used to specify more than one option for a single filter. For example, `datatype=Date,Datetime` will match concepts with datatypes of "Date" or "Datetime". Read the documentation for each resource for information about which field-level filters are supported.

Examples:
```
# Match concept datatype of Numeric OR Text
GET /concepts/?datatype=Numeric,Text

# Match source type of Dictionary
GET /sources/?sourceType=Dictionary
```

* Results may also be filtered by certain common metadata, such as `updatedSince`. Refer to each resource for additional metadata filters.
   * **updatedSince** - ISO 8601 timestamp (e.g. 2011-11-16T14:26:15Z)

Example:
```
GET /orgs/Columbia/sources/CIEL/concepts/?updatedSince=2014-11-16T14:26:15Z
```


## Facets
* Certain fields are implemented as facets, which means that the API optionally returns the possible options for a field along with the search results. Facets are not returned by default. To turn on facets, add "includeFacets: true" to the request header.

```
GET /concepts/?q=malaria&includeRetired=true
```
```json
{
    "facets": {
        "fields": {
            "retired": {
                "0": {
                    "0": "false",
                    "1": "44"
                },
                "1": {
                    "0": "true",
                    "1": "8"
                }
            },
            "conceptClass": {
                "0": {
                    "0": "Diagnosis",
                    "1": "42"
                },
                "1": {
                    "0": "Misc",
                    "1": "4"
                },
                "2": {
                    "0": "Question",
                    "1": "4"
                },
                "3": {
                    "0": "Misc Order",
                    "1": "1"
                },
                "4": {
                    "0": "Test",
                    "1": "1"
                },
                "5": {
                    "0": "Aggregate Measurement",
                    "1": "0"
                },
                "6": {
                    "0": "Anatomy",
                    "1": "0"
                }
            }
        }
    },
    "results": {
    }
}
```

## Resource Representations
Resources are returned in either **detail** or **summary** view:
* **detail** - Detailed view is returned when fetching a single object, or when the `verbose` URL parameter is set to `true`
* **summary** - Summary view is returned when fetching a list of objects



## Common Attributes
* **Audit Info** - all resources have the following audit fields:
    * **`created_by`** - username of the user that created the object
    * **`created_on`** - timestamp at which object was created
    * **`updated_on`** - username of the user that most recently updated the object
    * **`updated_by`** - timestamp at which object was most recently updated
* Additional common attributes (not shared by all resources)
    * `uuid` - Universally unique identifier for the resource
    * `url` - RESTful URL to the resource
    * `display_name` - Preferred display name based on current locale (default 'en')
    * `display_locale` - Locale of the display name
    * `retired` - Retired status of the resource (true/false)
    * `extras` - Optional custom attributes for a resource


## Response Codes
### HTTP Status Codes
* `200 OK` - All Indicates that the specified action was successfully completed. A 200 response indicates that the registry did successfully perform the operation and the response contains the final result of the action.
* `201 Created` - Indicates that a request was successful and as a result, a resource has been created
* `204 No Content` - Indicates that the request was received and that no content was returned
* `401 Unauthorized` - Raised when the client attempts to perform an operation against a resource which requires authorization. This error code indicates a challenge for client credentials.
* `403 Forbidden` - Indicates that the client does not have the necessary permission to perform the specified operation against the requested resource.
* `404 Not Found` - Indicates that a resource was not found or is not available.
* `405 Not Allowed` - Indicates that the requested operation is not allowed on the current resource (for example: DELETE on a collection)
* `409 Conflict` - Indicates that a conflict in the operation was detected and the operation was not performed.
* `410 Gone` - Indicates that a resource did exist but has been permanently removed.
* `500 Internal Server Error` - Indicates that the server encountered an error while attempting to execute the desired action.

### Success
Success differ from errors in that their body may not be a simple response object with a code and a message. The headers however are consistent across most calls:
* `GET`, `PUT`, `DELETE` return `200 OK` on success
* `POST` returns `201 Created` on success
Note that exports of repository versions use a different set of response codes that can be found in the export documentation.

### Error
Error responses are simply returning standard [HTTP error codes](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) along with some additional information:
* The error code is sent back as a status header
* The body includes an object describing both the code and message (for debugging and/or display purposes),
For example, for a call with when the resource is not found:

```
Status: 404 Not Found
```
```json
{
  "code": "404 Not Found",
  "message": "Resource not found"
}
```


## Response Formats
* JSON is used for all requests and responses by default, and an error occurs if an unsupported format is requested.
* JSON is the only format supported in the body of API **requests**.
* CSV **responses** are supported by some resources (documentation coming soon!).
* The "compress=true" header may be applied to any request to receive a zipped file of the JSON results. This is especially useful for exporting the results of a list query (e.g. GET /concepts/).


## API Versioning
API Documentation follows [semantic versioning](http://semver.org/).

API documentation revisions will be assigned a unique version number in the format MAJOR.MINOR.REVISION. These version numbers follow semantic versioning pattern whereby:
* **REVISION** is incremented for revisions to a MINOR version. These changes represent nonfunctional changes to the API.
* **MINOR** version numbers are incremented when new functionality is introduced which is backwards compatible with existing functionality in the MAJOR version. MINOR versions numbers are semantically compatible with previous MINOR versions.
* **MAJOR** version numbers are incremented when new functionality is introduced which is semantically incompatible with previous versions.

As new major versions of the code are released, prior versions will continue to be supported as long as they are still supported by the code.
