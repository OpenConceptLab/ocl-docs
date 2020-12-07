# Custom Attribute Filters

OCL exposes a method for filtering resources based on their custom attributes (ie `extras` field). Many resources in OCL support custom attributes, including: orgs, sources, collections, source/collection versions, concepts, and mappings.

Work happening in these tickets: #39, #448, #471

## New OCLAPI v2 Elastic Search (ES) implementation
The query structure is a bit different in ES. Following are the examples:
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

#### Following will work to get the above source in search results
* `GET /sources/?extras.datim_moh_period=FY19`   # Exact match
* `GET /sources/?extras.datim_moh_period=FY*`     # Starts With
* `GET /sources/?extras.datim_moh_period=*FY*`   # Contains
* `GET /sources/?extras.datim_moh_period=*fy*`   # Caseinsensitive by default
* `GET /sources/?extras.datim_moh_period=bar FY19`   # Multi Value OR (has to be separated by space, ES way)
* `GET /sources/?extras.datim_moh_period=XY* FY19`   # Multi Value OR (has to be separated by space, ES way)
* `GET /sources/?extras.datim_moh_period=XY* FY*`   # Multi Value OR (has to be separated by space, ES way)
* `GET /sources/?extras.datim_moh_period=FY*&extras.datim_moh_object=true`   # Multi Field AND
* `GET /sources/?q=datim&extras.datim_moh_period=FY*`   # with `q` param

#### Following will work to exclude this source:
* `GET /sources/?extras.datim_moh_period=!*FY*`   # not(!) contains clause
* `GET /sources/?extras.datim_moh_period=!FY*`   # not(!) contains clause
* `GET /sources/?extras.datim_moh_period=!FY19`   # not(!) contains clause

This was supported out of the box by ES, by just querying for extras.
Let me know if this is good enough, or if we have to add more custom behaviors.


### Attribute Exists:
* `/sources/?extras.exists=datim_moh_country_code` # single attr exists
* `/sources/?extras.exists=datim_moh_country_code,datim_moh_period` # multiple attrs exists
* `/sources/?extras.exists=datim_moh_country_code,datim_moh_period&extras.datim_moh_object=false` # multiple attrs exists with extra attr value search

### Space in attribute name:
* `/sources/?extras.foo bar=foobar`
* `/sources/?extras.foo+bar=foobar`
* `/sources/?extras.exists=foo bar`
* `/sources/?extras.exists=foo bar,datim_moh_country_code`

### Space in attribute value/search criteria
* `/sources/?extras.foo bar=foo bar foo` # will find extras["foo bar"] = "foo" or "foo bar" and similar


## Original OCLAPI v1 Solr implementation
An example of a supported search by field name:
.../concepts/?extras__field_name__nested-field-name=value,value2
(separating values with ',' applies OR to values)
'__' (two underscores) separate nested fields

Of course, since it's indexed in solr you can use wildcard search for values, e.g. value* (starts with) or *value* (contains), etc.

As a side effect it's also possible to pass any solr search query using 'q' parameter e.g.:
?q=extras_field_20name:*value* OR extras_field2:value*

I don't think we made a commitment in our API to support such queries and it wouldn't be correct since it exposes the underlying implementation and how we store data in solr, which does not necessarily match our API model.

Note that any non-alphanumeric characters in field names are stored in solr with hex representation proceeded by '_'.
Solr search query fields are also separated with just a single '_' (underscore), whereas for an extras query param we use'__'(two underscores). I couldn't use '__' in solr, because '__' is reserved by django queryset and cannot be used in field names. Yet, I wanted to provide an easy way to query fields with a single underscore in name using the field name search.

Made the search to ignore the case, added negation and support for collections.

https://api.qa.openconceptlab.org/orgs/CIEL/sources/CIEL/concepts/?extras__indicatorGroups__name=hts_tst

https://api.qa.openconceptlab.org/orgs/CIEL/sources/CIEL/concepts/?extras__indicatorGroups__name!=hts_tst

   - annualized is true/false (boolean)
   - dimensionItemType == "INDICATOR"
   - dimensionItemType != "INDICATOR"
   - denominatorDescription contains "vctmod" (for both case
   sensitive/insensitive)
   - is (or is not) a member of the "HTS_TST" indicatorGroup

https://api.staging.openconceptlab.org/orgs/PEPFAR-Test7/sources/MER/concepts/RIpjFLFhfsm/
"extras": {
        "annualized": false,
        "dimensionItemType": "INDICATOR",
        "denominatorDescription": "Total tests in VCTMod",
        "indicatorGroups": [
            {
                "name": "HTS_TST",
                "id": "AgMcHnlM9vm"
            }
        ]
}

Response from Rafal:

Here are the queries you requested:
* https://api.qa.openconceptlab.org/orgs/CIEL/sources/CIEL/concepts/?extras__annualized=false
* https://api.qa.openconceptlab.org/orgs/CIEL/sources/CIEL/concepts/?extras__dimensionItemType=INDICATOR
* https://api.qa.openconceptlab.org/orgs/CIEL/sources/CIEL/concepts/?extras__denominatorDescription=*VCTMod*

I didn't add support for collections so indicatorGroups query is not possible at the moment. I guess I could implement it as solr supports multivalued fields.

All queries are case sensitive at the moment. We can make them case insensitive or do you want to switch between modes? Switching modes would basically mean we need to index everything twice, if we want the search to be efficient.

The "not equal" query is also not yet supported unless you pass solr raw query with the q parameter, but as I wrote on the issue we shouldn't be providing support for solr raw queries. I mean it's fine to have them available, but without any guarantee they will continue to work...

I would imagine the not equal query to be constructed as ?extras__dimensionItemType!=INDICATOR (with the '!' after field name).
