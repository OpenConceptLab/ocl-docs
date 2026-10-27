# How Global Search Works

Global search looks across all of OCL at once. It returns concepts and mappings from the latest released version of each source, so one search shows what source owners have published for everyone to use.

To run one, type a term in the search box at the top of the TermBrowser and choose **Search all of OCL**. On a repository's page, the same box can also search just that repository.

This page covers global search for concepts and mappings, not for repositories, organizations or users.

## What global search includes

| Included | Excluded |
|---|---|
| The latest release of a **public** source | A source that hasn't released a version yet |
| | Changes made in HEAD since the latest release |
| | Released repo versions older than the latest |
| | Unreleased repo versions |
| | Collections |

To find anything in the right-hand column, open the repository and search there. The TermBrowser opens a repository at its latest released version, or at HEAD if it has no release. To search another version, pick it from the version menu next to the repository's name.

## Getting your source into global search

Release a version of your source. At HEAD, choose **Actions** > **Create Version** with **Release** checked, or open a version you've already created and choose **Actions** > **Release Version**. Global search moves to each new release once it's indexed, which can take a while for a large source.

## Global search in the OCL API

An API request runs a global search the same way when it sends the header `INCLUDESEARCHLATEST: true`, as the TermBrowser does. Without that header, a search across OCL covers each source's HEAD instead. The previous TermBrowser (TBv2) doesn't send it, so its results can differ.
