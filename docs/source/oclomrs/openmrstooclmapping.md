# OpenMRS to OCL Mapping
This page illustrates how all fields in the OpenMRS v1.11 Concept Data Model map to OCL fields. This model is used during the import of the CIEL dictionary into OCL. It can also be used as a guide for OpenMRS implementers that are connecting to the OCL API.



## OpenMRS Concept Data Model

Note that the OpenMRS documentation is valid up to OpenMRS v1.9. Will update these links when updated documentation becomes available.

[OpenMRS Wiki on the Concept Data Model](https://wiki.openmrs.org/display/docs/Concept+Data+Model)

![OpenMRS Concept Data Model](https://wiki.openmrs.org/download/attachments/36671129/openmrs_data_model_1.9.0-concepts.png?version=1&modificationDate=1349587444000&api=v2)



## Notes on how OCL maps to OpenMRS concept data
* Record identifiers
  * OCL stores OMRS UUIDs for all resources (excluding lookup tables) in an `external_id` field (e.g. `OMRS.concept.uuid` = `OCL.concept.external_id`)
  * OCL does not store "*_id" fields with the exception of `concept.concept_id`
* Lookup tables
  * Lookup fields are stored in full text rather than by identifier (e.g. OCL.concept.concept_class = 'Diagnosis')
  * Lookup tables from OMRS include: Concept Classes, Concept Datatypes, Map Types, Name Types, Locales
  * Lookup tables added by OCL include: Description Types, Source Types, Collection Types
* OCL does not import OMRS audit data ("date_changed", "date_created", "date_retired", "creator", "changed_by", "retired_by", "voided", "voided_on", "voided_by") - OCL stores its own `created_by`, `created_on`, `updated_by`, `updated_on`
* Mappings
  * Mappings defined in the OMRS `concept_reference_map` are stored as "external" mappings in OCL, meaning that the `from_concept` is a concept defined in OCL (e.g. in the CIEL dictionary), and the `to_concept` is not defined in OCL (e.g. a reference term in SNOMED-CT). However, the external source must still be defined in OCL.
  * Data in `concept_reference_term_map` is not imported, but these represent concepts from other sources, which in OCL actually exist (e.g. IHTSDO/SNOMED-CT is a source in OCL)
* OMRS Questions, Answers, Concept Sets
  * To OCL, these are simply different types of relationships between concepts, which are stored as mappings with specific map types.
  * These are stored as "internal" mappings, meaning that both the `from_concept` and `to_concept` are defined in OCL.
* Concept Names
  * Voided concept names are removed from the new concept version rather than being set as voided -- to reference a voided concept name in OCL, you must refer to the prior concept version
* Extras
  * OCL has the ability to store any additional fields as "extras" for any resource - OMRS `is_set` and all of the numeric type fields (e.g. minimum, maximum, etc.) are stored as extras in OCL (e.g. `OMRS.concept.is_set` = `OCL.concept.extras.is_set`)
* OCL does not store any information from these OMRS tables:
  * Drug Tables: drug, drug_ingredient
  * Concept Name Tags: concept_name_tag, concept_name_tag_map
  * Concept Proposal Tables: concept_proposal, concept_proposal_tag_map
  * Other: concept_word, concept_complex, concept_stop_word


## OpenMRS to OCL Maps
NOTE:
* Fields in plain text are directly copied from OpenMRS
* _Italicized fields_ are derived from the corresponding field in OpenMRS and are not directly copied
* **_Bolded and italicized fields_** are linked to the specified OpenMRS field via foreign key relationship
* `ALL CAPS CODE` represent the planned behavior but not yet implemented

| OpenMRS Table | OpenMRS Field | OCL Field | Notes |
|----|----|----|----|
| concept | concept_id | concept.id | |
| concept | uuid | concept.external_id | |
| concept | retired | concept.retired | |
| concept | datatype_id | _concept.datatype_ | Value stored in OCL is `concept_datatype.name` |
| concept | class_id | _concept.concept_class_ | Value stored in OCL is `concept_class.name` |
| concept | is_set | concept.extras.is_set | |
| concept | short_name, description, form_text, creator, date_created, version, changed_by, date_changed, retired_by, date_retired, retire_reason | IGNORED | |
| | | | |
| concept_name | uuid | concept.name.external_id |  |
| concept_name | name | concept.name.name |  |
| concept_name | locale | concept.name.locale |  |
| concept_name | concept_name_type | concept.name.name_type |  |
| concept_name | locale_preferred | concept.name.locale_preferred |  |
| concept_name | concept_id | FOREIGN KEY |  |
| concept_name | concept_name_id, creator, date_created, voided, voided_by, date_voided, void_reason | IGNORED |  |
| | | | |
| concept_description | uuid | concept.description.external_id |  |
| concept_description | description | concept.description.description |  |
| concept_description | locale | concept.description.locale |  |
| concept_description | locale_preferred | concept.description.locale_preferred |  |
| concept_description | concept_id | FOREIGN KEY |  |
| concept_description | concept_description_id, creator, date_created, changed_by, date_changed | IGNORED |  |
| | | | |
| concept_datatype | name | concept.datatype | NOTE: Full datatype metadata will be stored in its own OCL source titled OCL:Datatypes that in the future will populate datatype dropdowns |
| | | | |
| concept_class | name | concept.concept_class | NOTE: Full concept class metadata will be stored in its own OCL source titled OCL:Classes that in the future will populate concept class dropdowns |
| | | | |
| concept_map_type | name | mapping.map_type | NOTE: Full map type metadata will be stored in its own OCL source titled OCL:MapTypes that in the future will populate map type dropdowns |
| | | | |
| concept_reference_source | uuid | source.external_id |  |
| concept_reference_source | name | source.full_name<br> <br>_source.id_<br>_source.short_code_<br>_source.name_ | id, short_code, and name are all derived from the name in OMRS |
| concept_reference_source | description | source.description | |
| concept_reference_source | hl7_code | source.extras.hl7_code | |
| concept_reference_source | concept_source_id, creator, date_created, retired, retired_by, date_retired, retire_reason | IGNORED | |
| | | | |
| concept_reference_map | uuid | mapping.external_id | |
| concept_reference_map | concept_id | from_concept_code<br>_from_source_owner_<br>_from_source_owner_type_<br>_from_source_name_<br>_from_source_url_<br>_from_concept_name_<br>_from_concept_url_ | Refer to [[mappings]] documentation for more info |
| concept_reference_map | concept_reference_term_id | _to_source_owner_<br>_to_source_owner_type_<br>_to_source_name_<br>_to_source_url_<br>_to_concept_code_<br>_to_concept_name_ | Refer to [[mappings]] documentation for more info |
| concept_reference_map | concept_id, concept_reference_term_id, concept_map_type_id | FOREIGN_KEY |  |
| concept_reference_map | concept_map_id, creator, date_created, changed_by, date_changed | IGNORED |  |
| | | | |
| concept_reference_term | name | mapping.to_concept_name | |
| concept_reference_term | code | mapping.to_concept_code | |
| concept_reference_term | retired | mapping.retired | This is the current behavior, but look at discussion question below |
| concept_reference_term | concept_source_id | FOREIGN KEY | |
| concept_reference_term | concept_reference_term_id, version, description, creator, date_created, date_changed, changed_by, retired_by, date_retired, retire_reason, uuid | IGNORED | The `external_id` for an OCL mapping is populated using `concept_reference_map.external_id` |
| | | | |
| concept_set | concept_set | _mapping.from_concept_ | This defines the "from_concept" of an internal mapping with `map_type=concept_set` |
| concept_set | concept_id | _mapping.to_concept_ | This defines the "to_concept" of an internal mapping with `map_type=concept_set` |
| concept_set | uuid | mapping.external_id | |
| concept_set | concept_set_id, sort_weight, creator, date_created | IGNORED | |
| | | | |
| concept_answer | concept_id | _mapping.from_concept_ | This defines the "from_concept" (in this case, the "Question") of an internal mapping with `map_type=q_and_a` |
| concept_answer | answer_concept | _mapping.to_concept_ | This defines the "to_concept" (in this case, a "Linked Answer") of an internal mapping with `map_type=q_and_a` |
| concept_answer | uuid | mapping.external_id | |
| concept_answer | sort_weight, concept_answer_id, answer_drug, creator, date_created | IGNORED | |
| | | | |
| concept_numeric | hi_absolute | concept.extras.hi_absolute |  |
| concept_numeric | hi_critical | concept.extras.hi_critical |  |
| concept_numeric | hi_normal | concept.extras.hi_normal |  |
| concept_numeric | low_absolute | concept.extras.low_absolute |  |
| concept_numeric | low_critical | concept.extras.low_critical |  |
| concept_numeric | low_normal | concept.extras.low_normal |  |
| concept_numeric | units | concept.extras.units |  |
| concept_numeric | precise | concept.extras.precise |  |
| concept_numeric | display_precision | concept.extras.display_precision |  |
| concept_numeric | concept_id | IGNORED |  |



## Changes to OCL
For Discussion:
* OCL.mapping.retired points to OMRS.concept_reference_term.retired, but it should point to OMRS.concept_reference_map.retired even though that field doesn't exist. Retiring the term would be the equivalent of retiring the concept in the external source.
* OMRS.concept.short_name, description, and form_text are not used in OCL
* OMRS.concept_name_tag and OMRS.concept_name_tag_map do not appear to be in use by CIEL -- what's the plan for these?
* OMRS.concept_reference_map.description --> ? OCL.mapping.extras.description ?
* OMRS.concept_reference_map.version --> ? OCL.mapping.extras.version ?

Change:
* OCL.mapping.retired points to OMRS.concept_reference_term.retired, but it should point to OMRS.concept_reference_map.retired even though that field doesn't exist. Retiring the term would be the equivalent of retiring the concept in the external source.
* Add OMRS.concept_reference_source.hl7_code --> OCL.source.extras.hl7_code
