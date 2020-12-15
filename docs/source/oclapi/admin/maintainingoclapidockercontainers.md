# Maintaining OCL API's docker containers
## Overview

OclApi consists of the following containers:
   - ocl_api
   - ocl_mongo
   - ocl_flower
   - ocl_solr
   - ocl_worker
   - ocl_redis

When things go wrong, there are a number of ways to recycle some or all of the containers

## 1. Restarting the application through build pipeline

OCL API pipeline has stages named "QA deploy", "Staging Deploy" and "Production Deploy". These re deploy a certain version of the code and restart all container except ocl_mongo and ocl_solr.

This is the safest and most non-obtrusive, therefore the recommended way of restarting the OCLAPI application. However, further steps might be needed at times

## 2. Restarting the application using docker-compose commands

Though not recommended, this mode of operation needs SSH access to the server. Once SSH'd in:
   1. navigate to ```oclapi/django-nonrel/ocl```
   2. run ```docker-compose down```
   3. Redeploy using the pipeline, or run ``` deploy/start_docker.sh```

## 3. Hard recycling containers

Sometimes, docker containers mentioned hang and need to be recycled hard. In order to do this:

Go to `oclapi/django-nonrel/ocl` and run `docker-compose down` command. If it didn't work, use these steps.

   1. Run `docker ps` to find the container ID.
   2. To terminate the container, use the following command:

   ```sh
   docker rm -fv CONTAINER_ID
   ```

> The `-f` flag is short for `--force=false`, which forces the removal of a running container. The `-v` flag is
> short for `--volumes=false`, which removes the volumes associated with the container. You can use the long or
> short version of the flags.
   3. Redeploy using the pipeline, or run ``` deploy/start_docker.sh```

## 4. Completely restart docker

Sometimes docker just needs to be restarted completely. For example, if it is not possible to close the "default network". These steps will restart docker:

* Just stop docker deamon ( sudo service docker stop )
* Start again ( sudo service docker start )
* Go to API folder ( cd /root/oclapi/django-nonrel/ocl )
* Down dockers ( docker-compose down )
* Redeploy from GoCD


## 5. GoCD Docker

To start GoCD using docker:
```
docker start eloquent_brown
docker start go-agent
docker start go-agent-2
```
