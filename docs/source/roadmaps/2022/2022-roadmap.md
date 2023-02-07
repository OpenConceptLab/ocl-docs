# 2022 OCL Technical Roadmap

## Overview
### What is the OCL Open-source Technical Roadmap?
The **Open Concept Lab Technical Roadmap** is a set of goals that represent the OCL Community’s priorities across our major areas of work, including each of our community-supported open-source tools. These goals help the community to be intentional about meeting real needs.

Development of the OCL Technical Roadmap follows this general process:
- We will follow an open roadmap development process by soliciting community input through several channels (including meetings, tickets, Slack channels) and publishing a draft publicly for final review and prioritization.
- Product releases will be used to structure feature prioritization and we are seeking volunteer “Release Managers” for each tool.
- Timelines and priorities are dependent on resources, voluntary contributions, and client needs and will change accordingly.

## OCL Toolkit Overview
The OCL community develops and supports these tools as open-source software that is free to download and use under a Mozilla Public License, v. 2.0.:
- **OCL Terminology Service**: Core REST API, FHIR-enabled terminology service
- **OCL TermBrowser**: Customizable tool for searching, visualizing, and managing content in OCL
- **OpenMRS Dictionary Manager**: Specialized tool for managing clinical concept dictionaries for [OpenMRS](https://openmrs.org)

**OCL Online**, hosted at [openconceptlab.org](https://openconceptlab.org/), is where you can use OCL’s open-source tools in the cloud along with the global community and is pre-loaded with key terminologies.

![OCL Jun-Dec](https://raw.githubusercontent.com/OpenConceptLab/ocl-docs/main/docs/source/roadmaps/2021/OCL%20Jun-Dec.png)

## OCL Year of the User
The OCL Community’s theme for 2022 is the **Year of the User**. This is a reflection of the expected growth in new users in 2022 and a shift in the core OCL team’s focus toward supporting implementers. This new focus is evident throughout our roadmap, especially in the user experience related goals for the TermBrowser and OpenMRS Dictionary Manager, and with the inclusion of OCL Online and OCL Community as key categories.

In light of this, user experience and design expertise are being embedded as routine parts of our development process moving forward. OCL is structuring user experience research to align with the user roles defined in the Terminology Management Maturity Model maintained by the OpenHIE Terminology Services Subcommunity. A summary of these user roles is included below. Look for new features and user documentation targeting these user roles later in 2022.

| Role | Description |
| ---- | ----------- |
| Terminology Publisher | As a terminology publisher, I want to manage and publish original terminology resources, leveraging and mapping to reference vocabularies and other user-generated content, in a shared environment where I can learn from other users. As a content manager that wants to maximize adoption of the content I publish, I might also want to configure a presentation layer (e.g., MSPv2, Ethiopia NHDD) to make it easier for my users to interact with my content. |
| Terminology Implementer | As a terminology implementer, I adapt and repackage terminology resources from terminology publishers, such as WHO Digital Documentation of COVID-19 Certificates (DDCC)/SMART Guidelines or CIEL, based on my implementation needs, e.g. by adding a local dialect or mapping to local codes. |
| Terminology Consumer | As a health provider or administrator, I want an intuitive interface to browse, search, compare and export terminology resources that are relevant to my project(s) so that I can review and understand the meaning of concepts, use the right codes, and navigate relationships between terms. |

## 2022 OCL Technical Roadmap

### OCL TermBrowser
1. **SVCM and WHO ICD-11 Terminology Browsing:** Enable browsing of the WHO’s ICD-11 FHIR service and of any SVCM-compatible terminology servers, such as HAPI FHIR, directly in the TermBrowser. Also, incorporate full SVCM support into the primary OCL Connector.
2. **Enhanced Multi-server Terminology Browsing:** Support connecting the TermBrowser to multiple terminology servers (e.g. OCL, WHO ICD-11, HAPI FHIR) enabling users to search, compare and map to concepts across servers.
3. **UX Improvements for the Terminology Consumer Persona:** Create an enhanced user experience (UX) for less technical users whose primary use case for OCL consists of searching, browsing, and consuming curated content published in OCL. A key example of this is the new version of the PEPFAR Metadata Sharing Platform built on top of the OCL TermBrowser. UX testing will be embedded as a routine part of the TermBrowser development process to create a more intuitive experience.
4. **Better Collection and Value Set Management:** Among the most requested features for OCL is better support for collection and value set management workflows, which will make it easier to build and maintain collections in the TermBrowser. We will also expose upcoming terminology service features, including dynamic references, collection version expansions, concept modifications, and content tagging.

### OpenMRS Dictionary Manager
1. **OCL TermBrowser Integration:** Integrate OpenMRS Dictionary Manager and Open Concept Lab TermBrowser, improving sustainability and enhancing capabilities available to both the OpenMRS and OCL communities.
2. **Cross-team Capacity Building:** Strengthen capacity to contribute to terminology and concept dictionary management priorities within OpenMRS through extending and strengthening the  fellowship program and the OCL for OpenMRS Squad.
3. **OpenMRS Subscription Module:** Improve the dictionary subscription workflow and reliability, incorporating new features such as dynamic references and concept modifications.
4. **Revision and Publication of Key Documentation:** Revise and Expand field-staff-friendly documentation for the integrated Dictionary Manager and TermBrowser web application.
5. **Implementation of New API Features:** Implement new priority features, including concept modifications, shifting complex logic from the UI to the API (e.g. linked sources, dynamic references and cascading)

### OCL Terminology Service
1. **Launch OCL FHIR Core and improve SVCM compatibility:**
	- Launch OCL FHIR Core [2022Q1]: Launch the first version of the new OCL FHIR Core. This replaces and upgrades OCL’s current FHIR service with native support built into the OCL API core. This change improves performance, service reliability, reduces complexity, and makes the complete set of OCL core’s capabilities available to the FHIR service for future enhancements.
	- Refresh FHIR Terminology Resources [2022Q2]: Re-publish the complete set of WHO DDCC/SMART Guidelines and HL7 Terminologies using the new OCL FHIR Core.
	- Advanced FHIR Support [2022Q3/4]: Implement advanced FHIR capabilities, possibly including $subsumes, hierarchy, extensions, multi-tenant, and Computable Guideline CodeSystem and ValueSet requirements (see CPG IG)
2. **Trusted Sources and Canonical URL Registry:** Steer users toward reference vocabularies and other trusted content, backed by a managed registry of canonical URLs.
3. **Advanced User Groups/Permissions:** Give administrators more control over who has access to view and manage resources. We will do this by extending the existing model of organization membership to support user groups and role-based permissions.
4. **Dynamic References, Expansions and other Advanced Features:** New capabilities prioritized by the global community will be made available, including Linked Sources, Dynamic References and Collection Version Expansions, Content Tagging, Concept Modifications, Advanced parameterized search, and other back-end requirements to support integration with the OpenMRS Dictionary Manager.

### OCL Infrastructure
1. **Bundle OCL with Instant OpenHIE:** Streamline OCL installation and deployment by bundling OCL’s open-source tools into Instant OpenHIE in alignment with community-prioritized configurations.
2. **Cybersecurity Enhancements:** Strengthen OCL cybersecurity in alignment with FISMA certification and WHO hosting requirements, e.g. routine testing of failover and recovery scenarios.
3. **Infrastructure Auto-Scaling:** Improve service availability under load by automatically scaling up infrastructure resources when running out of resources and scaling down when not under load.
4. **Automate Performance & Load Testing:** Incorporate performance and load testing as a part of routine releases, set 2022 performance targets emphasizing key bottlenecks and areas of expected growth (e.g. bulk imports, exports), and make the results publicly available.
5. **Unit and Integration Test Coverage:** Improve reliability by increasing unit and integration test coverage across OCL open-source products to at least 90%. Implement browser-based test coverage to 50 or more tests.
6. **Automate Infrastructure Notifications/Reports:** Improve system maintainability by implementing a framework for automated notifications and reports, especially from AWS CloudWatch, and implementing notification for long running requests > 10s on all environments.

### OCL Online
1. **Publish Reference Vocabularies and other Emerging Standards:** Key reference vocabularies available for use and up-to-date in OCL Online, starting with ICD-10, SNOMED-GPS and LOINC, along with emerging standards for projects such as WHO DDCC/SMART Guidelines and PEPFAR metadata.
2. **Expand User Documentation:** Given expected growth in users and traffic to OCL Online during 2022, we aim to create new user documentation to meet user needs.
3. **Improve Community Site for new community members and participants:** Enhance the OCL Community Site to be a valuable resource for implementers and for people new to the OCL community.

### OCL Community
1. **Host Regular Community Events:** OCL will host regular community events to share information on the OCL toolkit and to address community needs. Includes quarterly Technical Q/A sessions, showcases for major releases, and office hours for implementers and end-users.
2. **Kick-off Communications Strategy:** The OCL secretariat will prepare regular communications directed to the OCL community of users and potential-users including: emails, blog posts, YouTube videos, and social media.
3. **Define User Personas:** Proactively work with the OCL community to define user personas and user journeys in alignment with the Terminology Management Maturity Model being revised by the OpenHIE Terminology Services Subcommunity
