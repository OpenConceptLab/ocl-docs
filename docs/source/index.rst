OCL documentation
==================

OCL Documentation is used to provide information to users on the use, operation, maintenance, and design of OCL tools

Technical Documentation
-----------------------

* **Terminology Service**

  OCL maintains a public metadata clearinghouse with a host of tools for making it easy for entities to store and retrieve their metadata data. This is possible through the OCL Terminology service which acts as the engine of all the services and makes it possible through a list of API endpoints. But, if an entity would like to host their own data, OCL makes it possible through the Open Source code and installation steps to setup their own instance of Terminology service.

  * Getting Started guide to setup and contribute to terminology service or:
    :doc:`OCL API </oclapi/developer/gettingstarted>`
  * Learn about recent changes, enhancements and bug fixes in OCL API:
    :doc:`Release Notes </technical/releases>`

* **Interface layer**

  .. note::
    Currently Interface layer only supports PEPFAR usecases through OpenHIM mediators.

  Sometimes it is useful to interact with OCL using custom information models, rather than using traditional terminology information models. The OCL Interface Layer exposes a set of APIs for some of these custom models

  * Learn how to interact with OCL through: `Interface layer API <https://documenter.getpostman.com/view/10981858/SzmjyuQC?version=latest>`_

* **CIEL Integration**

  When a new version of CIEL is released it needs to be updated in OCL to make it available for CIEL subscribers like OpenMRS. Importing into CIEL documents the steps and people involved to make the new CIEL version available in OCL.

  * Process for `importing into CIEL <https://docs.google.com/document/d/1YyGeCvKDuxG7ceQK0KZahPjNEj2N4EdJ01p67bIgMJk>`_




.. toctree::
   :maxdepth: 4
   :hidden:
   :caption: OCL API
   
   oclapi/overview
   oclapi/user/index
   oclapi/developer/index
   oclapi/admin/index

   
.. toctree::
   :maxdepth: 3
   :hidden:
   :caption: Technical Documentation

   technical/bulkimporting
   technical/csvimport
   technical/customattributefilters
   technical/exportapi
   technical/formattedcsvresponses
   technical/importingopenmrsconceptdictionary
   technical/openmrstooclmapping
   technical/releases
   technical/subscriptions
   technical/validationschemas


.. toctree::
   :maxdepth: 2
   :hidden:
   :caption: OCL Roadmaps

   roadmaps/OCL2020roadmap
