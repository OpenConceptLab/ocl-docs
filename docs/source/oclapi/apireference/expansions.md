# Collection Expansions

## Overview

The API exposes an expansion subresource, as part of collections, which represents an evaluated set of concepts and, optionally, associations. OCL uses a collection’s references and a set of parameters to determine how an expansion is evaluated. References can point directly to resources (e.g. “/orgs/WHO/sources/ICD-10-WHO/concepts/K14.0/”), or they can provide logic to find one or more resources (e.g. “/orgs/WHO/sources/ICD-10-WHO/concepts/?q=Glossitis”). A collection version, which consists of a set of references, can then be evaluated as an expansion. Expansions contain the results of the evaluated reference(s) at that point in time, based on a set of optional parameters (e.g. “published since date:2020-03-31”).

Expansions can be generated manually, both in the API and in the TermBrowser, or collections can be “auto-expanded”. An auto-expanded collection will be reactive to references that are added to the collection’s HEAD version, and it will update the automatically generated expansion with every change in references. No parameters are specified in this auto-expansion.

The list of supported expansion parameters are listed below, along with their current development status in OCL. 

## Generate an expansion for a collection version

```
POST /orgs/:org/collections/:collection/:collectionVersion/expansions/
POST /users/:user/collections/:collection/:collectionVersion/expansions/
POST /user/collections/:collection/:collectionVersion/expansions/
```
* Input
    * `mnemonic` (required) string - used to identify the expansion in the URL (usually an acronym e.g. Community-MCH)
    * `parameters` (required) dict - specifies which parameters will be used to evaluate the expansion. See below for parameter list, and see table below for more descriptions.
        * `filter` (optional) string 
        * `exclude-system` (optional) string
        * `system-version` (optional) string
        * `date` (optional) string
        * `count` (optional) numeric
        * `offset` (optional) numeric
        * `activeOnly` (optional) boolean
        * `includeDesignations` (optional) boolean
        * `includeDefinition` (optional) boolean
        * `excludeNested` (optional) boolean
        * `excludeNotForUI` (optional) boolean
        * `excludePostCoordinated` (optional) boolean
        * `check-system-version` (optional) string
        * `force-system-version` (optional) string

### Request Payload example
```
{
    "mnemonic": "test",
    "parameters": {
        "filter": "a",
        "exclude-system": "as",
        "system-version": "asd",
        "date": "2020",
        "count": 0,
        "offset": 0,
        "activeOnly": true,
        "includeDesignations": true,
        "includeDefinition": false,
        "excludeNested": false,
        "excludeNotForUI": true,
        "excludePostCoordinated": true,
        "check-system-version": "",
        "force-system-version": ""
    }
}
```
### Response
```
Status: 201 Created
```
```
{
    "mnemonic": "test",
    "id": 2457,
    "parameters": {
        "filter": "a",
        "exclude-system": "as",
        "system-version": "asd",
        "date": "2020",
        "count": 0,
        "offset": 0,
        "activeOnly": true,
        "includeDesignations": true,
        "includeDefinition": false,
        "excludeNested": false,
        "excludeNotForUI": true,
        "excludePostCoordinated": true,
        "check-system-version": "",
        "force-system-version": ""
    },
    "canonical_url": null,
    "url": "/users/ocladmin/collections/testjoe14Apr22/HEAD/expansions/test/",
    "is_processing": false
}
```

