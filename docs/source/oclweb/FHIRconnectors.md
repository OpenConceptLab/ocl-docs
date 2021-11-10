# (Beta) FHIR Connectors in OCL TermBrowser
Updated 2021-11-10

**Owner:** [Joe](https://github.com/jamlung-ri/)

**Maintainer:** [Joe](https://github.com/jamlung-ri/)

One feature of the OCL TermBrowser that is still being tested is the connection of the user interface to FHIR terminology servers. These include FHIR/SVCM compatible servers that contain terminology resources i.e. CodeSystem, ValueSet, and ConceptMap resources. Currently, this allows authorized users in the OCL TermBrowser to "switch servers" to redirect the TermBrowser to another server, which visualizes the resources that are made available via a FHIR API. 

Note that OCL plans to integrate this feature more into its TermBrowser so that "switching servers" is not necessary to visualize and use these resources.


### FHIR Server Configuration
Directing the TermBrowser to a FHIR terminology server currently requires initial configuration by pointing the TermBrowser to the FHIR server of interest. For example, connecting to HAPI FHIR's server requires the following server URL: http://hapi.fhir.org/baseR4 with this capability statement: http://hapi.fhir.org/baseR4/metadata 

### FHIR Connector Queries used by OCL TermBrowser

These are example queries that the TermBrowser uses to retrieve OCL's FHIR resource representations. OCL currently does not support full text search via FHIR, although it does support filtering by some attributes.

Filters currently supported by OCL FHIR:
* Status
* content-mode
* Publisher

**CodeSystem:**
- Global List: https://fhir.staging.openconceptlab.org/fhir/CodeSystem/?_total=accurate&page=1&_getpagesoffset=0&_count=10&_sort=_id 
- Search by Status: https://fhir.staging.openconceptlab.org/fhir/CodeSystem/?_total=accurate&page=1&status=active&_sort=_id 
- Search by content-mode: https://fhir.staging.openconceptlab.org/fhir/CodeSystem/?_total=accurate&page=1&content-mode=foo&_sort=_id 
- Search by Publisher: https://fhir.staging.openconceptlab.org/fhir/CodeSystem/?_total=accurate&page=1&publisher=foo&_sort=_id 
- Expand a row for a CodeSystem: https://fhir.staging.openconceptlab.org/orgs/Malawi-Demo/CodeSystem/Baobab/version/ 
- View individual CodeSystem and its codes: https://fhir.staging.openconceptlab.org/orgs/Malawi-Demo/CodeSystem/Baobab?page=1 
- View all CodeSystems from an organization: https://fhir.staging.openconceptlab.org/orgs/Malawi-Demo/CodeSystem/?_total=accurate&page=1&_getpagesoffset=0&_count=10&_sort=_id 

**ValueSet:**
- Global List: https://fhir.staging.openconceptlab.org/fhir/ValueSet/?_total=accurate&page=1&_getpagesoffset=0&_count=10&_sort=_id 
- Search by Status: https://fhir.staging.openconceptlab.org/fhir/ValueSet/?_total=accurate&page=1&status=active&_sort=_id
- Search by content-mode: https://fhir.staging.openconceptlab.org/fhir/ValueSet/?_total=accurate&page=1&content-mode=foo&_sort=_id
- Search by Publisher: https://fhir.staging.openconceptlab.org/fhir/ValueSet/?_total=accurate&page=1&publisher=foobar&_sort=_id 
- Expand a row for a ValueSet: https://fhir.staging.openconceptlab.org/orgs/MSFOCP/ValueSet/BA-test/version/ 
- View individual ValueSet and its codes: https://fhir.staging.openconceptlab.org/orgs/MSFOCP/ValueSet/BA-test?page=1 
- View all ValueSets from an organization: https://fhir.staging.openconceptlab.org/orgs/MSFOCP/ValueSet/?_total=accurate&page=1&_getpagesoffset=0&_count=10&_sort=_id 

**ConceptMap:**
- Global List: https://fhir.staging.openconceptlab.org/fhir/ConceptMap/?_total=accurate&page=1&_getpagesoffset=0&_count=10&_sort=_id 
- Search by Status: https://fhir.staging.openconceptlab.org/fhir/ConceptMap/?_total=accurate&page=1&status=active&_sort=_id 
- Search by content-mode: https://fhir.staging.openconceptlab.org/fhir/ConceptMap/?_total=accurate&page=1&content-mode=foobar&_sort=_id 
- Search by Publisher: https://fhir.staging.openconceptlab.org/fhir/ConceptMap/?_total=accurate&page=1&publisher=foo&_sort=_id 
- Expand a row for a ConceptMap: https://fhir.staging.openconceptlab.org/orgs/Malawi-Demo/ConceptMap/Baobab/version/ 
- View individual ConceptMap and its groups: https://fhir.staging.openconceptlab.org/orgs/Malawi-Demo/ConceptMap/Baobab?page=1 
- View all ConceptMaps from an organization: https://fhir.staging.openconceptlab.org/orgs/PEPFAR/ConceptMap/?_total=accurate&page=1&_getpagesoffset=0&_count=10&_sort=_id 



