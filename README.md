# ocl-docs
Consolidated documentation for OCL


## How to contribute -

This documentation supports both Markdown (.md) and Restructured Text (.rst) files. This repository is directly linked to the content in https://ocl-docs.readthedocs.io/ and any changes with commits will be built and deployed in that website.

#### To add/edit content in an existing page

Add or Edit content in any page under https://github.com/OpenConceptLab/ocl-docs/tree/main/docs/source and commit the changes.

#### To add a new page

Add a page under - https://github.com/OpenConceptLab/ocl-docs/tree/main/docs/source or any folder under this page

#### To build and verify changes locally

Note: If you want to edit files locally and/or are planning to make multiple edits and want to avoid multiple commits. Follow the following steps

1. Clone the repo
2. Go to `docs` folder
3. Execute `make clean` to clean earlier changes
4. execute `make html` and if the build is succesful it should return with -

  ```
  build succeeded.
  The HTML pages are in build/html.
  ```
5. You should be able to open the `index.html` file under `docs/build/html` and view the final result of your changes
6. You can commit and push your changes once you are happy with the end result
