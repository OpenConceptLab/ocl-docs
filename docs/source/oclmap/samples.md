# Mapper Sample Files

Use these CSVs to try OCL Mapper with a recommended target repository. **Guided demos** use clear terms to make matching easier to follow. **Real-world samples** use less uniform wording and are intended for a more challenging trial; results may need more review. For every sample, you can use its domain reference terminology or **CIEL:CIEL** as the target. Comparing both illustrates how an interface terminology can support mapping across clinical domains. Repository links open the corresponding source entries in OCL Online; sign-in or access may be required.

## Start a project

1. [Create a Mapper project](https://map.openconceptlab.org/map-projects/new).
2. Choose a project owner and name, then upload one CSV under **Input Data**.
3. Set **Input Language** to the language used in the file.
4. Select the sample's suggested **Target Repository** (`ORG:SOURCE`) and a version. This is where Mapper searches for candidate concepts.
5. In column configuration, map **Name** to **Name**. In the four guided demo files, `Code` contains CIEL identifiers; the original lab sample uses local codes, and the real-world files use row numbers. Leave `Code` unmapped unless it matches identifiers in the target.
6. Review candidates and confirm matches. The project is saved under its project owner; selecting a target repository searches its concepts but does not publish your mapping decisions there. Use **Save to Collection** to save selected concepts to a collection.

![Mapper project setup with the Input Data upload control highlighted](images/create-mapping-project-upload.jpg)

## Guided demos

The four guided demo files use `Code,Name`; the original lab sample also has a `Specimen` column. These examples help users inspect candidate results, but they do not guarantee an exact or equivalent match for every row.

| Sample | Suggested target repositories (`ORG:SOURCE`) | Download |
|---|---|---|
| Diagnoses | [WHO:ICD-10-WHO](https://app.openconceptlab.org/#/orgs/WHO/sources/ICD-10-WHO) or [CIEL:CIEL](https://app.openconceptlab.org/#/orgs/CIEL/sources/CIEL) | [CSV](samples/mapper_sample_github_diagnoses.csv) |
| Procedures | [IHTSDO:SNOMED-GPS](https://app.openconceptlab.org/#/orgs/IHTSDO/sources/SNOMED-GPS) or [CIEL:CIEL](https://app.openconceptlab.org/#/orgs/CIEL/sources/CIEL) | [CSV](samples/mapper_sample_github_procedures.csv) |
| Lab tests and imaging | [Regenstrief:LOINC](https://app.openconceptlab.org/#/orgs/Regenstrief/sources/LOINC) or [CIEL:CIEL](https://app.openconceptlab.org/#/orgs/CIEL/sources/CIEL) | [CSV](samples/mapper_sample_github_lab_imaging.csv) |
| Mixed domains (general purpose) | [CIEL:CIEL](https://app.openconceptlab.org/#/orgs/CIEL/sources/CIEL) | [CSV](samples/mapper_sample_github_general_ciel.csv) |
| Lab tests (original sample) | [Regenstrief:LOINC](https://app.openconceptlab.org/#/orgs/Regenstrief/sources/LOINC) or [CIEL:CIEL](https://app.openconceptlab.org/#/orgs/CIEL/sources/CIEL) | [CSV](samples/mapper_sample_lab_tests.csv) |

## Real-world samples

These three worksheets came from **Demo files for Mapper.xlsx**. They use informal, abbreviated or less predictable terms to show a harder matching task. Their term columns were renamed to `Name`; since none contains specimen data, each CSV has only `Code,Name` (comma-separated). The `Code` values are row numbers, not target terminology codes.

| Sample | Suggested target repositories (`ORG:SOURCE`) | Download |
|---|---|---|
| Diagnoses | [WHO:ICD-10-WHO](https://app.openconceptlab.org/#/orgs/WHO/sources/ICD-10-WHO) or [CIEL:CIEL](https://app.openconceptlab.org/#/orgs/CIEL/sources/CIEL) | [CSV](samples/mapper_sample_realistic_diagnoses.csv) |
| Procedures and imaging | [IHTSDO:SNOMED-GPS](https://app.openconceptlab.org/#/orgs/IHTSDO/sources/SNOMED-GPS) or [CIEL:CIEL](https://app.openconceptlab.org/#/orgs/CIEL/sources/CIEL) | [CSV](samples/mapper_sample_realistic_procedures_imaging.csv) |
| Medications | [CIEL:CIEL](https://app.openconceptlab.org/#/orgs/CIEL/sources/CIEL) | [CSV](samples/mapper_sample_realistic_medications.csv) |

The medication target is a broad starting point, not a claim that every drug name has an exact CIEL match. Compare candidates with the intended clinical meaning before confirming a mapping. For all samples, check that the suggested source and version are available in your Mapper instance; repository access and content can vary.
