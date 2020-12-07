# How to check logs
## Overview

In order to view containers' logs, `docker logs` is the best bet to go.
Before viewing logs, running containers could be seen by running `docker ps` and `docker ps -a` if wished to view stopped containers as well

## Checking logs

### OCL_API

```sh
   docker logs <container name> --tail <number of lines to tail> -f # -f is for following
```

### OCL_WEB

```sh
   less /var/log/ocl/web_debug.log
```

or

```sh
   less /var/log/ocl/web_debug.log.`date +%Y-%m-%d`_*
```
