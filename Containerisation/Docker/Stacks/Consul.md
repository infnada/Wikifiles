---
title: Consul
description: 
published: true
date: 2019-03-08T22:39:25.557Z
tags: 
---

- Hay muchas maneras de deployear un Consul, pero esta es la que me ha dado menos problemas para reestablecer el quorum en caso de caida.

```
#https://github.com/hashicorp/docker-consul/issues/66
version: '3.6'

services:

  consul:
    image: consul
    environment:
      - CONSUL_BIND_INTERFACE=eth0
      - CONSUL_CLIENT_INTERFACE=eth0
      - leave_on_terminate=true
    ports:
      - target: 8500
        published: 8500
        mode: host
    volumes:
      - consul-data:/consul/data
    networks:
      net_consul:
        aliases:
          - consul.cluster
      net_internal_web_gateway:
    command: "agent -server -bootstrap-expect 3 -ui -client 0.0.0.0 -retry-join consul.cluster"
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.port=8500"
        - "traefik.docker.network=net_internal_web_gateway"
        - "traefik.backend=consul"
        - "traefik.frontend.rule=Host:consul.gp.local"
      mode: global
      endpoint_mode: dnsrr
      resources:
        limits:
          cpus: '1'
          memory: 1000M
      placement:
        constraints:
          - node.role == manager
      restart_policy:
        condition: on-failure

networks:
  net_internal_web_gateway:
    driver: overlay
    external: true
  net_consul:
    driver: overlay
    external: true

volumes:
  consul-data:
```