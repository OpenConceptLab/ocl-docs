Verified repository versions
============================

A Verified mark on a repository version means its content was checked against the publisher's official release using the process below. Every difference from that release was reviewed, accepted and listed on the version's About tab.

What Verified covers
--------------------

Verification applies to a released version, not to HEAD or to every version of a repository. It applies to reference terminologies published by another organization and maintained by OCL, not community copies or repositories maintained on OCL by their own publishers.

The mark is not an endorsement by the publisher or a judgment of the terminology's quality. It does not mean that the version is the latest release. A version without the mark is not necessarily of lower quality.

What is checked
---------------

Each version is checked using a documented process. The checks below explain what the Verified mark means. The process can be adapted to the way a publisher makes a terminology available.

From the publisher
~~~~~~~~~~~~~~~~~~

The version is checked against the publisher's official release. The source is recorded so the same release can be used again for the checks. Depending on the terminology, the source may be a release file or a recorded copy of the publisher's data.

License and citation
~~~~~~~~~~~~~~~~~~~~

The publisher's license and citation are available on the repository and included in exports. Users should still follow the publisher's terms of use.

Correctly identified
~~~~~~~~~~~~~~~~~~~~

The name, official identifier, version and release date are checked against the publisher's release information.

Complete
~~~~~~~~

The content is compared with the official release to check for missing or extra entries. Every difference found is reviewed, accepted and listed (see Differences listed).

Differences listed
~~~~~~~~~~~~~~~~~~

The version's About tab lists how its content differs from the publisher's release: content that is added, missing or changed, with the reason for each difference. Each difference was reviewed and accepted.

Checked independently
~~~~~~~~~~~~~~~~~~~~~

An automated run checks the version again against the recorded source, and designated reviewers approve the record before the version is marked Verified. The full verification record is private and can be made available to the publisher on request.

Verification processes
----------------------

The process name identifies the recipe used to check a version. A suffix identifies an adaptation for a terminology. The descriptions below explain the recipes; they do not mean that a particular repository version has been verified.

.. _ocl-verification-v1:

ocl-verification-v1
~~~~~~~~~~~~~~~~~~~

The general recipe compares a released repository version with a recorded copy of the publisher's release. It checks the source, identification, license and citation, and content; differences are reviewed and accepted. An automated check and review are completed before the Verified mark is granted.

.. _ocl-verification-v1-atc:

ocl-verification-v1.atc
~~~~~~~~~~~~~~~~~~~~~~~

The ATC adaptation follows the general recipe using the WHO Collaborating Centre's ATC and ATC/DDD release files. The repository version is compared with those files, including the classification and defined daily doses. Differences in how the content is represented are reviewed and listed for the version.

.. _ocl-verification-v1-icd11:

ocl-verification-v1.icd11
~~~~~~~~~~~~~~~~~~~~~~~~~

The ICD-11 adaptation follows the general recipe using a recorded collection of WHO release data, including the Simple Tabulation and data captured from the WHO ICD API. The repository version is compared with that recorded source, including its concepts, names and mappings. The capture is assembled for verification; it is not a single file downloaded from WHO.

The verification record
-----------------------

The About tab shows when the version was verified, the process used and a fingerprint of the private verification record. The fingerprint can help confirm that a shared record is the one behind the mark; it does not show the evidence itself. The version's summary provides its counts, and its About tab lists the reviewed and accepted differences from the release.
