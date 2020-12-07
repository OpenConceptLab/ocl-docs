# Docker Networking and Security
## Purpose
This page shows what's done to block all inbound traffic except for __http__, __https__, __ssh__ and __icmp__ traffic. In other words, how access to mongo, solr etc is blocked from outside.

The major issue here is that docker overrides ```iptables``` rules and we need to do a workaround for that. What's below is just a workaround

## Steps

* First we need to install ```iptables-persistent``` on our servers

```sh
   apt-get install iptables-persistent
```

* Restart docker
```sh
service docker restart
```

* Block access to container ports from outside world. The commands below make sure that they are inserted on top of the DOCKER chain
```sh
iptables -I DOCKER 1 -p tcp ! -s 172.18.0.0/16 --dport 6379 -j DROP
iptables -I DOCKER 1 -p tcp ! -s 172.18.0.0/16 --dport 8983 -j DROP
iptables -I DOCKER 1 -p tcp ! -s 172.18.0.0/16 --dport 8000 -j DROP
iptables -I DOCKER 1 -p tcp ! -s 172.18.0.0/16 --dport 5555 -j DROP
iptables -I DOCKER 1 -p tcp ! -s 172.18.0.0/16 --dport 27017 -j DROP
```

* Finally save iptables
```sh
iptables-save
```
