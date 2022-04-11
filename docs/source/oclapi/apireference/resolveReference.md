# Operation: $resolveReference

## Overview
The API exposes a global `$resolveReference` operation to resolve one or more relative and canonical references to a `Source Version` or `Collection Version` that smartly handles namespaces and the global canonical URL registry. The `$resolveReference` operation implements the same behavior that is used throughout OCL to resolve collection or mapping references, so that a user can test in advance exactly how a reference will be resolved and can configure their namespace accordingly.

Note that this operation follows the FHIR convention of using the word “reference” to mean only a source or collection version, and “coding” to mean a source or collection version plus a specific code. For simplicity, the $resolveReference operation supports both, which means that a list of collection references may be processed directly by this operator.

OCL supports the following types of references:
* Inline reference syntax:
  * Relative URL:
```
“/orgs/CIEL/sources/CIEL/”
```
  * Relative URL with source version: 
```
“/orgs/CIEL/sources/CIEL/v2021-03-12/”
```
  * Relative URL with coding: 
```
“/orgs/CIEL/sources/CIEL/concepts/1948/”
```
  * Canonical URL: 
```
“http://hl7.org/fhir/CodeSystem/my-codesystem”
```
  * Canonical URL with piped CodeSystem version: 
```
“http://hl7.org/fhir/CodeSystem/my-codesystem|1.2”
```

* Expanded reference syntax:
  * Relative URL (Note: Version is optional):
```
{
  “url”: “/orgs/CIEL/sources/CIEL/”,
  “version”: “v2021-03-12”
}
```
  * Relative URL with coding (Note: Code is ignored by $resolveReference):
```
{
  “url”: “/orgs/CIEL/sources/CIEL/”,
  “version”: “v2021-03-12”,
  “code”: “1948”
}
```
  * Canonical URL with version, namespace and code:
```
{
  “url”: “http://hl7.org/fhir/CodeSystem/my-codeystem”,
  “version”: “0.8”,
  “code”: “1948”,
  “namespace”: “/orgs/MyOrg/”
}
```
  * Canonical URL with piped version in the global namespace:
```
{
  “url”: “http://hl7.org/fhir/CodeSystem/my-codeystem|0.8”
}
```

## Rules for Resolution of a Reference
The following rules specify how OCL resolves where a reference to a repository (source, collection, codesystem, valueset, conceptmap) points to:
1. If relative reference, then convert to full URI by prepending the default OCL namespace: “http://api.openconceptlab.org”
2. If scope is set within a namespace: First, attempt to resolve the canonical URL within the namespace
3. If scope is global or the canonical URL could not be resolved within the namespace: Attempt to resolve the canonical URL with the Global Canonical URL Registry
4. If the above rules do not resolve, then the reference cannot be resolved based on the current state of OCL

## Examples
* Resolve a single reference:
```
POST https://api.qa.openconceptlab.org/$resolveReference/
“/orgs/CIEL/sources/CIEL/concepts/1948/”
```
* Resolve multiple references in one request – embed each request in an ordered list:
```
POST https://api.qa.openconceptlab.org/$resolveReference/
[
  “/orgs/CIEL/sources/CIEL/concepts/1948/”,
  {"url": "/orgs/CIEL/sources/CIEL/concepts/123/", "namespace": "/orgs/CIEL/", "version": "v2021-03-12"},
  {“url”: “http://hl7.org/fhir/CodeSystem/my-codeystem”, “version”: “0.8”, “code”: “1948”, “namespace”: “/orgs/MyOrg/”}
]
```

### Response
The operation responds with a list of the results for each reference submitted, including the corresponding Source Version or Collection Version summary if the resolution was successful. Output parameters:
* **reference_type**: relative or canonical
* **timestamp**: date+time of the request
* **resolved**: boolean - whether or not OCL was able to resolve the reference
* **request**: a copy of the original reference that was requested to be resolved
* **result**: the Source Version or Collection Version

```
Status: 200
```
```
[
  {
    "reference_type": "relative",
    "timestamp": "2022-03-01T13:12:18.919301",
    "resolved": true,
    "request": “/orgs/CIEL/sources/CIEL/concepts/1948/”,
    "result": {
      "type": "Source Version",    # or Collection Version
      <repository version summary>
    }
  },
  {
    "reference_type": "relative",
    "timestamp": "2022-03-01T13:12:18.919301",
    "request": {"url": "/orgs/CIEL/sources/CIEL/concepts/123/", "namespace": "/orgs/CIEL/", "version": "v2021-03-12"},
    "resolved": true,
    "result": {
      "type": "Source Version",    # or Collection Version
      <repository version summary>
    }
  },
  {
    "reference_type": "canonical",
    "timestamp": "2022-03-01T13:12:18.919301",
    "request": {“url”: “http://hl7.org/fhir/CodeSystem/my-codeystem”, “version”: “0.8”, “code”: “1948”, “namespace”: “/orgs/MyOrg/”},
    "resolved": true,
    "result": {
      "type": "Source Version",    # or Collection Version
      <repository version summary>
    }
  }
]
```


## Additional Considerations
* Add an internal switch in $resolveReference that is set to resolve version-less "relative URLs" to HEAD that we can change to latest when we're ready -- note that version-less "canonical URLs" should always resolve to latest
* Add an optional "namespace" URL parameter that overrides whatever is in the body -- this would make it easier to see how references that have namespaces already set would behave if evaluated using an alternate namesapce
