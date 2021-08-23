OCL Content Publication Plan
----------------------------

This document outlines OCL’s plans to publish and maintain selected vocabularies in OCL Online. OCL Online is a hosted implementation
of all of OCL’s open-source tools in the cloud and pre-loaded with key terminologies, available at openconceptlab.org.

OCL’s Goal for Content Publication
----------------------------------
OCL aims to publish and maintain key reference vocabularies and other emerging health data standards, especially those that are
important to low and middle income countries. Published content will be available on OCL Online and may also be available for distribution.

Scope of Content Publication
----------------------------------

.. list-table::
   :widths: 30 85 85
   :header-rows: 1

   * - Status
     - Reference Vocabularies
     - Emerging Health Data Standards
   * - Available in OCL Production
     - - HL7 FHIR Terminologies
     - - CIEL Interface Terminology
   * - Planned and In Progress
     - - WHO ICD-10
       - LOINC
       - SNOMED-CT
       - RxNorm
       - US ICD-10-CM
     - - WHO Smart Guidelines
       - PEPFAR MER and MOH Alignment
       - Partners in Health (PIH) OpenMRS Concept Dictionary

How do we validate content when we publish it in OCL?
----------------------------------
To ensure that the content loads into OCL as expected, there are several validation steps taken for each set of content to check for accuracy between what should show in OCL versus what is actually appearing in OCL.

1. Spot Checks: Looking at 5 or more exemplary concepts in depth, we check that the concept appears in OCL and that all the expected attributes are present. If applicable, we check that these concepts are correctly placed in their hierarchy.

2. Hierarchy Check: If the content is hierarchical, we look down its tree, particularly looking for concepts that should but do not have a parent concept.

3. Automated Script: A python script uses two files, the bulk import file and the export file created within OCL, for a set of content to check for multiple potential errors:
    - Missing concepts and mappings that are not present in OCL
    - Extra concepts and mappings that are present in OCL but should not have been loaded in
    - Checks for duplicate concept or mapping IDs in the export file
    - Compares concept attributes to ensure these attributes are similar between the import and the export files


How do we keep content up-to-date?
----------------------------------
Each terminology has its own process and schedule for releasing updates. OCL and its partners, Apelon and the Regenstrief Institute,
work actively to keep published content up-to-date. The table below outlines the release process for curated terminologies published in OCL production.

Note that most content available in OCL is managed directly by other users or organizations and thus will have their own release processes.


.. list-table::
   :widths: 40 80
   :header-rows: 1

   * - Content
     - Release Process
   * - CIEL
     - Approximately monthly releases, made available on OCL within 72 hours
   * - HL7 FHIR Terminologies
     - Updated only as needed
   * - PEPFAR MOH Codelists
     - New codelists published annually, with maintenance releases as needed
