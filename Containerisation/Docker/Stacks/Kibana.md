---
title: Kibana
description: 
published: true
date: 2019-03-09T18:23:58.080Z
tags: 
---

```yaml
version: '3.6'
services:
  kibana:
    image: docker.elastic.co/kibana/kibana:6.6.0
    environment:
      SERVER_NAME: kibana.gp.local # Set your domain
      ELASTICSEARCH_HOSTS: http://elasticsearch:9200  # Set your elastic address
      ELASTICSEARCH_USERNAME: YOUR_ELASTIC_USERNAME
      ELASTICSEARCH_PASSWORD: YOUR_ELASTIC_PASSWORD
    networks:
      - net_internal_web_gateway
      - net_elasticsearch
      - net_kibana
    deploy:
      placement:
        constraints:
          - node.platform.os == linux
          - node.role == worker
          - node.labels.tier == T2
      labels:
        - "traefik.enable=true"
        - "traefik.port=5601"
        - "traefik.frontend.rule=Host:kibana.gp.local"  # Set your domain
        - "traefik.docker.network=net_internal_web_gateway"
        - "traefik.backend=kibana"
      restart_policy:
        condition: on-failure

networks:
  net_elasticsearch:
    driver: overlay
    external: true
  net_kibana:
    driver: overlay
    external: true
  net_internal_web_gateway:
    driver: overlay
    external: true
```