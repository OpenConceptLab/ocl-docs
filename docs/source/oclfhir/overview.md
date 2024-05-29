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

## FHIR NPM package import
This is work in progress. ETA is June 2024. 

The feature provides a way to import content from the FHIR NPM package registry at https://registry.fhir.org/

It enhances the bulk import tool in OCL to support the FHIR resouurces.

The integral part of the work is to make large imports memory efficient by implementing streamed file read as opposed to loading the entire file in memory.

The implementation so far includes:
1. Endpoint for fetching a package from NPM registry and importing for authenticated users under a selected namespace (org or user) (API only) (must for MVP)
2. Overwrite existing resources if id and version matches in the same namespace (must for MVP)
3. Create new versions of resources if only id matches (must for MVP)
4. Import all currently supported resources i.e. CodeSystem, ValueSet, ConceptMap (must for MVP)

Remaining work:
1. Endpoint for uploading and importing a package for authenticated users under a selected namespace (org or user) (API only) (must for MVP)
2. UI for uploading or fetching a package and importing (UI only) (must for MVP)
3. Fetching dependent packages (must for MVP)
4. A summary of imported packages and created/updated resources (nice to have for MVP)
5. A dialog presented before importing for the user to confirm which lists all package dependencies that needs to be imported (nice to have for MVP)
