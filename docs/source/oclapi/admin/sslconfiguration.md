# SSL Configuration
SSL certificates have been implemented on the staging and production environments for both the Web Interface (https://www.openconceptlab.org and https://staging.openconceptlab.org) and for API requests (https://api.staging.openconceptlab.org and https://api.openconceptlab.org).

These are free certificates which are valid only for 3 months. In order to renew the certificate once expired, there is a crontab entry in both staging and production (but not on showcase), which checks for certificate validity daily at 5:00 AM and renews the certificate if required.

These SSL certificates (https) are attached to both `ocl_web` and `oclapi`.

The free certificates are registered in the "letsencrypt" community: https://certbot.eff.org/#ubuntuxenial-nginx
