---
title: Elasticsearch Cluster
description: 
published: true
date: 2019-03-08T23:00:44.685Z
tags: 
---

```
version: '3.6'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.6.0
    container_name: elasticsearch
    environment:
      - ELASTIC_PASSWORD=YOUR_ELASTIC_PASSWORD
      - xpack.security.enabled=true
      - xpack.monitoring.collection.enabled=true
      - xpack.license.self_generated.type=trial
      - node.name=elasticsearch
      - network.host=0.0.0.0
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=false
      - "ES_JAVA_OPTS=-Xms3g -Xmx3g"
      - "discovery.zen.ping.unicast.hosts=elasticsearch2"
    volumes:
      - esdata1:/usr/share/elasticsearch/data
      - eslog1:/var/log/elasticsearch
    networks:
      - net_internal_web_gateway
      - net_elasticsearch
    deploy:
      placement:
        constraints:
          - node.platform.os == linux
          - node.role == worker
          - node.labels.tier == T2
      labels:
        - "traefik.enable=true"
        - "traefik.port=9200"
        - "traefik.frontend.rule=Host:elasticsearch.gp.local" # Set you domain
        - "traefik.docker.network=net_internal_web_gateway"
        - "traefik.backend=elasticsearch"
      restart_policy:
        condition: on-failure

  elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.6.0
    container_name: elasticsearch2
    environment:
      - ELASTIC_PASSWORD=YOUR_ELASTIC_PASSWORD
      - xpack.security.enabled=true
      - xpack.monitoring.collection.enabled=true
      - xpack.license.self_generated.type=trial
      - node.name=elasticsearch2
      - network.host=0.0.0.0
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=false
      - "ES_JAVA_OPTS=-Xms3g -Xmx3g"
      - "discovery.zen.ping.unicast.hosts=elasticsearch"
    volumes:
      - esdata2:/usr/share/elasticsearch/data
      - eslog2:/var/log/elasticsearch
    networks:
      - net_internal_web_gateway
      - net_elasticsearch
    deploy:
      placement:
        constraints:
          - node.platform.os == linux
          - node.role == worker
          - node.labels.tier == T2
      labels:
        - "traefik.enable=true"
        - "traefik.port=9200"
        - "traefik.frontend.rule=Host:elasticsearch.gp.local" # Set you domain
        - "traefik.docker.network=net_internal_web_gateway"
        - "traefik.backend=elasticsearch"
      restart_policy:
        condition: on-failure

volumes:
  esdata1:
    driver: local
    driver_opts:
      type: nfs
      o: addr=111.111.111.111,nfsvers=4.1,nolock,soft,rw
      device: ":/esdata1"
  esdata2:
    driver: local
    driver_opts:
      type: nfs
      o: addr=111.111.111.111,nfsvers=4.1,nolock,soft,rw
      device: ":/esdata2"
  eslog1:
    driver: local
    driver_opts:
      type: nfs
      o: addr=111.111.111.111,nfsvers=4.1,nolock,soft,rw
      device: ":/eslog1"
  eslog2:
    driver: local
    driver_opts:
      type: nfs
      o: addr=111.111.111.111,nfsvers=4.1,nolock,soft,rw
      device: ":/eslog2"

networks:
  net_elasticsearch:
    driver: overlay
    external: true
  net_internal_web_gateway:
    driver: overlay
    external: true
```