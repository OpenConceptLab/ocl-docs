# Matching Algorithms

A matching algorithm finds candidate concepts in your target repository for each row of your input file. You choose one or more in the project configuration, under **Matching Algorithm**. Which algorithms you can select depends on your target repository and on how the Mapper you use is set up.

For each row, the Mapper:

1. **Retrieves candidates.** Each selected algorithm searches for concepts that match the row.
2. **Merges them.** A concept found by more than one algorithm becomes one candidate.
3. **Scores them.** Every candidate gets a unified score from 0 to 100, whichever algorithm found it.
4. **Groups them.** Your score thresholds sort the candidates into Recommended, Available and Low Ranked. AutoMatch proposes a row's best candidate as its match when that candidate is Recommended.
5. **Recommends a match, if you ask.** On OCL Online, the OCL AI Assistant reviews the candidates and recommends one, with its reasons.

Everything on this page is part of the open-source OCL Mapper and OCL API, except what the last section, [Available on OCL Online](#available-on-ocl-online), describes. The OCL algorithms run through the OCL API's `$match` operation, and a custom algorithm answers the same request. The [`$match` reference](../oclapi/apireference/match.md) specifies that request and its response.

## The algorithms

The OCL Mapper offers three algorithms:

- **OCL Search Algorithm**: keyword search, plus codes and mappings.
- **OCL Semantic Algorithm**: search by meaning.
- **Custom Match Algorithm**: calls a matching service you run.

OCL Online adds two more, Bridge terminology search and ScispaCy LOINC search.

### OCL Search Algorithm

OCL Search runs a keyword search of your target repository in Elasticsearch. It uses the input columns you assign in **Field Configuration**:

- **Name** and **Synonyms**: keyword search of each concept's names and synonyms that tolerates small spelling differences.
- **ID**: search of concept IDs, word by word as in a text search, so a row with `ABC-123` can also find a concept whose ID is `DEF-123`, not only the exact ID.
- **Mapping: Code** and **Mapping: List**: concepts that share a mapping with the row, for example to the same LOINC code.
- **Description**, **Property: Class**, **Property: Datatype** and other properties: string matching.

It works with any target repository. Pick it when your input uses the same wording as the target terminology, when you match on codes or mappings, or when the target repository doesn't support semantic search.

### OCL Semantic Algorithm

OCL Semantic searches by meaning instead of by keywords. The OCL API stores a vector for each name and synonym in a repository that has semantic search, made with a sentence-embedding model (by default `all-MiniLM-L6-v2`). The algorithm compares the row's **Name** and **Synonyms** with those vectors, so it can find a concept whose name uses different words from your input. Exact matches are boosted: a concept whose name is the row's name, or whose synonyms include one of the row's synonyms. The **ID** and mapping columns count as they do for OCL Search.

It needs a target repository version with semantic search turned on: one whose `match_algorithms` setting includes `llm`. Only staff accounts can change that setting. If your target repository version doesn't have it, you can't select OCL Semantic, and switching a project to such a version removes OCL Semantic from it.

Pick it when your terms are worded differently from the target terminology, as local and free-text terms often are. It takes longer per row than OCL Search, so AutoMatch sends it 10 rows per request by default, against 50 for OCL Search.

### Custom Match Algorithm

A custom algorithm calls a matching service you run. You configure:

- **API URL**: where the Mapper sends match requests. When you enter it, the Mapper sends a `GET` request to it and fills in **ID**, **Name** and **Description** from the `id`, `name` and `description` in the response. If the service answers 401 or 403, the Mapper asks for a token.
- **API Token**, if your service needs one. The Mapper sends it in an `Authorization: Token <token>` header. It never sends your OCL credentials to a custom algorithm.
- **Canonical URL**, required: the canonical URL of the code system whose codes your service returns, such as `http://loinc.org`. Its results count as concepts in your target repository only when this matches the target repository's canonical URL.

For each batch of rows, the Mapper sends your service a `POST` request, the same request it sends to the OCL API's [`$match` operation](../oclapi/apireference/match.md), and expects a response in the same shape: one entry per row, with the `row` as it was sent, `__index` included, and a `results` list of concepts, each with an `id` (its code) and a `display_name`. The reranker scores them like any other candidates, with one exception. When the custom algorithm is a project's only algorithm, AutoMatch takes each result's `search_meta.search_normalized_score`, if your service returns one, as its unified score, so return that score on the 0 to 100 scale.

### Batch size and concurrent requests

AutoMatch sends rows to these algorithms in batches. **Batch Size** is the number of rows in one request, and **Concurrent Requests** the number of requests it keeps open at once. Lower the batch size for a slow service whose requests time out. See "Batching & Concurrency" in the [`$match` reference](../oclapi/apireference/match.md).

## One algorithm or several

**One algorithm** is the simplest place to start: every candidate comes from a single method. Use OCL Semantic Algorithm if your target repository supports it, and OCL Search Algorithm if it doesn't.

**Several algorithms** widen the net. You can select up to five. Add a second algorithm when one method alone misses matches, for example a custom algorithm built for your terminology alongside OCL Semantic Algorithm.

**OCL Search and OCL Semantic can't be selected together**, and OCL Semantic needs a target repository version with semantic search.

### How candidates are merged

OCL Search and OCL Semantic each return up to 15 candidates per row, and **Fetch More** in the candidate list asks the algorithms for the next ones. The Mapper identifies every candidate by the canonical URL of its code system and its code. A concept that several algorithms found is therefore one candidate, labeled with every algorithm that found it.

### How candidates are scored

Each algorithm scores its candidates in its own way, so those scores can't be compared with each other. The Mapper gives every candidate a **unified score** from 0 to 100 instead. It comes from the OCL API's reranker, a cross-encoder model (by default `BAAI/bge-reranker-v2-m3`) that reads the row's **Name** next to the candidate's name and up to five of its synonyms, the five closest to the row's name in spelling. The candidate scores as its best-matching name.

- When a project has one algorithm, and it's OCL Search or OCL Semantic, the reranker runs in the same request and the candidates arrive scored.
- Otherwise, with several algorithms, a custom algorithm or one of OCL Online's, the Mapper sends the row's merged candidates to the OCL API's `$rerank` endpoint once every algorithm has answered for the row. Until then, the candidates wait under **Unranked Candidates**.

The unified score measures how well the names match. Codes, mappings and the other columns help the algorithms find candidates, but they don't change the score, so assign the column that holds your terms' names to **Name**. A row without a name gets no unified score, and its candidates stay unranked. The exception is a project whose only algorithm is OCL Search or OCL Semantic: there, such a row's candidates get the search's own score relative to its best candidate, which scores 100, so AutoMatch always proposes it.

The candidate list sorts by **Unified Score**. To see each algorithm's own score, sort by **Raw Algorithm Score**; the list then groups the candidates by algorithm, since those scores aren't comparable across algorithms.

## Score thresholds

Under **Score Configuration** in the project configuration, a slider sets two thresholds on the unified score, **Recommended** (99 by default) and **Available** (70 by default). They sort the candidates into three groups:

- **Recommended**: a score at or above the Recommended threshold.
- **Available**: a score at or above the Available threshold, but below the Recommended threshold.
- **Low Ranked**: a score below the Available threshold. The configuration panel calls this group **Not Recommended**.

The thresholds apply in three places:

- **The candidate list.** Grouped by **Match Quality**, the default, a row's candidates appear under **Recommended Candidates**, **Available Candidates** and **Low Ranked Candidates**, and the low-ranked group starts collapsed. Only concepts in your target repository are grouped this way.
- **AutoMatch.** When a row's best candidate scores at or above the Recommended threshold, AutoMatch proposes it as the row's match, and the row becomes **Proposed**, ready for your review. Rows that AutoMatch runs on without such a candidate are left **Unmapped**.
- **The row list.** Each row has a match quality: the group of its proposed match or, where no match is proposed yet, of its best candidate. The row list counts the rows in each group, and you can show only the rows in one group.

Changing the thresholds regroups candidates and rows at once, but it doesn't change matches already proposed: AutoMatch uses the new Recommended threshold the next time it runs. A higher Recommended threshold means fewer proposed matches that are more likely to be right; a lower one proposes more, and leaves more for you to check.

## Available on OCL Online

[OCL Online](https://openconceptlab.org/) is where the OCL tools run for the global community, hosted by OCL. Its Mapper adds two algorithms and the OCL AI Assistant, which run on services that OCL hosts, so a Mapper you install yourself doesn't offer them. Your OCL Online account decides which of them you can use, and it can also limit the batch size and concurrent requests you can set and how much matching you can run.

### Bridge terminology search

Bridge terminology search uses CIEL, an interface terminology with mappings to many reference terminologies, as a bridge. It runs a semantic search of CIEL, limited to CIEL concepts that have mappings into your target repository, and follows those mappings: the concepts they point to in your target repository become candidates. It can find target concepts that a direct search of the target repository misses.

In the candidate list, a candidate found this way shows the CIEL concept it came through and the map type of that mapping. The CIEL concepts themselves are never proposed as matches. They appear when you group candidates by algorithm.

Pick it when CIEL has mappings into your target repository, for example alongside OCL Semantic Algorithm; it isn't available for every target repository. It matches one row at a time, so Batch Size and Concurrent Requests don't apply to it.

### ScispaCy LOINC search

ScispaCy LOINC search sends the row's **Name** to a matching service that OCL hosts, built on scispaCy, an open-source library for processing biomedical text, and gets back LOINC codes with their long common names. It uses no other column, and skips rows without a name.

Its results are LOINC codes, so they become candidates in your target repository only when that repository is LOINC, with the canonical URL `http://loinc.org`. Pick it alongside OCL Search or OCL Semantic when you map laboratory test names to LOINC. It matches one row at a time, so Batch Size and Concurrent Requests don't apply to it.

The service can take a few minutes to start. Meanwhile the Mapper says it's warming up and tries again every two minutes, for up to ten minutes.

### Where the OCL AI Assistant fits

The OCL AI Assistant doesn't search for candidates. It works on the candidates the algorithms found: it weighs a row's candidates against the row's fields and your project's **Context**, then recommends one of them as the match and others as alternates, or none if nothing fits, and explains why. The Mapper gives it the row's candidates in your target repository to choose from, and the CIEL concepts that Bridge terminology search went through as context only.

You can run it:

- on one row, from the candidate list, or
- in AutoMatch, by selecting **Recommend matches with the OCL AI Assistant**. It then runs after candidates are retrieved and scored, and skips rows it has already analyzed with the same candidates. Without **Retrieve candidates**, it analyzes only rows that already have candidates.

Its analysis appears in the candidate list with its assessment, its primary candidate, the alternates and its reasoning, and you can group the candidates by **AI Analysis**. Each analysis stays with the row, so you can page back to earlier ones. In the row list, an icon next to a row's match shows whether the OCL AI Assistant recommended that match, suggested it as an alternate, or neither.

The OCL AI Assistant doesn't change which match AutoMatch proposes: that still follows the Recommended threshold. The assistant's recommendation sits beside it for you to accept or reject. In **Field Configuration**, **Send to OCL AI Assistant** chooses which input columns it receives, and the project's **Input Language** decides which languages' concept names it gets.
