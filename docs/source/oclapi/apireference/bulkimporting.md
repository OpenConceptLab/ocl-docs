# Bulk Import API

## Overview
OCL exposes a method for submitting an OCL-formatted bulk import JSON file to the OCL API that is processed asynchronously on the server. Note that the OCL bulk import methods do not currently support FHIR resources, but 
A bulk import file may include creates, updates, or deletes for multiple owners and repositories. This approach is significantly more efficient than using individual REST API calls to modify or create one resource at a time. A bulk import file is processed using the credentials provided in the bulk import request.

### Authorization
The bulk importer processes a bulk import script using the credentials provided in the bulk import request (eg. the `Authorization` request header). All actions taken by the bulk importer use these credentials, meaning that the user must have the required permissions for each action. This includes GET requests that the bulk importer submits to determine whether resources already exist in OCL.
The header uses the following key/value:
* Key: “Authorization”
* Value: “Token [API token]”
The API token can be received from OCL’s TermBrowser UI on your Profile page, once you have created and logged into an OCL account.

### Parallel vs. Asynchronous Processing of Bulk Imports
By default, OCL attempts to process bulk imports in parallel using multiple workers where it can, providing a significant performance improvement. OCL will process a sequential list of resources of the same type, eg `Concept` or `Mapping`, in parallel, pausing before moving onto a resource of a different type. For example, if a bulk import script contains 5 concepts and 5 mappings, in that order, the 5 concepts would be processed in parallel and then the 5 mappings would be processed in parallel after the concepts had all been processed.

Note that any `DELETE` action will occur in sequence and finish processing before moving to the next resource.

