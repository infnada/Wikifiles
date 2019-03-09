---
title: TIG
description: Telegraf + Influx + Grafana
published: true
date: 2019-03-09T16:50:54.046Z
tags: 
---

> Basado en Docker Stack

- https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Telegraf
- https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/InfluxDB
- https://wiki.isartnavarro.io/Containerisation/Docker/Stacks/Grafana

---

- Crear base de datos

`$ curl -i -XPOST http://telegraf_host:8086/query --data-urlencode "q=CREATE DATABASE telegraf"`

- Algunos querys TSQL para Stacks de Docker

```
$host = show tag values with key = "host"
$stack = show tag values with key = "com.docker.stack.namespace"
$service = show tag values with key = "com.docker.swarm.service.name" WHERE "com.docker.stack.namespace" =~ /^$stack/
$container = show tag values with key = "container_name" WHERE "host" =~ /^$host$/ AND "com.docker.stack.namespace" =~ /^$stack/  AND "com.docker.swarm.service.name" =~ /^$service/
```