### Expansion Parameters
Adapted from [FHIR’s value set expansion specification](https://www.hl7.org/fhir/valueset-operation-expand.html):

|     Input Parameter    | Development Timeline |                                                                                                                                                                                                                                                                                                                                                                                                                                 Description                                                                                                                                                                                                                                                                                                                                                                                                                                |
|:----------------------:|:--------------------:|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| url                    | Done                 | A canonical reference to a value set. The server must know the value  set (e.g. it is defined explicitly in the server's value sets, or it is  defined implicitly by some code system known to the server                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| valueSet               | TBD                  | The value set is provided directly as part of the request. Servers may choose not to accept value sets in this fashion                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| valueSetVersion        | Done                 | The identifier that is used to identify a specific version of the  value set to be used when generating the expansion. This is an arbitrary  value managed by the value set author and is not expected to be  globally unique. For example, it might be a timestamp (e.g. yyyymmdd) if  a managed version is not available.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| context                | TBD                  | The context of the value set, so that the server can resolve this to  a value set to expand. The recommended format for this URI is  [Structure Definition URL]#[name or path into structure definition] e.g.   http://hl7.org/fhir/StructureDefinition/observation-hspc-height-hspcheight#Observation.interpretation.  Other forms may be used but are not defined. This form is only usable  if the terminology server also has access to the conformance registry  that the server is using, but can be used to delegate the mapping from  an application context to a binding at run-time                                                                                                                                                                                                                                                                              |
| contextDirection       | TBD                  | If a context is provided, a context direction may also be provided.  Valid values are: - 'incoming': the codes a client can use for PUT/POST  operations, and - 'outgoing', the codes a client might receive from the  server. The purpose is to inform the server whether to use the value set  associated with the context for reading or writing purposes (note: for  most elements, this is the same value set, but there are a few elements  where the reading and writing value sets are different)                                                                                                                                                                                                                                                                                                                                                                  |
| filter                 | Deployed - testing   | A text filter that is applied to restrict the codes that are  returned (this is useful in a UI context). The interpretation of this is  delegated to the server in order to allow to determine the most optimal  search approach for the context. The server can document the way this  parameter works in TerminologyCapabilities.expansion.textFilter. Typical  usage of this parameter includes functionality like: * using left  matching e.g. "acut ast" * allowing for wild cards such as %, &, ? *  searching on definition as well as display(s) * allowing for search  conditions (and / or / exclusions) Text Search engines such as Lucene or  Solr, long with their considerable functionality, might also be used.  The optional text search might also be code system specific, and servers  might have different implementations for different code systems |
| date                   | Deployed - testing   | The date for which the expansion should be generated. if a date is  provided, it means that the server should use the value set / code  system definitions as they were on the given date, or return an error if  this is not possible. Normally, the date is the current conditions  (which is the default value) but under some circumstances, systems need  to generate an expansion as it would have been in the past. A typical  example of this would be where code selection is constrained to the set  of codes that were available when the patient was treated, not when the  record is being edited. Note that which date is appropriate is a matter  for implementation policy.                                                                                                                                                                                |
| offset                 | Upcoming             | Paging support - where to start if a subset is desired (default = 0). Offset is number of records (not number of pages)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| count                  | Upcoming             | Paging support - how many codes should be provided in a partial page  view. Paging only applies to flat expansions - servers ignore paging if  the expansion is not flat. If count = 0, the client is asking how large  the expansion is. Servers SHOULD honor this request for hierarchical  expansions as well, and simply return the overall count                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| includeDesignations    | Upcoming             | Controls whether concept designations are to be included or excluded in value set expansions                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| designation            |                      | A token that specifies a system+code that is either a use or a  language. Designations that match by language or use are included in the  expansion. If no designation is specified, it is at the server  discretion which designations to return                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| includeDefinition      | Upcoming             | Controls whether the value set definition is included or excluded in value set expansions                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| activeOnly             | Deployed - testing   | Controls whether inactive concepts are included or excluded in value  set expansions. Note that if the value set explicitly specifies that  inactive codes are included, this parameter can still remove them from a  specific expansion, but this parameter cannot include them if the value  set excludes them                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| excludeNested          | TBD                  | Controls whether or not the value set expansion nests codes or not (i.e. ValueSet.expansion.contains.contains)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| excludeNotForUI        | TBD                  | Controls whether or not the value set expansion is assembled for a  user interface use or not. Value sets intended for User Interface might  include 'abstract' codes or have nested contains with items with no code  or abstract = true, with the sole purpose of helping a user navigate  through the list efficiently, where as a value set not generated for UI  use might be flat, and only contain the selectable codes in the value  set. The exact implications of 'for UI' depend on the code system, and  what properties it exposes for a terminology server to use. In the FHIR  Specification itself, the value set expansions are generated with  excludeNotForUI = false, and the expansions used when generated schema /  code etc, or performing validation, are all excludeNotForUI = true.                                                             |
| excludePostCoordinated | TBD                  | Controls whether or not the value set expansion includes post coordinated codes                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| displayLanguage        | Deployed - testing   | Specifies the language to be used for description in the expansions  i.e. the language to be used for ValueSet.expansion.contains.display                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| exclude-system         | Deployed - testing   | Code system, or a particular version of a code system to be excluded  from the value set expansion. The format is the same as a canonical  URL: [system]\|[version] - e.g. http://loinc.org\|2.56                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| system-version         | Deployed - testing   | Specifies a version to use for a system, if the value set does not  specify which one to use. The format is the same as a canonical URL:  [system]\|[version] - e.g. http://loinc.org\|2.56                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| check-system-version   | Upcoming             | Edge Case: Specifies a version to use for a system. If a value set  specifies a different version, an error is returned instead of the  expansion. The format is the same as a canonical URL: [system]\|[version]  - e.g. http://loinc.org\|2.56                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| force-system-version   | Upcoming             | Edge Case: Specifies a version to use for a system. This parameter  overrides any specified version in the value set (and any it depends  on). The format is the same as a canonical URL: [system]\|[version] -  e.g. http://loinc.org\|2.56. Note that this has obvious safety issues, in  that it may result in a value set expansion giving a different list of  codes that is both wrong and unsafe, and implementers should only use  this capability reluctantly. It primarily exists to deal with situations  where specifications have fallen into decay as time passes. If the  value is override, the version used SHALL explicitly be represented in  the expansion parameters                                                                                                                                                                                  |


## Get a list of expansions for a collection version
```
GET /orgs/:org/collections/:collection/:collectionVersion/expansions/
GET /users/:user/collections/:collection/:collectionVersion/expansions/
GET /user/collections/:collection/:collectionVersion/expansions/
```

* Input
    * `includeSummary` (optional) boolean - default is false; set to true to return summary counts of active concepts and mappings
    * `verbose` (optional) boolean - default is false; set to true to return full concept details instead of the summary

### Response
Status: 200 OK

```
[
    {
      "mnemonic": "test",
      "id": 2457,
      "parameters": {
        "date": "2020",
        "count": 0,
        "filter": "a",
        "offset": 0,
        "activeOnly": true,
        "excludeNested": false,
        "exclude-system": "as",
        "system-version": "asd",
        "excludeNotForUI": true,
        "includeDefinition": false,
        "includeDesignations": true,
        "check-system-version": "",
        "force-system-version": "",
        "excludePostCoordinated": true
      },
      "canonical_url": null,
      "url": "/users/ocladmin/collections/testjoe14Apr22/HEAD/expansions/test/",
      "summary": {
        "active_concepts": 10,
        "active_mappings": 10
      },
      "created_on": "2022-04-29T14:34:44.284572Z",
      "created_by": "ocladmin",
      "is_processing": false
    },
    {
      "mnemonic": "v4-once-more",
      "id": 2456,
      "parameters": {
        "date": "",
        "count": 0,
        "filter": "",
        "offset": 0,
        "activeOnly": false,
        "excludeNested": false,
        "exclude-system": "",
        "system-version": "http://test.com/fakeurl|v4",
        "excludeNotForUI": true,
        "includeDefinition": false,
        "includeDesignations": true,
        "check-system-version": "",
        "force-system-version": "",
        "excludePostCoordinated": true
      },
      "canonical_url": null,
      "url": "/users/ocladmin/collections/testjoe14Apr22/HEAD/expansions/v4-once-more/",
      "summary": {
        "active_concepts": 4,
        "active_mappings": 3
      },
      "created_on": "2022-04-28T15:23:19.439663Z",
      "created_by": "ocladmin",
      "is_processing": false
    }
]
```

## Get an expansion for a collection version
```
GET /orgs/:org/collections/:collection/:collectionVersion/expansions/:expansionID
GET /users/:user/collections/:collection/:collectionVersion/expansions/:expansionID
GET /user/collections/:collection/:collectionVersion/expansions/:expansionID
```
* Input
    * `includeSummary` (optional) boolean - default is false; set to true to return summary counts of active concepts and mappings
    * `verbose` (optional) boolean - default is false; set to true to return full concept details instead of the summary

### Response
Status: 200 OK
```
{
  "mnemonic": "test",
  "id": 2457,
  "parameters": {
    "date": "2020",
    "count": 0,
    "filter": "a",
    "offset": 0,
    "activeOnly": true,
    "excludeNested": false,
    "exclude-system": "as",
    "system-version": "asd",
    "excludeNotForUI": true,
    "includeDefinition": false,
    "includeDesignations": true,
    "check-system-version": "",
    "force-system-version": "",
    "excludePostCoordinated": true
  },
  "canonical_url": null,
  "url": "/users/ocladmin/collections/testjoe14Apr22/HEAD/expansions/test/",
  "summary": {
    "active_concepts": 10,
    "active_mappings": 10
  },
  "is_processing": false
}
```

## Delete an expansion for a collection version
```
DELETE /orgs/:org/collections/:collection/:collectionVersion/expansions/:expansionID
DELETE /users/:user/collections/:collection/:collectionVersion/expansions/:expansionID
DELETE /user/collections/:collection/:collectionVersion/expansions/:expansionID
```

### Response
Status: 204 No Content

