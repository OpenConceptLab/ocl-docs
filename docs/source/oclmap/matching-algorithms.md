# Matching Algorithms

*This page is a stub. It introduces the matching algorithms in the OCL Mapper; a full guide is on the way.*

A matching algorithm finds candidate concepts in your target repository for each row of your input file. You choose one or more in the project configuration, under **Matching Algorithm**. Which algorithms you can select depends on your target repository and your account.

## The algorithms

- **OCL Search Algorithm** — Token and keyword search using Elasticsearch. Works best when your input uses the same wording as the target terminology.
- **OCL Semantic Algorithm** — Vector-based search that matches on meaning, not just keywords, so it can find a concept worded differently from your input. Available when the target repository has semantic search enabled.
- **Bridge terminology search** — Searches the CIEL interface terminology and follows its mappings into your target repository, finding candidates the direct search can miss. Available for compatible target repositories.
- **ScispaCy LOINC search** — Uses scispaCy, a biomedical language-processing library, to match laboratory terms to LOINC. Returns LOINC codes only.
- **Custom Match Algorithm** — Calls a matching service you run, at an API URL you provide.

## One algorithm or several

**One algorithm** is the simplest place to start: every candidate comes from a single method, so the scores are easy to compare. Use OCL Semantic Algorithm if your target repository supports it, and OCL Search Algorithm if it doesn't.

**Several algorithms** widen the net. Each one runs on every row, and their candidates are merged into one list per row; a concept found by more than one algorithm appears once. Add a second algorithm when one method alone misses matches — for example, OCL Semantic Algorithm plus Bridge terminology search.

OCL Search Algorithm and OCL Semantic Algorithm can't be selected together; choose one of them.
