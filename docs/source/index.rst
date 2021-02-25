What is OCL?
------------
OCL is an open-source terminology management system to help you collaboratively manage, publish and use your metadata in the cloud alongside the global community.

Here are the main use cases for adopting OCL

.. image:: OCL_overview.png
  :width: 800
  :alt: Ocl Overview

OCL Overview
-------------

Learn more about OCL, its tools and the core features.

* **Overview of core OCL tools:** :doc:`OCL API Overview</oclapi/overview>` |  :doc:`OCL FHIR Overview</oclfhir/overview>` | :doc:`API Reference of OCL</oclapi/apireference/index>`
* **OCL roadmaps:** :doc:`OCL Roadmap 2021 </roadmaps/2021/roadmap>` |  :doc:`OCL Roadmap 2020 </roadmaps/2020/roadmap>`
* **OCL releases:** :doc:`Release Notes </oclapi/releases>` 


Getting Started with OCL
-------------------------
Dive deeper into OCL features and how you can get started interacting with OCL. 

* **As a developer:** :doc:`Getting Started as a Developer</oclapi/developer/gettingstarted>` | :doc:`Developers Guide</oclapi/developer/developersguide>`

Advanced Features of OCL
--------------------------

OCL offers many advanced features. Learn more about these integrations and how you can interact with OCL.

* **Importing into OCL:** :doc:`Bulk Importing into OCL</oclapi/apireference/bulkimporting>` | :doc:`Importing CIEL into OCL </oclomrs/importingciel>`
* **Integration layer:** Interface layer is a set of API endpoints to supports PEPFAR usecases through OpenHIM mediators. `Interface layer API <https://documenter.getpostman.com/view/10981858/TW77f3MH?version=latest>`_


OCL Servers
------------

.. list-table:: Title
   :widths: 25 25 50
   :header-rows: 1

   * - Server
     - URL
     - Description
   * - AWS Production API
     - https://api.aws.openconceptlab.org/ 
     - OCL API v2 Production server 
   * - AWS Production APP
     - https://app.aws.openconceptlab.org
     - OCL Web v2 Production server 
   * - AWS Staging API
     - https://api.staging.aws.openconceptlab.org/ 
     - OCL API v2 Staging server 
   * - AWS Staging APP
     - https://app.staging.aws.openconceptlab.org
     - OCL Web v2 Staging server 
   * - AWS QA API
     - https://api.qa.aws.openconceptlab.org/ 
     - OCL API v2 QA server 
   * - AWS QA APP
     - https://app.qa.aws.openconceptlab.org
     - OCL Web v2 QA server 
   * - Production API
     - https://api.openconceptlab.org/ 
     - OCL API v1 Production server 
   * - Production APP
     - https://openconceptlab.org
     - OCL Web v1 Production server 
   * - Staging API
     - https://api.staging.openconceptlab.org/ 
     - OCL API v1 Staging server 
   * - AWS Staging APP
     - https://staging.openconceptlab.org
     - OCL Web v1 Staging server 
   * - QA API
     - https://api.qa.openconceptlab.org/
     - OCL API v1 QA server 
   * - QA APP
     - https://qa.openconceptlab.org/
     - OCL Web v1 QA server 


.. toctree::
   :maxdepth: 4
   :hidden:
   :caption: OCL API
   
   oclapi/overview
   oclapi/apireference/index
   oclapi/developer/index
   oclapi/admin/index
   oclapi/releases

.. toctree::
   :maxdepth: 2
   :hidden:
   :caption: OCL on FHIR
   
   oclfhir/overview
   oclfhir/codesystem
   oclfhir/valueset
   oclfhir/conceptmap

.. toctree::
   :maxdepth: 2
   :hidden:
   :caption: OCL for OpenMRS
   
   oclomrs/openmrstooclmapping
   oclomrs/importingciel

.. toctree::
   :maxdepth: 2
   :hidden:
   :caption: OCL Roadmaps

   roadmaps/2021/roadmap
   roadmaps/2020/roadmap
