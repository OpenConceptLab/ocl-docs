# (Beta) FHIR Connectors in OCL TermBrowser
Updated 2021-11-10

**Owner:** [Joe](https://github.com/jamlung-ri/)

**Maintainer:** [Joe](https://github.com/jamlung-ri/)

A system administrator may configure the OCL TermBrowser to connect to one or more OCL Terminology Servers or SVCM-compatible FHIR servers, such as HAPI FHIR. This allows authorized users to "switch servers" to redirect the TermBrowser to another server to search and browse terminology resources. The TermBrowser implements a basic FHIR Connector that can be adapted to as needed to optimize searching and browsing of FHIR Code Systems, Value Sets and Concept Maps.

### FHIR Server Configuration
Directing the TermBrowser to a FHIR terminology server currently requires initial configuration by pointing the TermBrowser to the FHIR server of interest. For example, connecting to HAPI FHIR's server requires the following server URL: http://hapi.fhir.org/baseR4 with this capability statement: http://hapi.fhir.org/baseR4/metadata 

### FHIR Connector Queries used by OCL TermBrowser
The following queries are used by the FHIR Connector. Note that the base FHIR Connector was developed against HAPI FHIR, so some features such as full text searching are not shown here.


#### CodeSystem Queries

- **Global List:** https://fhir.staging.openconceptlab.org/fhir/CodeSystem/?_total=accurate&page=1&_getpagesoffset=0&_count=10&_sort=_id 
- **Search by Status:** https://fhir.staging.openconceptlab.org/fhir/CodeSystem/?_total=accurate&page=1&status=active&_sort=_id 
- **Search by content-mode:** https://fhir.staging.openconceptlab.org/fhir/CodeSystem/?_total=accurate&page=1&content-mode=foo&_sort=_id 
- **Search by Publisher:** https://fhir.staging.openconceptlab.org/fhir/CodeSystem/?_total=accurate&page=1&publisher=foo&_sort=_id 
- **Expand a row for a CodeSystem:** https://fhir.staging.openconceptlab.org/orgs/Malawi-Demo/CodeSystem/Baobab/version/ 
- **View individual CodeSystem and its codes:** https://fhir.staging.openconceptlab.org/orgs/Malawi-Demo/CodeSystem/Baobab?page=1 
- **View all CodeSystems from an organization:** https://fhir.staging.openconceptlab.org/orgs/Malawi-Demo/CodeSystem/?_total=accurate&page=1&_getpagesoffset=0&_count=10&_sort=_id 

#### ValueSet Queries

- **Global List:** https://fhir.staging.openconceptlab.org/fhir/ValueSet/?_total=accurate&page=1&_getpagesoffset=0&_count=10&_sort=_id 
- **Search by Status:** https://fhir.staging.openconceptlab.org/fhir/ValueSet/?_total=accurate&page=1&status=active&_sort=_id
- **Search by content-mode:** https://fhir.staging.openconceptlab.org/fhir/ValueSet/?_total=accurate&page=1&content-mode=foo&_sort=_id
- **Search by Publisher:** https://fhir.staging.openconceptlab.org/fhir/ValueSet/?_total=accurate&page=1&publisher=foobar&_sort=_id 
- **Expand a row for a ValueSet:** https://fhir.staging.openconceptlab.org/orgs/MSFOCP/ValueSet/BA-test/version/ 
- **View individual ValueSet and its codes:** https://fhir.staging.openconceptlab.org/orgs/MSFOCP/ValueSet/BA-test?page=1 
- **View all ValueSets from an organization:** https://fhir.staging.openconceptlab.org/orgs/MSFOCP/ValueSet/?_total=accurate&page=1&_getpagesoffset=0&_count=10&_sort=_id 

#### ConceptMap Queries

- **Global List:** https://fhir.staging.openconceptlab.org/fhir/ConceptMap/?_total=accurate&page=1&_getpagesoffset=0&_count=10&_sort=_id 
- **Search by Status:** https://fhir.staging.openconceptlab.org/fhir/ConceptMap/?_total=accurate&page=1&status=active&_sort=_id 
- **Search by content-mode:** https://fhir.staging.openconceptlab.org/fhir/ConceptMap/?_total=accurate&page=1&content-mode=foobar&_sort=_id 
- **Search by Publisher:** https://fhir.staging.openconceptlab.org/fhir/ConceptMap/?_total=accurate&page=1&publisher=foo&_sort=_id 
- **Expand a row for a ConceptMap:** https://fhir.staging.openconceptlab.org/orgs/Malawi-Demo/ConceptMap/Baobab/version/ 
- **View individual ConceptMap and its groups:** https://fhir.staging.openconceptlab.org/orgs/Malawi-Demo/ConceptMap/Baobab?page=1 
- **View all ConceptMaps from an organization:** https://fhir.staging.openconceptlab.org/orgs/PEPFAR/ConceptMap/?_total=accurate&page=1&_getpagesoffset=0&_count=10&_sort=_id 

####
Filters currently supported by the FHIR Connector:
 * Status
 * content-mode
 * Publisher
