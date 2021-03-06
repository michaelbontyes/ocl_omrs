# ocl_omrs

This django project has scripts that make it easier to work with OCL and OpenMRS:
* **extract_db** will generate a JSON file from an OpenMRS v1.11 concept dictionary formatted for import into OCL
* **import** submits a file for bulk import into OCL
* **validate_export** validates an OCL export file against an OpenMRS v1.11 concept dictionary

## extract_db: OpenMRS Database JSON Export
The extract_db script reads an OpenMRS v1.11 database and extracts the concept and mapping data as JSON formatted for import into OCL. This script is typically run on a local machine with MySQL installed.

1. Update settings.py with MySQL database settings for target OpenMRS concept dictionary.
   If you are trying to import a new CIEL dictionary, you can import it into MySQL like this:

    mysql -u root -p ciel_20200124 < openmrs_concepts_1.11.4_20200124.sql

2. Check sources in specified OCL environment:
```
python manage.py extract_db --check_sources --env=demo --token=[my-token-here]
```
3. Check sources and output as OCL-formatted bulk JSON:
   (Note: the import file generated is not specific to an environment)
```
    python manage.py extract_db --check_sources --env=demo --token=<my-token-here> --org_id=MyOrg --source_id=MySource --raw -v0 --concepts --mappings --format=bulk > my_ocl_bulk_import_file.json
```
4. Alternatively, create "old-style" OCL import scripts (separate for concept and mappings)
    designed to be run directly on OCL server:
```
python manage.py extract_db --org_id=MyOrg --source_id=MySource --raw -v0 --concepts > concepts.json
python manage.py extract_db --org_id=MyOrg --source_id=MySource --raw -v0 --mappings > mappings.json
```
The 'raw' option indicates that JSON should be formatted one record per line (JSON lines file)
instead of human-readable format.

Optionally restrict output to a single concept or a limited number of concepts with the `concept_id` or `concept_limit` paramters. For example, `--concept_id=5839` will only return the concept with an ID of 5839, or `--concept_limit=10` will only return the first 10 concept entries.

## Submit import using bulk import API
If using the bulk import API format (see step #3 above), then you can validate and submit your import file using the following commands:

1. Validate import file:
```
    python manage.py import --validate-only --filename=[filename-here]
```
2. Submit using bulk import API:
``` 
    python manage.py import --env=production --token=[my-token-here] --filename=[filename-here]
```

## validate_export: OCL Export Validation
This command compares OCL export files to an OpenMRS concept dictionary stored in MySql.
Usage:
```
python manage.py validate_export --export=EXPORT_FILE_NAME [--ignore_retired_mappings] [-v[2]]
```

## Design Notes
* OCL-OpenMRS Subscription Module does not handle the OpenMRS drug table, so it is ignored for now
* `models.py` was created partially by scanning the mySQL schema, and the fixed up by hand. Not all classes are fully mapped yet, as not all are used by the OCL-OpenMRS Subscription Module.
