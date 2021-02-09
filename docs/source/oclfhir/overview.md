# Overview

OpenConceptLab FHIR terminology service allows OCL users to manage resources in FHIR format. The OCL FHIR terminology service is implemented based on IHE's (Integrating the Healthcare Enterprise) mobile sharing ValueSets, CodeSystems, and ConceptMaps (mSVCM) profile. Based on mSVCM profile, OCL FHIR terminology service provides ability to retrieve ValueSets, CodeSystems and ConceptMaps along with ability to perform FHIR operations on this resources.

## versions
FHIR        v4.0.1

HAPI FHIR   v5.0.0

Java        OpenJDK 14.0.2

## Building

In order to build run: `docker-compose build`

## Running

Before running make sure that oclapi2 is running. Check the oclapi2 network by running: `docker network list`

If it is different than `oclapi2_default`, please prepend network variable `OCLAPI_NETWORK=oclapi` to the following run commands.

In order to startup the FHIR service in development mode run: `SERVER_PORT=9000 docker-compose up`

To run in production run: `SERVER_PORT=9000 docker-compose -f docker-compose.yml up`

## Testing
Import `OCL FHIR Postman Collection <https://www.postman.com/collections/e7ff349688c4c10f474a>`_ into yourr postman client to test the OCL FHIR endpoints 
