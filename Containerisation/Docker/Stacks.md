---
title: Stacks
description: Stacks
published: true
date: 2019-03-09T18:28:11.045Z
tags: 
---

> La mayoría de estos ejemplos utilizan volumenes por NFS y Traefik. Al mismo tiempo pueden contener constraints específicos para mis pruebas.

# Networks used

> Para un buen aislamiento, cada servicio debería conectar con Traefik desde una red diferente. En este caso siempre utilizamos `net_internal_web_gateway`

```sh
$ docker network create -d overlay --attachable net_postgres
$ docker network create -d overlay --attachable net_redis  
$ docker network create -d overlay --attachable net_consul
$ docker network create -d overlay --attachable net_traefik
$ docker network create -d overlay --attachable net_internal_web_gateway
$ docker network create -d overlay --attachable net_prometheus
$ docker network create -d overlay --attachable net_grafana
$ docker network create -d overlay --attachable net_elasticsearch
$ docker network create -d overlay --attachable net_logstash
$ docker network create -d overlay --attachable net_kibana
$ docker network create -d overlay --attachable net_mysql
```

# Domains used
```
https://portainer.gp.local/
https://swarmpit.gp.local
https://gitlab.gp.local
https://traefik.gp.local
https://grafana.gp.local
https://kibana.gp.local
https://prometheus.gp.local
https://consul.gp.local
https://weavescope.gp.local
https://netdata.gp.local
```

# Base
- [Consul](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Consul)
- [Istio](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Istio)
- [Portainer](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Portainer)
- [Swarmpit](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Swarmpit)
- [Traefik](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Traefik)
- [Traefik with Consul](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Traefik%20Consul)
- [Weaveworks Scope](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Weaveworks%20Scope)

# Monitoring
- [Auditbeat](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Auditbeat)
- [Elasticsearch Cluster](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Elasticsearch%20Cluster) - Not really monitoring
- [Filebeat](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Filebeat)
- [Grafana](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Grafana)
- [Heartbeat](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Heartbeat)
- [InfluxDB](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/InfluxDB) - Not really monitoring
- [Kibana](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Kibana)
- [Logspout](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Logspout)
- [Logstash](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Logstash)
- [Metricbeat](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Metricbeat)
- [Netdata Stream](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Netdata%20Stream)
- [Packetbeat](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Packetbeat)
- [Prometheus](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Prometheus)
- [Telegraf](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Telegraf)

# CI/CD
- [Gitlab with Runners](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Gitlab%20Runners)

# Services
- [MySQL Cluster](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/MySQL%20Cluster)
- [Custom PHP](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Custom%20PHP)
- [Postgres](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Postgres)
- [Redis Cluster](https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Redis%20Cluster)
