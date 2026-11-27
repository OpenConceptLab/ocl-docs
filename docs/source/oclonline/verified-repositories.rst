Verified Repositories
=====================

A verified repository version in OCL Online matches the publisher's official release, apart from the differences listed for that version. The release is checked entry by entry and every difference is reviewed, accepted and listed. You can use a verified version as you would the publisher's own release, without downloading, modeling and loading it yourself.

.. raw:: html

   <div class="verified-chip-row">
     <span class="verified-chip">
       <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M12 2 4 5v6.09c0 5.05 3.41 9.76 8 10.91 4.59-1.15 8-5.86 8-10.91V5zm6 9.09c0 4-2.55 7.7-6 8.83-3.45-1.13-6-4.82-6-8.83V6.31l6-2.12 6 2.12zm-9.18-.5L7.4 12l3.54 3.54 5.66-5.66-1.41-1.41-4.24 4.24z"></path></svg>
       Verified
     </span>
     <p>Look for this chip next to a version's name, in the repository header, in search results and on an organization's pinned repositories. Select it to see what was checked.</p>
   </div>

.. note::

   The publisher's license still applies. Each verified repository shows the license and the publisher's citation, and every export includes them.

What OCL checks
---------------

A version is verified only when it passes all six checks. These are the checks of the general process, ``ocl-verification-v1``; adaptations for a terminology are described under `Verification processes`_. The Verification section of a version's About tab lists the checks, and each one links to its description here.

.. rst-class:: verified-check

From the publisher
~~~~~~~~~~~~~~~~~~

The version is checked against the release from the publisher's own distribution: its website, API or download service. The source is recorded, with a checksum, so the same release can be used again for the checks. Depending on the terminology, the source may be a release file or a recorded copy of the publisher's data.

.. rst-class:: verified-check

License and citation
~~~~~~~~~~~~~~~~~~~~

The release's license allows OCL to redistribute it, or the publisher has given permission. The repository shows the license and the publisher's citation, and every export includes them. Users should still follow the publisher's terms of use.

.. rst-class:: verified-check

Correctly identified
~~~~~~~~~~~~~~~~~~~~

The repository's name, canonical URL, version and release date match the publisher's. Where HL7 keeps a record of the terminology at terminology.hl7.org, the URL and name in that record are used.

.. rst-class:: verified-check

Complete
~~~~~~~~

The content is compared with the release: every entry in the release is in the repository version, and the counts match for each kind of entry, such as chapters and blocks in ICD-11. The only differences allowed are those listed for that version (see `Differences listed`_). Any other difference fails the check.

.. rst-class:: verified-check

Differences listed
~~~~~~~~~~~~~~~~~~

The version's notes, in the Verification section of its About tab, describe how its content differs from the publisher's release: content that is added, missing or changed, with the reason for each difference. Each difference was reviewed and accepted, and the check confirms that nothing else differs. Common kinds of difference are described under `How a version can differ`_.

.. rst-class:: verified-check

Checked independently
~~~~~~~~~~~~~~~~~~~~~

An automated run checks the version again against the recorded source, and designated reviewers approve the record before the version is marked Verified. The full verification record is private and can be made available to the publisher on request.

How a version can differ
------------------------

A verified version can differ from the publisher's release, for example so that the release can be browsed and searched in OCL. The version's notes say which differences it has. The kinds below are examples.

.. list-table::
   :class: verified-table
   :header-rows: 1
   :widths: 22 56 22

   * - Difference
     - What it is
     - Example
   * - Root concept
     - A single top concept, so the hierarchy can be browsed from one place. It is not part of the publisher's release.
     - ``ICD-11-Root``
   * - Concept class
     - A concept class taken from the publisher's own classification of each entry.
     - Chapter, Block or Diagnosis in ICD-11
   * - Link to the publisher's entry
     - A link from each concept to the publisher's own page or entity for it.
     - WHO Foundation URL
   * - Assigned ID
     - An ID for entries the publisher leaves without a code, built from the publisher's own numbering.
     - ``Chapter-01`` in ICD-11
   * - Concept ID rule
     - Which of the publisher's identifiers becomes the concept ID, when an entry has more than one.
     - ICD-11 code, else block ID

What Verified covers
--------------------

Verified is a statement about one released version of a repository.

* **It's OCL's check.** It isn't an endorsement by the publisher, and it isn't a judgment of the terminology's quality. The publisher didn't load or verify the version; OCL maintains it. A version without the chip isn't necessarily of lower quality.
* **It covers one version.** Other versions of the repository are covered only when they show the chip themselves. HEAD never does.
* **It doesn't track newer releases.** For the publisher's latest release, check its website.
* **Copies made by others aren't verified,** even when they share a name. Search shows the verified version first.
* **Publishers' own repositories don't need it.** The chip is for terminologies published by another organization and maintained by OCL. When a publisher maintains its terminology in OCL itself, as CIEL does, its repository is already the official source.

Finding verified repositories
-----------------------------

In repository search, verified repositories come first among the results that match, and the Verification filter shows only them. A repository counts as verified when its latest released version is verified. In concept search, a concept from a verified version shows a small Verified mark inside its version and ranks a little higher than copies of the same code.

Verification details
--------------------

The API returns a short record in a verified version's ``verification`` field: when the version was verified (``verified_at``), which process was used (``process``), and the SHA-256 of the full record of the check (``hash``). Only OCL staff can set the field. It isn't copied to a new version and it isn't part of an export.

How the version differs from the publisher's release is in the version's notes, the custom attribute ``verification_notes``, which the About tab shows. The version's summary provides its counts. The full record of each check stays private and can be made available to the publisher on request; the hash shows that a record shared with a publisher is the one behind the mark, but it doesn't show the evidence itself.

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

Questions
---------

If you think a verified version differs from the publisher's release, contact the OCL team and name the repository and version. Publishers can ask for the full record of any check.

.. The contact address is still open (OpenConceptLab/ocl_online#500, Jon, 13 Nov). Add it to the paragraph above once it is decided.
