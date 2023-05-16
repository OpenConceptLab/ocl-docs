# Operation: $resolveReference

## Overview
The API exposes a global `$resolveReference` operation to resolve one or more relative and canonical references to their corresponding `Source Version` or `Collection Version` resources in the current OCL environment. The `$resolveReference` operation appropriately handles namespaces and the global canonical URL registry, implementing the same business logic that is used throughout OCL to resolve collection or mapping references, so that a user can test in advance exactly how a reference will be resolved within a particular context and can configure their namespace accordingly.

Note that this operation follows the FHIR convention of using the word “reference” to mean only a source or collection version, and “coding” to mean a source or collection version plus a specific code. That said, the `$resolveReference` operation supports both references and codings, which has the benefit of allowing a list of collection references to be processed directly using this operation.

OCL supports both "Inline" and "Expanded" reference syntaxes.

## Rules for Resolution of a Reference
OCL resolves a reference to a repository version (source, collection, codesystem, valueset, conceptmap) by following this process:
1. If relative reference, then convert to full URI by prepending the default OCL namespace: “http://api.openconceptlab.org”
2. If scope is set within a namespace: First, attempt to resolve the canonical URL within the namespace
3. If scope is global or the canonical URL could not be resolved based on the above rules (and if `oldstyle=false`): Attempt to resolve the canonical URL with the Global Canonical URL Registry
4. If the above rules do not resolve, then the reference cannot be resolved based on the current state of OCL

### Inline Reference Syntax
* Relative repository URL:
```
“/orgs/CIEL/sources/CIEL/”
```
* Relative URL with repository version:
```
“/orgs/CIEL/sources/CIEL/v2021-03-12/”
```
* Relative URL with coding:
```
“/orgs/CIEL/sources/CIEL/concepts/1948/”
“/orgs/CIEL/sources/CIEL/v2021-03-12/concepts/1948/”
```
* FHIR-formatted Canonical URL:
```
“http://hl7.org/fhir/CodeSystem/my-codesystem”
"https://CIELterminology.org"
```
* FHIR-formatted Canonical URL with piped CodeSystem version:
```
“http://hl7.org/fhir/CodeSystem/my-codesystem|1.2”
"https://CIELterminology.org|v2021-03-12"
```

### Expanded reference syntax:
* Relative URL (Note: Version is optional):
```
{
  “url”: “/orgs/CIEL/sources/CIEL/”,
  “version”: “v2021-03-12”
}
```
* Relative URL with coding (Note: Code is ignored by `$resolveReference`):
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

## Expanded Reference Syntax
* **url** (required) - A relative or canonical URL that points to a source, collection, CodeSystem, ValueSet or ConceptMap. If a canonical URL, a version may be piped, e.g. `http://hl7.org/fhir/CodeSystem/my-codeystem|0.8`. This field may also include a full inline reference expression, possibly including a relative URL or canonical URL, version, code, mapping id, filters, etc.
* **version** (optional) - The version of a repository to use. Note that in most situations it is best to omit this field and instead to specify a repository version in the expansion request.
* **filter** (optional) - An ordered list of filters used to select concepts or mappings by their properties. Note that the “filter” field cannot be combined with the “code” or “id” fields. Each filter has the following fields:
  * **property** - A property/filter defined by the code system
  * **op** - The type of operation to apply for the specified property. Examples: = | is-a | descendent-of | is-not-a | regex | in | not-in | generalizes | exists [see FilterOperator] – also see http://hl7.org/fhir/filter-operator
  * **value** - Code from the system, or regex criteria, or boolean value for exists
* **namespace** (optional) - Forces a reference to be evaluated within the context of a specific namespace. Generally, this value should be omitted and instead determined by the context of a request, which would take into account possible namespace configuration and OCL’s global canonical URL registry (which doesn’t currently exist).
* **includeExclude** (optional) - default=”Include”; set to “Exclude” to make this an exclusion. All inclusions are processed first, and then all exclusions are processed. The rules for processing inclusions and exclusions are otherwise the same.
* **resourceType** (optional) - default=”Concept” if blank. Set this field to “Mapping” to retrieve a mapping resource instead of a Concept. Note that Mapping references cannot be represented via FHIR.
* If `resourceType=Concept`:
  * **code** (optional) - A concept code to include in a collection. Note that the code field cannot be used in combination with the filter field.
  * **display** (optional) - A display name that overrides a concept’s default display_name within the expansion. If this field is used, then the concept’s display_locale is always None (or an empty string). The “display” field may only be used when the “code” field is used.
  * **cascade** (optional) - TBD
* If `resourceType=Mapping`:
  * **id** (optional) - A mapping id to include in a collection. Note that the id field cannot be used in combination with the filter field.

## $resolveReference Parameters
* **oldstyle** (optional) - default=false; set to True to evaluate version-less repository references to HEAD instead of to latest
* **namespace** (optional) - default="Global"; the context in which to evaluate the references, e.g. a relative URL to an organization or user in OCL. This parameter allows a client to compare the results of evaluating the same reference(s) in multiple namespaces. When this operation is performed internally, namespace is typically set automatically based on the context of the request, i.e. the owner of the resource under consideration.

### $resolveReference Examples
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
* Add an internal switch in `$resolveReference` that is set to resolve version-less "relative URLs" to HEAD that we can change to latest when we're ready -- note that version-less "canonical URLs" should always resolve to latest
* In the future, we may want to implement an optional URL parameter that returns the full list of repositories that OCL could resolve a URL to across all namespaces that a requesting user has access to
