# Switching to Maintenance Mode on Production Server

### Connect to the server via ssh

    ssh user@openconceptlab.org

### Change directory to OCL directory

    cd /var/www/openconceptlab

### Make a copy of the maintenance page file

    cp maintenance.html.original maintenance.html

Congrats you are in the maintenance mode.

## Switching to production mode
### Connect to the server via ssh

    ssh user@openconceptlab.org

### Change directory to OCL directory

    cd /var/www/openconceptlab

### Remove maintenance file

    rm maintenance.html

Congrats you are in the production mode.

## Notes

* We change the /etc/nginx/sites-enabled/default file to make this happen.
```
server {
  server_name api.openconceptlab.org www.api.openconceptlab.org;
  ..........
  location / {
    if (-f $document_root/maintenance.html) {
      return 503;
    }   
    ................
  }
  error_page 503 @maintenance;
  location @maintenance {
    rewrite ^(.*)$ /maintenance.html break;
  }
}
```
* You can find a maintenance file example from [ocl_web repo](https://github.com/OpenConceptLab/ocl_web/blob/master/docs/maintenance.html.original)