## Bulk Import File Formats
Two types of Bulk Import files are currently supported for OCL: CSV and [JSON Lines](https://jsonlines.org/) files. Both file types support Bulk Importing of multiple resource types. In CSV files, each row represents an OCL resource, with columns representing the attributes. In JSON Lines files, each line is an OCL-formatted JSON resource.

Regardless of format, when creating resources using Bulk Imports, each type of OCL resource has required and optional fields that can be used. The summary of required and optional fields is listed below, but here are some basic rules for Bulk Importing into OCL:
* Each resource must specify a valid resource type, e.g. `Concept`, `Source`, or `Organization`. In CSV, this is specified with the `resource_type` attribute. In OCL-formatted JSON, use the `type` attribute.
* For all resources other than orgs and users, each resource must define an owner and, if applicable, a repository. These are defined using one or more of these attributes: `owner`, `owner_type`, `source`, `collection`.
* Each resource may optionally provide processing directives. Currently supported processing directives are:
   * `__action`: There are 4 action types supported:
      * `CREATE_OR_UPDATE` (default) - By default, the bulk importer will attempt to update a resource if it already exists; otherwise it will try to create a new resource.
      * `CREATE` - The bulk importer will attempt to create a new resource without first checking if it already exists
      * `UPDATE` - The bulk importer will attempt to update a resource without first checking if it already exists
      * `DELETE` - The bulk importer will attempt to delete a resource without first checking if it already exists
      * `SKIP` (not currently implemented) - The bulk importer will skip the resource
      * `DELETE_IF_EXISTS` (not currently implemented) - The bulk importer will attempt to delete a resource if it confirms that it exists
   * `__cascade`: For resources of type `Reference`, it is possible to specify whether and how mappings are cascaded:
      * `None` (default) - No cascading will occur. Only the
      * `sourcemappings` - Mappings stored in the same source whose `from_concept` matches a concept that is being added to a collection will also be added

### OCL-formatted JSON Format Example
Link: [https://drive.google.com/file/d/1n1wC5-w4fYKNDx5uViQ5MaaAokHuBOn8/view?usp=sharing](https://drive.google.com/file/d/1n1wC5-w4fYKNDx5uViQ5MaaAokHuBOn8/view?usp=sharing) 

```
{"type": "Organization", "id": "DemoOrg", "name": "My Demo Organization", "company": "DemoLand Inc.", "website": "www.demoland.fake", "location": "DemoLand", "public_access": "View", "logo_url": "https://thumbs.dreamstime.com/b/demo-icon-demo-147077326.jpg", "description": "Generic Demo description text", "text": "This organization is demo-tastic!", "extras": {"Ex_Num":"6", "extra_names": [{"name": "Demotastic Name", "short_name": "demo"}, {"name": "Out-of-Date Demo Name", "short_name": "old"}]}}
{"type": "Source", "id": "MyDemoSource", "name": "My Test Source", "full_name": "My Demonstrative Test Source", "owner": "DemoOrg", "owner_type": "Organization", "description": "Using this source just for testing purposes", "source_type": "Dictionary", "public_access": "Edit", "default_locale": "en", "supported_locales": ["en","fk"], "custom_validation_schema": "None", "external_id ": "164531246546-IDK", "website": "www.demoland.fake/source", "extras": {"ex_name": "Source Name"}, "canonical_url": "https://demo.fake/CodeSystem/Source", "hierarchy_meaning": "is-a", "hierarchy_root_url": "/orgs/DemoOrg/sources/MyDemoSource/concepts/Act/"}
{"type": "Concept", "retired": false, "datatype": "None", "concept_class": "Misc", "source": "MyDemoSource", "extras": null, "descriptions": [{"description":"Just one description","locale":"en"}], "owner": "DemoOrg", "owner_type": "Organization", "external_id": "HSpL3hSBx6F", "id": "Act", "names": [{"locale": "en", "locale_preferred": true, "external_id": null, "name": "Active Demo Concept", "name_type": "Fully Specified"}]}
{"type": "Concept", "retired": true, "datatype": "None", "concept_class": "Misc", "source": "MyDemoSource", "extras": null, "descriptions": null, "owner": "DemoOrg", "owner_type": "Organization", "external_id": "HSpL3hSBx6F", "id": "Ret", "names": [{"locale": "en", "locale_preferred": true, "external_id": null, "name": "Retired Demo Concept", "name_type": "Fully Specified"}]}
{"type": "Concept", "retired": false, "datatype": "None", "concept_class": "Misc", "source": "MyDemoSource", "extras": null, "descriptions": [{"description":"Main description","locale":"en","locale_preferred":true,"type":"IDK","external_id":"123456"},{"description":"Secondary description","locale":"en","locale_preferred":true,"type":"IDK","external_id":"234567"}], "owner": "DemoOrg", "owner_type": "Organization", "external_id": "HSpL3hSBx6F", "id": "Child", "names": [{"locale": "en", "locale_preferred": true, "external_id": null, "name": "Child Demo Concept", "name_type": "Fully Specified"}], "parent_concept_urls":["/orgs/DemoOrg/sources/MyDemoSource/concepts/Act/"]}
{"type": "Concept", "retired": false, "datatype": "None", "concept_class": "Misc", "source": "MyDemoSource", "extras": null, "descriptions": null, "owner": "DemoOrg", "owner_type": "Organization", "external_id": "asdkfjhasLKfjhsa", "id": "Child_of_child", "names": [{"locale": "en", "locale_preferred": true, "external_id": null, "name": "Child of the Child Demo Concept", "name_type": "Fully Specified"}], "parent_concept_urls":["/orgs/DemoOrg/sources/MyDemoSource/concepts/Child/"],"mappings":[{"map_target":"Internal","map_type":"Child-Parent","to_concept_url":"/orgs/DemoOrg/sources/MyDemoSource/concepts/Child/"}]}
{"type":"Mapping","map_type":"Parent-child","to_concept_url":"/orgs/DemoOrg/sources/MyDemoSource/concepts/Child/","from_concept_url":"/orgs/DemoOrg/sources/MyDemoSource/concepts/Act/","source":"MyDemoSource","owner_type":"Organization","owner":"DemoOrg"}
{"type": "Collection", "id": "MyDemoCollection", "name": "My Test Collection", "full_name": "My Demonstrative Test Collection", "owner": "DemoOrg", "owner_type": "Organization", "description": "Using this collection just for testing purposes", "collection_type": "Value Set", "public_access": "Edit", "default_locale": "en", "supported_locales": ["en","fk"], "custom_validation_schema": "None", "external_id": "654246546-IDK", "website": "www.demoland.fake/source", "extras": {"ex_name": "Collection Name"}, "canonical_url": "https://demo.fake/ValueSet/Collection", "publisher": "DemoLand, Inc.", "purpose": "To demonstrate", "copyright": "Please don't use this for anything but test importing.", "immutable": false, "revision_date": "2021-07-09", "logo_url": "https://thumbs.dreamstime.com/b/demo-icon-demo-147077326.jpg", "text": "Generic About entry", "experimental": true, "jurisdiction": ["DZA", "EGY"], "contact": [{"telecom" : [{"system" : "url", "value" : "http://demoland.fake/fhir"}]}], "identifier": [{"system" : "Fake System", "value" : "Fake Value"}]}
{"type" : "Reference", "collection_url" : "/orgs/DemoOrg/collections/MyDemoCollection/", "data": {"expressions" : ["/orgs/DemoOrg/sources/MyDemoSource/concepts/Act/","/orgs/DemoOrg/sources/MyDemoSource/concepts/Ret/","/orgs/DemoOrg/sources/MyDemoSource/concepts/Unresolved/"]}}
```


### CSV Format Example

Link to example: [https://drive.google.com/file/d/1lmK0qDlDJU4Mth__gCeSkPkiON0c0I02/view?usp=sharing](https://drive.google.com/file/d/1lmK0qDlDJU4Mth__gCeSkPkiON0c0I02/view?usp=sharing) 

```
resource_type,id,name,company,website,location,public_access,logo_url,description,text,attr:Ex_Num,attr:ex_name,full_name,owner_id,owner_type,source_type,default_locale,supported_locales,custom_validation_schema,external_id,canonical_url,hierarchy_meaning,hierarchy_root_url,internal_reference_id,meta,collection_reference,publisher,purpose,copyright,revision_date,experimental,jurisdiction,content_type,case_sensitive,compositional,version_needed,external_id,retired,datatype,concept_class,source,description[1],description[2],name[1],name_type[1],attr:extra_names:list,attr:extra_bool:bool,attr:extra_float:float,attr:extra_int:int,parent_concept_urls[0],map_type[0],map_from_concept_id[0],map_to_concept_id[0],map_type,to_concept_url,from_concept_url,attr:extra_names,collection_type,immutable,jurisdiction[1],jurisdiction[2],collection_url,data:expressions
Organization,DemoOrg,My Demo Organization,DemoLand Inc.,https://www.demoland.fake,DemoLand,View,https://thumbs.dreamstime.com/b/demo-icon-demo-147077326.jpg,Generic Demo description text,This organization is demo-tastic!,6,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
Source,MyDemoSource,My Test Source,,https://www.demoland.fake/source,,Edit,https://thumbs.dreamstime.com/b/demo-icon-demo-147077326.jpg,Using this source just for testing purposes,,,Source Name,My Demonstrative Test Source,DemoOrg,Organization,Dictionary,en,"en,fk",None,164531246546-IDK,https://demo.fake/CodeSystem/Source,is-a,/orgs/DemoOrg/sources/MyDemoSource/concepts/Act/,askjdhbas,IDK,/orgs/DemoOrg/collections/MyDemoCollection/,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
Source,MyFHIRSource,My FHIR Source,,https://www.demoland.fake/source,,Edit,,Using this source just for FHIR testing purposes,,,FHIR Source Name,My Demonstrative FHIR Test Source,DemoOrg,Organization,Dictionary,en,"en,fk",None,FHIR1641246546-IDK,https://demo.fake/CodeSystem/FHIRSource,,,,,,"DemoLand, Inc.",Only for testing,For testing only,2021-07-27,TRUE,[Record],example,TRUE,FALSE,TRUE,,,,,,,,,,,,,,,,,,,,,,,,,,,
Concept,Act,,,,,,,,,,,,DemoOrg,Organization,,,,,HSpL3hSBx6F,,,,,,,,,,,,,,,,,,FALSE,None,Misc,MyDemoSource,Just one description,,Active Demo Concept,Fully Specified,"[name1,name2]",TRUE,2.5,5,,,,,,,,,,,,,,
Concept,Ret,,,,,,,,,,,,DemoOrg,Organization,,,,,HSpL3hSBx6F,,,,,,,,,,,,,,,,,,TRUE,None,Misc,MyDemoSource,,,Retired Demo Concept,Fully Specified,,,,,,,,,,,,,,,,,,
Concept,Child,,,,,,,,,,,,DemoOrg,Organization,,,,,HSpL3hSBx6F,,,,,,,,,,,,,,,,,,FALSE,None,Misc,MyDemoSource,,,Child Demo Concept,Fully Specified,,,,,/orgs/DemoOrg/sources/MyDemoSource/concepts/Act/,,,,,,,,,,,,,
Concept,Child_of_child,,,,,,,,,,,,DemoOrg,Organization,,,,,asdkfjhasLKfjhsa,,,,,,,,,,,,,,,,,,FALSE,None,Misc,MyDemoSource,Main description,Secondary description,Child of the Child Demo Concept,Fully Specified,,,,,/orgs/DemoOrg/sources/MyDemoSource/concepts/Child/,Child-Parent,/orgs/DemoOrg/sources/MyDemoSource/concepts/Child_of_child/,/orgs/DemoOrg/sources/MyDemoSource/concepts/Child/,,,,,,,,,,
Mapping,,,,,,,,,,,,,DemoOrg,Organization,,,,,,,,,,,,,,,,,,,,,,,,,,MyDemoSource,,,,,,,,,,,,,Parent-child,/orgs/DemoOrg/sources/MyDemoSource/concepts/Child/,/orgs/DemoOrg/sources/MyDemoSource/concepts/Act/,,,,,,,
Collection,MyDemoCollection,My Test Collection,,https://www.demoland.fake/source,,Edit,https://thumbs.dreamstime.com/b/demo-icon-demo-147077326.jpg,Using this collection just for testing purposes,Generic About entry,,Collection Name,My Demonstrative Test Collection,DemoOrg,Organization,,en,"en,fk",None,654246546-IDK,https://demo.fake/ValueSet/Collection,,,,,,"DemoLand, Inc.",To demonstrate,Please don't use this for anything but test importing.,44386,TRUE,,,,,,,,,,,,,,,,,,,,,,,,,,,Value Set,FALSE,DZA,EGY,,
Reference,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,/orgs/DemoOrg/collections/MyDemoCollection/,/orgs/DemoOrg/sources/MyDemoSource/concepts/Act/
Reference,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,/orgs/DemoOrg/collections/MyDemoCollection/,/orgs/DemoOrg/sources/MyDemoSource/concepts/Ret/
Reference,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,/orgs/DemoOrg/collections/MyDemoCollection/,/orgs/DemoOrg/sources/MyDemoSource/concepts/Unresolved/
```


## Required and Optional Bulk Import Fields
### Organization

**Required**
* **resource_type** - “Organization”
* **id** - OCL resource identifier for the organization
* **name** - Name of the organization

**Optional**
* **company** - Group or organization that owns the organization resource
* **website** - URL of the organization’s main website
* **location** - State, country, etc. of the organization
* **public_access** - Allows users outside of the organization to “View” or “Edit” the Organization and its resources, or the organization can be hidden from unauthorized users by setting this to “None” (default=”View”) 
* **Extra attributes** - custom attributes that are outside of OCL’s model
* **logo_url** - URL of logo image for this organization
* **description** - Description of organization
* **text** - “About” text for organization
### Repositories (Sources and Collections)
**Required**
* **resource_type** - “Source” or “Collection”
* **owner** - OCL resource identifier of the OCL user or organization that will own this object
* **id** - OCL resource identifier
* **name** - Primary name of the repository
* **owner_type** - “Organization” or “User”

**Optional**
* **short_code** - Automatically set to id if omitted
* **full_name** - Automatically set to name if omitted
* **description** - Description of the repository
* **external_id** - Optional external identifier for the repository
* **source_type** or **collection_type** - eg. “Dictionary”, “Interface Terminology”, “Indicator Registry”, “Code List”, “Subset”, etc.
* **default_locale** - default=”en”
* **supported_locales** - default=”en”
* **website** - URL of the main website for the source
* **custom_validation_schema** - 
* **public_access** - Allows users outside of the organization to “View” or “Edit” the Organization and its resources, or the organization can be hidden from unauthorized users by setting this to “None” (default=”View”)
* **Extra attributes** - custom attributes that are outside of OCL’s model
* **canonical_url** - ([FHIR Attribute](https://www.hl7.org/fhir/codesystem-definitions.html#CodeSystem.url)) Identifying URL for the source (i.e. the CodeSystem in FHIR) or collection (i.e. the ValueSet in FHIR) 
* **publisher** - ([FHIR Attribute](https://www.hl7.org/fhir/codesystem-definitions.html#CodeSystem.publisher)) The name of the organization or individual that published the resource
* **jurisdiction** - ([FHIR Attribute](https://www.hl7.org/fhir/codesystem-definitions.html#CodeSystem.jurisdiction)) A legal or geographic region in which the resource is intended to be used.
* **purpose** - ([FHIR Attribute](https://www.hl7.org/fhir/codesystem-definitions.html#CodeSystem.purpose)) Explanation of why this resource is needed and why it has been designed as it has.
* **copyright** - ([FHIR Attribute](https://www.hl7.org/fhir/codesystem-definitions.html#CodeSystem.copyright)) A copyright statement relating to the resource and/or its contents. Copyright statements are generally legal restrictions on the use and publishing of the resource.
* **meta** - 
* **identifier** - ([FHIR Attribute](https://www.hl7.org/fhir/codesystem-definitions.html#CodeSystem.identifier)) A formal identifier that is used to identify this code system when it is represented in other formats, or referenced in a specification, model, design or an instance.
* **contact** - ([FHIR Attribute](https://www.hl7.org/fhir/codesystem-definitions.html#CodeSystem.contact)) Contact details to assist a user in finding and communicating with the publisher.
* **content_type** - ([FHIR Attribute](https://www.hl7.org/fhir/codesystem-definitions.html#CodeSystem.content)) The extent of the content of the resource (the concepts and codes it defines) are represented in this resource instance.
* **revision_date** - ([FHIR Attribute](https://www.hl7.org/fhir/codesystem-definitions.html#CodeSystem.date)) The date (and optionally time) when the resource was published. The date must change when the business version changes and it must change if the status code changes. In addition, it should change when the substantive content of the code system changes.
* **logo_url** - URL of logo image for this resource
* **text** - “About” text for resource
* **experimental** - ([FHIR Attribute](https://www.hl7.org/fhir/codesystem-definitions.html#CodeSystem.experimental)) A Boolean value to indicate that this resource is authored for testing purposes (or education/evaluation/marketing) and is not intended to be used for genuine usage.
* **case_sensitive (Source only)** - ([FHIR Attribute](https://www.hl7.org/fhir/codesystem-definitions.html#CodeSystem.caseSensitive)) If code comparison is case sensitive when codes within this resource are compared to each other.
* **collection_reference** - 
* **hierarchy_meaning (Source only)** - ([FHIR Attribute](https://www.hl7.org/fhir/codesystem-definitions.html#CodeSystem.hierarchyMeaning)) The meaning of the hierarchy of concepts as represented in this resource.
* **compositional (Source only)** - ([FHIR Attribute](https://www.hl7.org/fhir/codesystem-definitions.html#CodeSystem.compositional)) The resource defines a compositional (post-coordination) grammar.
* **version_needed (Source only)** - ([FHIR Attribute](https://www.hl7.org/fhir/codesystem-definitions.html#CodeSystem.versionNeeded)) This flag is used to signify that the resource does not commit to concept permanence across versions. If true, a version must be specified when referencing this resource.
* **hierarchy_root_url (Source only)** - For hierarchical sources, the OCL-style URL of the highest level concept allows for viewing of hierarchical concepts within the source
* **immutable (Collection only)** - ([FHIR Attribute](https://www.hl7.org/fhir/valueset-definitions.html#ValueSet.immutable)) If this is set to 'true', then no new versions of the content logical definition can be created. Note: Other metadata might still change.

### Concepts
**Required**
* **resource_type** - “Concept”
* **owner_id** - OCL resource identifier of the OCL user or organization that will own this object
* **source** - OCL resource identifier of the source in which this concept will be defined
* **id** - OCL resource identifier
* **concept_class** - Type of concept, which can vary across and within sources and collections. 

**Optional**
* **owner_type** - “Organization” or “User”; default=”Organization”
* **retired** - default=False
* **external_id** -
* **datatype** - default=”None”
* Initial Name:
   * **name** -
   * **name_locale** (Optional) - default=”en”
   * **name_locale_preferred** (Optional) - default=True
   * **name_type** (Optional) - default=”Fully Specified”
   * **name_external_id** (Optional) -
* Additional names:
   * **name[index]** -
   * **name_locale[index]** (Optional) - default=”en”
   * **name_locale_preferred[index]** (Optional) - default=False
   * **name_type[index]** (Optional) -
   * **name_external_id[index]** (Optional) -
* Initial Description:
   * **description** -
   * **description_locale** (Optional) - default=”en”
   * **description_locale_preferred** (Optional) - default=False
   * **description_type** (Optional) -
   * **description_external_id** (Optional) -
* Additional Descriptions:
   * **description[index]** -
   * **description_locale[index]** (Optional) - default=”en”
   * **description_locale_preferred[index]** (Optional) - default=False
   * **description_type[index]** (Optional) -
   * **description_external_id[index]** (Optional) -
* Custom Attributes: 
   * CSV Syntax:
      * **attr:[custom-attribute-key]** (Optional) - custom attributes using default _string_ format
      * **attr:[custom-attribute-key]:bool** (Optional) - custom attributes using _boolean_ format
      * **attr:[custom-attribute-key]:list** (Optional) - custom attributes using _list_ format
      * **attr:[custom-attribute-key]:float** (Optional) - custom attributes using _float_ format (i.e. decimal numbers like 2.5)
      * **attr:[custom-attribute-key]:int** (Optional) - custom attributes using _int_ format (i.e. numbers without decimals like 4)
      * **attr_value[index], attr_key[index]** (Optional) - custom attributes
   * JSON Syntax:
      * **"extras"**:{"[attr_key]":"attr_value", … }
* Internal Concept Mappings: (Where the concept defined in the row is the `from_concept`)
   * **map_target** - default=”Internal”
   * **map_owner_id** - Automatically set to the concept `owner_id` if omitted
   * **map_owner_type** - Automatically set to the concept `owner_type` if omitted
   * **map_source** - Automatically set to the concept `source` if omitted
   * **map_type[index]** - default=”Same As”
   * to_concept must provide a minimum set of fields to resolve to a `to_concept_url`
      * **map_to_concept_url[index]**
      * **map_to_concept_id[index]**
      * **map_to_concept_name[index]**
      * **map_to_concept_owner_id[index]**
      * **map_to_concept_owner_type[index]** - default=”Organization”
      * **map_to_concept_source[index]**
* External Concept Mappings: (Where the concept defined in the row is the `from_concept`)
   * **extmap_target** (Optional) - default=”External”
   * **extmap_owner_id** (Optional) - Automatically set to the concept `owner_id` if omitted
   * **extmap_owner_type** (Optional) - Automatically set to the concept `owner_type` if omitted
   * **extmap_source** (Optional) - Automatically set to the concept `source` if omitted
   * **extmap_type[index]** (Optional) - default=”Same As”
   * to_concept must provide a minimum set of fields to resolve to a to_concept_url
      * **extmap_to_concept_id[index]**
      * **extmap_to_concept_name[index]** (Optional)
      * **extmap_to_concept_owner_id[index]**
      * **extmap_to_concept_owner_type[index]** (Optional) - default=”Organization”
      * **extmap_to_concept_source[index]**
* **parent_concept_url** - If a hierarchical concept, the OCL-formatted URL for this concept’s parent concept.
### Standalone Mappings (Internal or External)
**Required**
* **resource_type** - “Mapping” or “External Mapping”
* **owner_id** - OCL resource identifier of the OCL user or organization that will own this object
* **source** - OCL resource identifier of the source in which this concept will be defined
* from_concept must provide a minimum set of fields to resolve to a `from_concept_url`
   * **map_from_concept_url** (Optional)
   * **map_from_concept_id** (Optional)
   * **map_from_concept_owner_id** (Optional)
   * **map_from_concept_owner_type** (Optional) - default=”Organization”
   * **map_from_concept_source** (Optional)
* to_concept must provide a minimum set of fields to resolve to a `to_concept_url`
   * **map_to_concept_url** (Optional)
   * **map_to_concept_id** (Optional)
   * **map_to_concept_name** (Optional)
   * **map_to_concept_owner_id** (Optional)
   * **map_to_concept_owner_type** (Optional) - default=”Organization”
   * **map_to_concept_source** (Optional)

**Optional**
* **owner_type** - “Organization” or “User”; default=”Organization”
* **map_type** - default=”Same As”


### References (Add concepts to a collection)
**Required**
* **resource_type** - “Reference”
* **collection_url** - OCL-formatted URL of the collection to which the reference(s) will be added
* **data/expressions** - List of references to be added to the collection, each of which is in the OCL-formatted concept URL. Note that any URL string can be used here, even if the concept is not stored in OCL itself. If the URL cannot be resolved within OCL, then it will appear in the collection’s References tab but not in its Concepts tab in OCL’s web interface.


### Repository Versions (Sources or Collections)
**Required**
* **resource_type** - “Source Version” or “Collection Version”
* **owner_id** - OCL resource identifier of the OCL user or organization that will own this object
* **source** OR **collection** - identifier of the source or collection for the new repository version
* **id** - Identifier for the repository version (e.g. “v1.0”)
* **description** - Description of the repository version


**Optional**
* **owner_type** - “Organization” or “User”; default=”Organization”
* **released** - default=False
* **retired** - default=False
  

## OCL Bulk Import API Reference 
### Bulk Import via API
API calls for bulk importing can be found in OCL’s [Swagger page](https://api.openconceptlab.org/swagger/) under the `importers` section. Inline importing can be performed using `POST /importers/bulk-import-inline/`, while parallel importing can be performed using `POST /importers/bulk-import-parallel-inline/`.

Using parallel importing allows the specification of a number of parallel threads, which speed up the import process but consume more of OCL’s resources. By default, 5 workers are used for a bulk import, but this number can be anywhere between 2 and 10.

### Bulk Import Queues
Submitting to the Standard Queue
Post a JSON bulk import file for asynchronous processing in the standard queue. The standard queue has multiple workers processing in parallel, and therefore bulk imports may not be processed in the order that they are submitted.

```
POST /manage/bulkimport/:queue/
```

* POST Request Parameters:
   * **test_mode** - default=`false`; set to `true` to only run a test import \<NOT CURRENTLY SUPPORTED!\>
   * **update_if_exists** - default=`true`; set to `false` to skip updating resources that already exist

Submitting to a User Assigned Queue
Adds a JSON bulk import file for asynchronous processing in a user assigned queue. User assigned queues process bulk import files using only one worker, therefore guaranteeing that they will be processed in the order in which they are submitted.


```
POST /manage/bulkimport/:queue/
```

* POST Request Parameters:
   * **test_mode** - default=`false`; set to `true` to only run a test import \<NOT CURRENTLY SUPPORTED!\>
   * **update_if_exists** - default=`true`; set to `false` to skip updating resources that already exist


Get a list of active and recent bulk imports for a user in the standard and user assigned queues
To monitor the list of bulk imports by your account, use a GET request. Specify a particular queue ID to look at the status of that queue.


```
GET /manage/bulkimport/
GET /manage/bulkimport/:queue/
```
*  GET Request Parameters:
   *  Root user only:
      *  **username** - optionally filter by username; for root, bulk imports for all users are returned by default

NOTE: Returns an empty list `[]` if no recent or active bulk imports are queued


Get the status or results of a previously submitted bulk import
To view the final outcome of a previous bulk import, use a GET request to specify the task ID, optionally specifying a result format


```
GET /manage/bulkimport/?task=:taskid[&result=:format]
```
* GET Request Parameters:
   * **task** (Required for GET request) - Task ID of a previously submitted bulk import request
   * **result** (Optional) - default="summary"; format of the results to be returned. Options are:
      * **summary** -- one line of plain text (see `OclImportResults.get_detailed_summary()`)
      * **report** -- longer report of plain text (see `OclImportResults.display_report()`)
      * **json** -- full results object serialized to JSON (see `OclImportResults.to_json()`)
_”Summary” example_
```
Processed 348 of 348 -- 346 NEW (200:39, 201:307); 1 UPDATE (200:1); 1 DELETE (200:1)
```


_”Report” example_


```
REPORT OF IMPORT RESULTS:
/orgs/DATIM-MOH-BW-FY19/collections/HTS-TST-N-MOH-HllvX50cXC0/:
NEW 200:
[{"message": "Added the latest versions of concept to the collection. Future updates will not be added automatically.",
"added": true, "expression":
...
```


_”JSON” example_




```
{
        "count": 348,
        "elapsed_seconds": 94.10947012901306,
        "total_lines": 348,
        "num_skipped": 0,
        "results": {
            "/orgs/DATIM-MOH-BW-FY19/collections/HTS-TST-N-MOH-HllvX50cXC0/": {
                "NEW": {
                    "200": [
                        {
                            "obj_type": "Reference",
                            "text": "{\"data\": {\"expressions\": [\"/orgs/DATIM-MOH-BW-FY19/sources/DATIM-Alignment-Indicators/mappings/MAP-DATIM-HAS-OPTION-HTS_TST_N_MOH-HllvX50cXC0/\", \"/orgs/PEPFAR/sources/DATIM-MOH-FY19/concepts/HTS_TST_N_MOH/\", \"/orgs/PEPFAR/sources/DATIM-MOH-FY19/concepts/HllvX50cXC0/\", \"/orgs/DATIM-MOH-BW-FY19/sources/
```


* Notes:
   * Bulk imports are processed on behalf of the requesting user. Each line in an import file is processed separately and the user must have appropriate permissions for each line to be processed successfully.
   * The payload for POSTs should contain resources to be created (each in a new line). Note that you are able to mix multiple resource types.




### Example Request - POST
```
POST /manage/bulkimport/
```
```
{"type": "Source", "id": "JonTestSource", "short_code": "JonTestSource", "name": "Jon test source", "full_name": "Jon test source", "owner": "paynejd", "owner_type": "User", "description": "", "source_type": "Indicator Registry", "public_access": "View", "default_locale": "en", "supported_locales": "en", "custom_validation_schema": "None"}
{"retired": false, "datatype": "None", "type": "Concept", "concept_class": "Disaggregate", "source": "JonTestSource", "extras": null, "descriptions": null, "owner": "paynejd", "owner_type": "User", "external_id": "HSpL3hSBx6F", "id": "HSpL3hSBx6F", "names": [{"locale": "en", "locale_preferred": true, "external_id": null, "name": "50+, Male, Negative", "name_type": "Fully Specified"}]}
```


###  Response - POST
* In response you will receive a JSON with `status` and `task` attributes. The `task` attribute contains a UUID of the asynchronous task, which you can use to further query for the status of the task or, when processing is complete, the results of the import. For example:
```
{
        "status": "PENDING",
        "task": "2344a457-cfdf-4985-ae0f-b2797d33a1a2"
}
```


### Example Request - GET
```
GET /manage/bulkimport/?task=2344a457-cfdf-4985-ae0f-b2797d33a1a2&result=json
```


### Response - GET
* 200: Import finished processing (though there may have been an error)
        * If the import is complete, the response will contain the results of the import in the requested format (summary, report, or JSON)
* 200 (in the future this will be 202):
        ***PENDING**: If the bulk import is queued and processing has not yet begun, the response will have a status of `PENDING` (the same as above)
        * **STARTED**: If the bulk import is being processed and is not yet complete, the response will have a status of `STARTED`
* 404: The import task ID was not found for the requesting user




### Cancelling a Bulk Import
Ongoing bulk imports can be cancelled before completion, although it will not undo what parts of the import have already been done.
```
DELETE /importers/bulk-import/?task_id=2344a457-cfdf-4985-ae0f-b2797d33a1a2&signal=SIGKILL
```

Parameters:
* task_id (required) - ID of the task to be deleted
* signal - default=SIGKILL ; Other signals available [here](https://man7.org/linux/man-pages/man7/signal.7.html)


## Bulk Import examples using curl
OCL provides two ways of importing Parallel (new and recommended) and sequential (legacy) modes.
The parallel import can take import content input in three ways:
* JSON/CSV file:
```
curl -X 'POST' \
  'http://localhost:8000/importers/bulk-import-parallel-inline/custom-queue/?update_if_exists=true' \
  -H 'accept: application/json' \
  -H 'Authorization: Token XXXXXXXXXXXXX' \
  -H 'Content-Type: multipart/form-data' \
  -F 'file=@sample_ocldev.json;type=application/json' \
  -F 'parallel=2'
```
* HTTPs URL for JSON/CSV File:
```
curl -X 'POST' \
  'http://localhost:8000/importers/bulk-import-parallel-inline/custom-queue/?update_if_exists=true' \
  -H 'accept: application/json' \
  -H 'Authorization: Token XXXXXXXXXXXXX' \
  -H 'Content-Type: multipart/form-data' \
  -F 'file_url=https://my-file.json' \
  -F 'parallel=2'
```
* JSON content:
```
curl --location --request POST 'http://127.0.0.1:8000/importers/bulk-import-parallel-inline/custom-queue/?update_if_exists=true' \
--header 'Accept: */*' \
--header 'Content-Type: multipart/form-data' \
--header 'Authorization: Token XXXXXXXXXXXXX' \
--form 'parallel=2' \
--form 'data={"foo": "bar"}' \
```


## Bulk Import via the OCL TermBrowser
When logged into an OCL account, the Bulk Import interface in the TermBrowser is available in the App menu at the top right. This interface allows the use of the following bulk import features:
* Content Load
   * Upload JSON or CSV file for loading
   * Paste in JSON text for loading
   * Paste in a URL for a JSON or CSV file for loading
   * Specify a queue name for the import
   * “Update Existing” option to update a resource that is already available in OCL
   * “Parallel” option to speed up the process by running multiple lines of the same resource type synchronously
   * “Hierarchy” option to load content that has a hierarchical structure
* Import Queue Monitoring
   * View active and past import queues (for up to 3 days), with filters to find imports
   * View and download a report for completed imports, including runtime, results, etc.

