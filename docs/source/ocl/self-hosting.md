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
