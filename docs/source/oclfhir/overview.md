# OCL FHIR Core (beta)
OCL provides native support for a subset of the FHIR R4B specification. OCL FHIR Core (beta) implementation supports the FHIR `CodeSystem`, `ValueSet` and `ConceptMap` terminology resources and their operations (`$validate-code`, `$lookup`, `$expand`, and `$translate`) in compliance with the IHE Sharing Valuesets and Concept Maps (SVCM) Profile. Users may interchangeably interact with terminology resources loaded into OCL using the OCL FHIR Core API or the OCL API.

## FHIR Validator
The [FHIR Validator](https://github.com/hapifhir/org.hl7.fhir.validator-wrapper) is deployed at: https://fhir-validator.qa.openconceptlab.org. Requests may use `GET` or `POST`. The FHIR Validator is used by OCL to transform XML FHIR requests to JSON, which is natively understood by the OCL FHIR Core.

Example requests:
```
curl --location --request GET 'https://fhir.qa.openconceptlab.org/fhir/CodeSystem' --header 'Accept: application/xml'
curl --location --request GET 'https://fhir.qa.openconceptlab.org/fhir/CodeSystem' --header 'Accept: application/fhir+xml'
curl --location --request GET 'https://fhir.qa.openconceptlab.org/fhir/ValueSet' --header 'Accept: application/xml'
```
