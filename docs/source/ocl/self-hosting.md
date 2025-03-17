## Self-hosting OCL Documentation

### Customizing OCL TermBrowser
https://github.com/OpenConceptLab/oclweb2/blob/master/src/common/serverConfigs.js#L76
* App logo
* Header (name)

### API Settings
* Email/SMTP provider (for sending account verification, forgot password, etc.)

### Connecting to cloud storage
OCL Online uses AWS S3 to store cached repository exports, but you can connect to any cloud storage service.
* Create your own an AWS S3 account
* Use the local temp folder
* Write your own custom service with 2 methods: (1) Upload and (2) Get URL

### Deploying with OpenSearch
OpenConceptLab does support OpenSearch in version 2.8.0. In order to run OCL with OpenSearch instead of ElasticSerach one needs to
apply changes from the opensearch branch. See https://github.com/OpenConceptLab/oclapi2/compare/master...opensearch

The changes do the following:
* Adjust docker-compose to use the opensearch image
* Use django-opensearch-dsl==0.5.1 and opensearch-dsl==2.1.0 instead of django-elasticsearch-dsl==7.3 as a client library
* Adjust code to use classes from opensearch_dsl instead of elasticserach_dsl
* Adjust index maintenance commands

We plan to do the migration to OpenSearch and upgrade to a newer version in OCL Online at some point. It is not yet determined the exact timeline for this migration to happen.


### Configuring API Rate Limiting (throttling)
By default Rate Limiting is turned off, enable it by making ENABLE_THROTTLING=true
See core/settings.py for throttling policies guest and lite.

* Existing users are by default assigned "lite" plan.
* Anonymous users are configured to use "guest" plan.
* Create a new policy by adding new User Throttle class(s) in core/common/throttling.py

For more info checkout https://www.django-rest-framework.org/api-guide/throttling/
