# Importing OpenMRS Concept Dictionary
Process:
1. Set up `ocl_omrs` repository
2. Get CIEL SQL file
    * This is currently made available by Andy Kanter in a Dropbox folder
3. Import CIEL into MySQL (usually in your local MySQL instance), e.g.:
```
create database ciel_20171120
mysql -u uid -p ciel_20171120 < ciel_20171120.sql
```
4. Update `settings.py` with the Mysql database name, uid, and password
5. Verify that mapping sources exist in OCL
    * `env` options are "production", "staging", "qa", "demo""
    * `token` must be valid user token that has access to the environment
```
python manage.py extract_db --checksources --env=... --token=...
```
6. Resolve missing orgs and sources -- add missing orgs and sources to the OCL deploy scripts and push them to relevant OCL environments
7. Verify that all lookup values used in the OpenMRS concept dictionary (e.g. concept classes, datatypes, name types, etc.) are available in the target OCL environment -- add missing lookup values OCL deploy scripts and push to relevant OCL environments
8. Generate OCL-formatted JSON import files
```
python manage.py extract_db --org_id=CIEL --source_id=CIEL --raw -v0 --concepts > ciel_20171120_concepts.json
python manage.py extract_db --org_id=CIEL --source_id=CIEL --raw -v0 --mappings > ciel_20171120_mappings.json
```
9. Optionally, generate smaller sample import files:
```
python manage.py extract_db --org_id=CIEL --source_id=CIEL --raw -v0 --concept_limit=2000 --concepts > ciel_20171120_concepts_2k.json
python manage.py extract_db --org_id=CIEL --source_id=CIEL --raw -v0 --concept_limit=2000 --mappings > ciel_20171120_mappings_2k.json
```
10. Import into OCL -- because of the size of the CIEL dictionary, this process currently needs to be run by an OCL admin directly on the OCL servers
11. Create new CIEL repository version, e.g.:
`POST /orgs/CIEL/sources/CIEL/versions/`
```
{
    "id": "v2019-07-01",
    "released": true,
    "description": "CIEL release v2019-07-01",
}
```
12. Download export of new CIEL repository version (Note that you will have to wait until the source version has completed processing, which may take awhile due to its size) -- e.g.:
`GET /orgs/CIEL/sources/CIEL/versions/v2019-07-01/export/`
13. Verify that the new export matches the CIEL SQL file:
```
python manage.py validate_export --export=EXPORT_FILE_NAME [--ignore_retired_mappings] [-v[2]]
```
