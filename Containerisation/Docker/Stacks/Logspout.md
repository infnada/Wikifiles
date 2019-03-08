---
title: Logspout
description: 
published: true
date: 2019-03-08T23:16:53.048Z
tags: 
---

```
version: "3.6"

services:
  logspout:
    image: bekt/logspout-logstash
    environment:
      ROUTE_URIS: 'logstash://logstash:5000' # Set your logstash url
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - net_logstash
    deploy:
      mode: global
      restart_policy:
        condition: on-failure
        delay: 30s

networks:
  net_logstash:
    driver: overlay
    external: true
```