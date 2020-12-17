# Release Notes
## Release Process Overview
* All OCL projects will follow the same release approach
* Publish a single consolidated Release Notes page
* We will release each project separately, but attempt to align major version numbers (eg working now on v2 of all services)
* Create ZenHub release for tickets and create GitHub release for code
* Add links to releases and build in the release notes and summarize both bug fixes and enhancements

***

## Release: oclapi-356 (September 11, 2020)
### Bug Fixes
* Multiple versions of concepts show up in a collection's HEAD version due to improper concept reference deletes ([ocl_issues/#303](https://github.com/openconceptlab/ocl_issues/issues/303))
### Meta
* [Release build](https://ci.openmrs.org/browse/OCL-OA-355)
* [Commit 4d1b8ee8f1df3b1b21f217e65220edec8c1e9cd5](https://github.com/OpenConceptLab/oclapi/commit/4d1b8ee8f1df3b1b21f217e65220edec8c1e9cd5)

***


## Release: oclapi-355 (September 9, 2020)
### Enhancements
* Bulk import returns HTTP 202 if import is in progress ([ocl_issues/#259](https://github.com/openconceptlab/ocl_issues/issues/259))
* Concept display name is returned based on dictionary locale ([ocl_issues/#264](https://github.com/openconceptlab/ocl_issues/issues/264))
### Bug Fixes
* Bulk import yields inconsistent results ([ocl_issues/#297](https://github.com/openconceptlab/ocl_issues/issues/297))
* User specific endpoint returns all organizations ([ocl_issues/#295](https://github.com/openconceptlab/ocl_issues/issues/295))
* Collections in one organization shows up in query for collections in other organization ([ocl_issues/#285](https://github.com/openconceptlab/ocl_issues/issues/285))
### Meta
* [Release build](https://ci.openmrs.org/browse/OCL-OA-355)
* [Commit 67a8b9f3f00acfd3bfc04cc7b87fa97a7d616d8d](https://github.com/OpenConceptLab/oclapi/commit/67a8b9f3f00acfd3bfc04cc7b87fa97a7d616d8d)
