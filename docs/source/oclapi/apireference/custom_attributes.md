# Custom (Extra) Attributes

## Introduction

OCL provides a set of API endpoints to create and manage custom attributes, called `extras`, for certain resources. The resources that support `extras` are:
* Organizations
* Users
* Sources and Source Versions
* Collections and Collection Versions
* Concepts and Concept Versions
* Mappings and Mapping Versions

`extras` use a common API syntax, so the documentation here applies to all resources that support custom attributes.

Custom attributes are designed to allow content administrators to store data that is outside of a resource’s core attributes. Each custom attribute consists of an attribute name and a value. The attribute name must be a string, and the value may be of any JSON datatype. A resource may have any number of custom attributes.

For an example of custom attributes on an owner, a repository, and resources, see [DemoOrg](https://app.openconceptlab.org/#/orgs/DemoOrg/) on OCL Online. This organization contains multiple examples of custom attributes of varying data types. Some examples are shown below:

|           **Resource**          | **Example Attribute Name** |                                                        **Example Value**                                                        |
|:-------------------------------:|:--------------------------:|:-------------------------------------------------------------------------------------------------------------------------------:|
|      Organization ([DemoOrg](https://app.openconceptlab.org/#/orgs/DemoOrg/))     |          "Ex_Num"          |                                                                6                                                                |
|                                 |        "extra_names"       | [  {   "name": "Demotastic Name",   "short_name": "demo"  },   {   "name": "Out-of-Date Demo Name",   "short_name": "old"  }  ] |
|      Source ([MyDemoSource](https://app.openconceptlab.org/#/orgs/DemoOrg/sources/MyDemoSource/))      |          “ex_name”         |                                                          "Source Name"                                                          |
| Concept ([“Active Demo Concept”](https://app.openconceptlab.org/#/orgs/DemoOrg/sources/MyDemoSource/concepts/Act/)) |       “Bool_Example”       |                                                              FALSE                                                              |
|                                 |       “JSON_example”       |                           {"Word" : "Chair", "Definition" : "A thing you sit on", "WordType", "Noun"}                           |
|                                 |       “List_example”       |                                                         ["1", "6", "7"]                                                         |

## Interacting with Custom Attributes using the OCL API
**Get a single Custom Attribute**
```
GET /orgs/:org/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]extras/:field_name/
 
GET /users/:user/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]extras/:field_name/
 
GET /user/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]extras/:field_name/
```

**List all Custom Attributes**
```
GET /orgs/:org/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]extras/
 
GET /users/:user/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]extras/

GET /user/sources/:source/[:sourceVersion/]concepts/:concept/[:conceptVersion/]extras/
```

**Creating and Updating Custom Attribute**
```
PUT /orgs/:org/sources/:source/concepts/:concept/extras/:field_name/

PUT /users/:user/sources/:source/concepts/:concept/extras/:field_name/

PUT /user/sources/:source/concepts/:concept/extras/:field_name/

(was POST)
```

* Note: Custom attributes are part of a concept, therefore creating or updating a custom attribute will generate a new version of the underlying concept

**Deleting Custom Attribute**
```
DELETE /orgs/:org/sources/:source/concepts/:concept/extras/:field_name/

DELETE /users/:user/sources/:source/concepts/:concept/extras/:field_name/

DELETE /user/sources/:source/concepts/:concept/extras/:field_name/

(was POST)
```

* Note: Custom attributes are part of a concept, therefore creating or updating a custom attribute will generate a new version of the underlying concept

## Filtering using Custom Attribute

OCL exposes a method for filtering resources based on their custom attributes (ie `extras` field). Many resources in OCL support custom attributes, including: orgs, sources, collections, source/collection versions, concepts, and mappings.


Below is an example of filtering a list of Sources to find a particular Source.

Consider the following Source:
```
{
  "short_code": "DATIM-Alignment-Indicators",
  "name": "DATIM MOH Burundi Alignment Indicators",
  "url": "/orgs/DATIM-MOH-BI-FY19/sources/DATIM-Alignment-Indicators/",
  "owner": "DATIM-MOH-BI-FY19",
  "owner_type": "Organization",
  "owner_url": "/orgs/DATIM-MOH-BI-FY19/",
  "version": "HEAD",
  "created_at": "2020-09-11T05:04:52.310583Z",
  "id": "DATIM-Alignment-Indicators",
  "source_type": "Dictionary",
  "updated_at": "2020-09-14T12:00:16.660844Z",
  "canonical_url": null,
  "extras": {
    "datim_moh_object": true,
    "datim_moh_period": "FY19",
    "datim_moh_country_code": "BI"
  }
}
```

#### The following will work to get the above source in search results:
* `GET /sources/?extras.datim_moh_period=FY19`   # Exact match
* `GET /sources/?extras.datim_moh_period=FY*`     # Starts With
* `GET /sources/?extras.datim_moh_period=*FY*`   # Contains
* `GET /sources/?extras.datim_moh_period=*fy*`   # Caseinsensitive by default
* `GET /sources/?extras.datim_moh_period=bar FY19`   # Multi Value OR (has to be separated by space, ES way)
* `GET /sources/?extras.datim_moh_period=XY* FY19`   # Multi Value OR (has to be separated by space, ES way)
* `GET /sources/?extras.datim_moh_period=XY* FY*`   # Multi Value OR (has to be separated by space, ES way)
* `GET /sources/?extras.datim_moh_period=FY*&extras.datim_moh_object=true`   # Multi Field AND
* `GET /sources/?q=datim&extras.datim_moh_period=FY*`   # with `q` param

#### The following will work to exclude this source:
* `GET /sources/?extras.datim_moh_period=!*FY*`   # not(!) contains clause
* `GET /sources/?extras.datim_moh_period=!FY*`   # not(!) contains clause
* `GET /sources/?extras.datim_moh_period=!FY19`   # not(!) contains clause



#### Attribute Exists:
* `/sources/?extras.exists=datim_moh_country_code` # single attr exists
* `/sources/?extras.exists=datim_moh_country_code,datim_moh_period` # multiple attrs exists
* `/sources/?extras.exists=datim_moh_country_code,datim_moh_period&extras.datim_moh_object=false` # multiple attrs exists with extra attr value search

#### Space in attribute name:
* `/sources/?extras.foo bar=foobar`
* `/sources/?extras.foo+bar=foobar`
* `/sources/?extras.exists=foo bar`
* `/sources/?extras.exists=foo bar,datim_moh_country_code`

#### Space in attribute value/search criteria
* `/sources/?extras.foo bar=foo bar foo` # will find extras["foo bar"] = "foo" or "foo bar" and similar

## Creating and Formatting Custom Attributes in TermBrowser
When creating or editing any resources in the OCL TermBrowser, there will be a section called “Custom Attributes”, where any number of custom attributes can be created, each with their respective value. 

A data type for each value may be inferred based on what is entered in the field. In general, OCL attempts to keep the value of what is typed in the field. If a JSON or list value is entered (e.g. any value surrounded by `{}` or `[]`), then that type will be inferred. If a numeric value, with or without decimals, then OCL will keep that data type. If the value is either `true` or `false`, then the boolean type will be used. Otherwise, the value will be stored as a string.

## Creating and Formatting Custom Attributes using CSV Import or ocldev tools
When importing resources using OCL’s CSV Import module or importing a CSV via OCL’s API, columns that are intended to be custom attributes can be designated using the prefix “attr:”. For example, the custom attribute “Ex_Num” can be assigned using a “attr:Ex_Num” column in the CSV file.

The data type of the field can be assigned using a suffix following the attribute name. For example, the custom attribute “Ex_Num” can be a numeric (specifically integer) data type using the CSV header “attr:Ex_Num:int”. The following data types are supported when using this method:
* Boolean as 'bool' - e.g. `attr:Ex_Num:bool`
* String as 'str' - e.g. `attr:Ex_Num:str`
* Integer as 'int' - e.g. `attr:Ex_Num:int`
* Float as 'float' - e.g. `attr:Ex_Num:float`
* List as 'list’ - e.g. `attr:Ex_Num:list`
* JSON as 'json' - e.g. `attr:Ex_Num:json`

In short, a custom attribute in the CSV header is designated using the following syntax: `attr:<name>:<type>`. For more examples of this, refer to [OCL’s CSV example file](https://drive.google.com/file/d/1lmK0qDlDJU4Mth__gCeSkPkiON0c0I02/view).

