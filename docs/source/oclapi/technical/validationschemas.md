# Validation Schemas
Concept/Mapping Lookup Values:

| Field | Basic Validation | OpenMRS Validation |
| --- | --- | --- |
| [Class](https://openconceptlab.org/orgs/OCL/sources/Classes/concepts/) | (Required) May be any non-empty value | (Required)  Must be value in OCL lookup list |
| [Datatypes](https://openconceptlab.org/orgs/OCL/sources/Datatypes/concepts/) | (Required) May be any non-empty value; Empty evaluates to "None" | (Required) Must be value in OCL lookup list |
| [Locales](https://openconceptlab.org/orgs/OCL/sources/Locales/concepts/) | (Required) Must be 2-character value in OCL lookup list | (Required)  Must be 2-character value in OCL lookup list |
| [NameTypes](https://openconceptlab.org/orgs/OCL/sources/NameTypes/concepts/) | (Optional) May be any value | (Optional) Must be empty or value in OCL lookup list |
| [DescriptionTypes](https://openconceptlab.org/orgs/OCL/sources/DescriptionTypes/concepts/) | (Optional) May be any value | (Optional) May be any value |
| [MapTypes](https://openconceptlab.org/orgs/OCL/sources/MapTypes/concepts/) | (Required) May be any value | (Required) Must be value in OCL lookup list |
