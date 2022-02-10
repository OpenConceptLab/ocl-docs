# OCL Technical Roadmap: June-December 2021

## Overview
### What is the OCL Open-source Technical Roadmap?
The **Open Concept Lab Technical Roadmap** is a set of goals and objectives for each of our community-supported open-source tools that help us meet the needs of our community of users. The OCL Technical Roadmap follows this general process:

- We will follow an open technical roadmap development process by soliciting community input through several channels (meetings, tickets, slack channels) and publishing a draft publicly for final review and prioritization.
- We will begin formal product releases for version 2 releases of each OCL tool, expected in late 2021. Product releases will be used to structure feature prioritization and we are seeking volunteer “Release Managers” for each tool.
- Timelines and priorities are dependent on resources and client needs and will change accordingly.

## OCL Toolkit Overview
The OCL community develops and supports these tools as open-source software that are free to download:
- **OCL Terminology Service**: Core REST API, FHIR-enabled terminology service
- **OCL TermBrowser**: Customizable tool for searching, visualizing, and managing content in OCL
- **OpenMRS Dictionary Manager**: Specialized tool for managing clinical concept dictionaries

**OCL Online**, hosted at [openconceptlab.org](https://openconceptlab.org/), is where you can use OCL’s open-source tools in the cloud along with the global community and is pre-loaded with key terminologies.

## Roadmap
#### _High-level direction setting markers that represent the OCL Community’s priorities and that inform the work that we do._

### OCL Terminology Service & Infrastructure
1. Enhanced FHIR v4.0.1 and SVCM support, including compatibility between OCL and FHIR transactions in the core OCL data model
1. Advanced OCL core capabilities, including dynamic collections, resource diffs, bulk import enhancements
1. Implement requirements from OpenMRS community to support PIH and OHRI implementations (e.g. concept modifications)
1. Scalable infrastructure for OCL Online and enhanced standalone OCL support
1. Test advanced FHIR functionality in an HL7 Connectathon

### OCL TermBrowser
1. Customized organization homes implemented using the OCL TermBrowser, starting with the PEPFAR Metadata Sharing Platform (MSP) v2, CIEL, and reusable templates for common presentation needs
1. Integrated OCL/FHIR experience — browsing an OCL server provides access to all OCL features, including FHIR
1. Intuitive browsing of FHIR terminology resources and operations in any SVCM compatible server, such as a HAPI FHIR server, even if not connected to an OCL Terminology Server (standalone mode)
1. Multi-server capability in a single instance of the OCL TermBrowser, starting with OCL and SVCM/FHIR servers
1. Easy bulk importing of user content via OCL TermBrowser

### OpenMRS Dictionary Manager
1. Implement features and workflow to streamline concept dictionary assembly, including ability to reuse concepts from dictionaries (in addition to sources), non-breaking changes (e.g. add translation) of trusted concepts, and bundling of multiple dictionaries
1. Improve the OCL Subscription Module for OpenMRS, including an improved workflow for updating a subscription and better messages documenting the results (and any errors) of a subscription
1. Improve availability of CIEL updates for the OpenMRS and OCL communities by improving CIEL release process, concept proposals, and source management within OCL
1. Drive users toward high quality trusted content ([examples here](https://docs.google.com/document/d/1T6iK3c-DC4mJFpFa7ngRavtCWcVbYcc8ivZlJgqb1GY/edit#)), possibly through a ranking system for content publishers and sources


### Content Publication for OCL Online
1. Key reference terminologies available for use and up to date in OCL Online, starting with WHO ICD-10 and HL7 terminologies
1. Pipeline established for routine publication by Apelon of terminology content to OCL Terminology Server and addressing technical gaps. Terminologies to initially include SNOMED CT, LOINC, and possibly others in the future. The timeline and pricing for availability of these terminologies in OCL Online and for local subscription is under development.
1. PEPFAR Content (MER, DASH, MOH) available and maintained within OCL using a common information model
1. Publication of WHO Smart Guideline terminology resources
1. Support a generic, standards-based process available for community publication of DASH resources
1. Make progress toward source management of CIEL in OCL to enable an author’s ability to quickly respond to change requests, to reduce publication overhead, and to enable editorial insights from derivative artifacts (eg CIEL subsets)
