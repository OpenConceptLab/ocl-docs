# Updating Core Table Values for OCL TermBrowser
Updated 2024-Feb-15

**Owner:** [Joe](https://github.com/jamlung-ri/)

**Maintainer:** [Joe](https://github.com/jamlung-ri/)

The OCL TermBrowser includes standard taxonomies and terminology schemas that facilitate standardization of terminology content across organizations and repositories. Some of these standard values, including Map Types, Description Types, Name Types, Datatypes, Locales, and Concept Classes populate drop down menus in the OCL TermBrowser interface, which users encounter when creating new concepts, seleting a default locale for a repository, etc. 

These standard values can generally be found in the OCL organization on OCL Online (or your local instance of OCL) e.g. https://app.openconceptlab.org/#/orgs/OCL/sources/

As new options need to be added over time to support evolving use cases, these Core tables may need to be updated occasionally. By following the steps outlined below to update the Core tables, new options will then appear in the TermBrowser interface for end users. 

If the need is for new values to appear in OCL Online, contact an OCL administrator with the proposed new values. They will review the request and, if appropriate, follow the process outlined below to add the new values to OCL's standard values. 

Adding/changing Core table values in an OCL instance:
1. Locate the standard table JSON files in the `lookup_fixtures` folder in the GitHub repository e.g. https://github.com/OpenConceptLab/oclapi2/tree/master/core/lookup_fixtures
2. Add new values to the appropriate json file, following the formatting and attributes of other values in the file. Repeat until all values have been updated in their respective files in the repository.
3. Commit changes to the GitHub repository
4. An OCL Administrator generates a data migration to the appropriate OCL instance(s)
5. Once deployed everywhere, confirm that values are appearing appropriately in the sources in the OCL organization on OCL TermBrowser

