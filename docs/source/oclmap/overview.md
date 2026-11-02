# OCL Mapper Overview

OCL Mapper is an AI-enabled, open-source terminology mapping tool.

## Quick Start

1. [Create a Mapper Project](https://map.openconceptlab.org/map-projects/new).
2. **Set up the project.** Choose a **Project Owner**, enter a name, and optionally describe the data and setting in **Context**.
3. **Upload the input.** Select a CSV, XLS, or XLSX spreadsheet with one row per local term and a header row. See [Mapper Sample Files](samples.html) for downloadable examples.
4. **Choose language and target.** Set the **Input Language**, then choose the target [**Source**](../oclapi/apireference/sources.html) or [**Collection**](../oclapi/apireference/collections.html) containing the concepts you want to map to, and select its version. This is where Mapper searches for candidate concepts.
5. **Configure matching.** Choose one or more algorithms, map **Name** to **Name**, and map additional columns only when their values match fields the algorithms can use. See [Matching Algorithms](matching-algorithms.html) for algorithm choices and [Mapper Sample Files](samples.html) for sample formats and target suggestions.
6. **Save and review.** Select **Save & Close** to save the project under its owner, run AutoMatch, and review each proposed candidate before confirming your decisions. Use **Save to Collection** to save selected concepts to a collection.

## Detailed fields overview

### Project and input

| Field | What it does | Guidance |
|---|---|---|
| **Project Owner** | Selects the user or organization workspace that owns the project. | Choose the workspace where you want to manage the project. |
| **Name** | Names the mapping project. | Required; use a name that identifies the input or task. |
| **Context** | Describes the data and its setting. The OCL AI Assistant uses it to tailor recommendations. | Optional. Include useful clinical or operational context when using the assistant. |
| **Input Data** | Uploads the spreadsheet to map. CSV, XLS, and XLSX are accepted. | Use one row per local term, with a header row. See [Mapper Sample Files](samples.html) for examples. |
| **Input Language** | Sets the language of the input terms. Candidates are shown with names in this language first; it also guides which target names are sent to the OCL AI Assistant. | Choose the language actually used in the input. |

### Target and retrieval

| Field | What it does | Guidance |
|---|---|---|
| **Target Repository** | Selects the [Source](../oclapi/apireference/sources.html) or [Collection](../oclapi/apireference/collections.html) whose concepts Mapper searches for candidates. | Choose the intended destination terminology or concept set and its version. Selecting it does not publish mapping decisions. |
| **Version** | Pins the target content version used for matching. | Required before saving. Choose the release you want to map against. |
| **Global Locale Filter** | Restricts candidate names to selected language locales in the target. | Optional. It can improve results when input and target names use the same language, but may reduce cross-lingual matches. |
| **Default Filter** | Applies the filter configured by the target repository, when one is available. | Keep it enabled when you want the repository's normal scope; disable it to search outside that preset. |

### Matching and scoring

| Field | What it does | Guidance |
|---|---|---|
| **Matching Algorithm** | Selects how Mapper retrieves candidate concepts. Some options depend on the target, account, or OCL Online. | Start with one suitable algorithm. See [Matching Algorithms](matching-algorithms.html) for what each uses and when to select it. |
| **Algorithm settings** | Many algorithms expose **Batch Size** and **Concurrent Requests**. Custom algorithms also use an ID, name, optional description, API URL, canonical URL, and sometimes an API token. Bridge algorithms use a Bridge Source URL and version. | Keep defaults unless requests time out or the service requires specific settings. Algorithm-specific options are covered on the matching page. |
| **Score Configuration** | Sets the score thresholds for **Recommended**, **Available**, and **Low Ranked** candidates. Recommended candidates can be proposed by AutoMatch. | Defaults are 99 for Recommended and 70 for Available. Adjust them to control how conservative proposals should be. |
| **Reranker Configuration** | Where available, selects the model used to calculate unified candidate scores. | The default model is selected automatically; custom model selection is restricted to authorized users. |

### Field Configuration

Field Configuration appears after a spreadsheet is uploaded. It connects each **Input Column** to a **Mapped Field** that matching can understand.

| Control or mapped field | Purpose |
|---|---|
| **Input Column** | The header from your spreadsheet. |
| **Mapped Field** | Tells Mapper what the column contains. Suggestions are a starting point and can be changed. |
| **Show** | Controls whether the column is displayed. Hidden columns are also excluded from matching requests. |
| **Send to OCL AI Assistant** | Controls whether that column is included in AI Assistant recommendations, when available. It does not change the fields sent to matching algorithms. |
| **Name** | The main local term to match. Keep this mapped whenever the input has a usable term name. |
| **Synonyms** | Alternative names for the same local term. Include them when the input provides useful variants. |
| **ID** | An identifier to search against target concept IDs. Use it only when the code is meaningful in the selected target. |
| **Description** | Additional descriptive text that may help retrieve candidates. |
| **Mapping: Code / Mapping: List** | Existing mapping codes or lists. Select the source repository for those codes so Mapper can compare mappings correctly. |
| **Property: Class / Property: Datatype / Property: _name_** | Concept class, datatype, or a target property exposed by the repository. Use only when your input values correspond to the target's stored values. |

For useful results, map the evidence your input actually contains; leave unrelated columns unmapped. Name, codes, mappings, and properties can help retrieve candidates, but additional fields do not replace a good **Name** value or change the unified score. The [Matching Algorithms](matching-algorithms.html) page explains which fields each algorithm uses. The [sample page](samples.html) gives file-specific guidance for its downloads.

### Optional or account-dependent settings

| Field | What it does |
|---|---|
| **Lookup Configuration** | Optionally supplies a repository URL and token for looking up fuller candidate concept definitions. Use it only when your environment provides a lookup source. |
| **Resolution Namespace** | Chooses the namespace used to resolve canonical URLs through OCL's URL Registry. When blank, it defaults to the project owner. |
| **Use Lexical Variants** | Where available, expands OCL search with English spelling variants such as “color/colour.” |
| **OCL AI Assistant settings** | Where enabled, the prompt template and model configure recommendations; **Output Locale** sets the language of those recommendations. Context, selected input columns, and Input Language also inform the assistant. |

These optional settings may not appear for every account. Start with the required project fields, one suitable algorithm, and accurate column mappings; add advanced settings only when your workflow needs them.
