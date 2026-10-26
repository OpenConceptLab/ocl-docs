# ocl-docs

[![DPG Badge](https://img.shields.io/badge/Verified-DPG-3333AB?logo=data:image/svg%2bxml;base64,PHN2ZyB3aWR0aD0iMzEiIGhlaWdodD0iMzMiIHZpZXdCb3g9IjAgMCAzMSAzMyIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KPHBhdGggZD0iTTE0LjIwMDggMjEuMzY3OEwxMC4xNzM2IDE4LjAxMjRMMTEuNTIxOSAxNi40MDAzTDEzLjk5MjggMTguNDU5TDE5LjYyNjkgMTIuMjExMUwyMS4xOTA5IDEzLjYxNkwxNC4yMDA4IDIxLjM2NzhaTTI0LjYyNDEgOS4zNTEyN0wyNC44MDcxIDMuMDcyOTdMMTguODgxIDUuMTg2NjJMMTUuMzMxNCAtMi4zMzA4MmUtMDVMMTEuNzgyMSA1LjE4NjYyTDUuODU2MDEgMy4wNzI5N0w2LjAzOTA2IDkuMzUxMjdMMCAxMS4xMTc3TDMuODQ1MjEgMTYuMDg5NUwwIDIxLjA2MTJMNi4wMzkwNiAyMi44Mjc3TDUuODU2MDEgMjkuMTA2TDExLjc4MjEgMjYuOTkyM0wxNS4zMzE0IDMyLjE3OUwxOC44ODEgMjYuOTkyM0wyNC44MDcxIDI5LjEwNkwyNC42MjQxIDIyLjgyNzdMMzAuNjYzMSAyMS4wNjEyTDI2LjgxNzYgMTYuMDg5NUwzMC42NjMxIDExLjExNzdMMjQuNjI0MSA5LjM1MTI3WiIgZmlsbD0id2hpdGUiLz4KPC9zdmc+Cg==)](https://digitalpublicgoods.net/r/open-concept-lab) [![Documentation Status](https://readthedocs.org/projects/ocl-docs/badge/?version=latest)](https://ocl-docs.readthedocs.io/en/latest/?badge=latest)


Consolidated documentation for OCL. This documentation supports both Markdown (.md) and Restructured Text (.rst) files. This repository is directly linked to the content in https://docs.openconceptlab.org (or https://ocl-docs.readthedocs.io/). Commits are automatically built and deployed to that website.


## How to contribute -

#### To add/edit content in an existing page

Add or Edit content in any page under https://github.com/OpenConceptLab/ocl-docs/tree/main/docs/source and commit the changes.

#### To add a new page

Add a page under - https://github.com/OpenConceptLab/ocl-docs/tree/main/docs/source or any folder under this page

#### To build and verify changes locally

Note: If you want to edit files locally and/or are planning to make multiple edits and want to avoid multiple commits. Follow the following steps

1. Clone the repo
2. install requirements using `pip install -r requirements.txt`
3. Go to `docs` folder
4. Execute `make clean` to clean earlier changes
5. execute `make html` and if the build is successful it should return with -

  ```
  build succeeded.
  The HTML pages are in build/html.
  ```
5. You should be able to open the `index.html` file under `docs/build/html` and view the final result of your changes
6. You can commit and push your changes once you are happy with the end result

## License

This repository is part of the [OCL Digital Public Good](https://github.com/OpenConceptLab/.github/blob/main/DIGITAL-PUBLIC-GOOD.md). Its documentation is licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE).
