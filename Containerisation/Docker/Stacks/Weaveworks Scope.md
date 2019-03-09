---
title: Weaveworks Scope
description: 
published: true
date: 2019-03-09T18:22:46.288Z
tags: 
---

```yaml
version: '3.5'

services:
  app:
    image: weaveworks/scope
    command:
      - /home/weave/scope
      - --mode=app
    ports:
      - "4040:4040"
    networks:
      - net_internal_web_gateway
    deploy:
      mode: replicated
      placement:
        constraints:
          - "node.hostname==dBCNmanager01" # --------- MODIFY
      replicas: 1
      labels:
        - "traefik.enable=true"
        - "traefik.port=4040"
        - "traefik.frontend.rule=Host:weavescope.gp.local" # Set you domain
        - "traefik.docker.network=net_internal_web_gateway"
        - "traefik.backend=wavescope"

  probe-launcher:
    image: weaveworks/scope-swarm-launcher
    command:
      - /bin/sh
      - -c
      - |
        scope launch --mode=probe --probe-only --probe.docker.bridge=docker0 --probe.docker=true DOCKER_NODE_IP_WHERE_APP_SERVICE_IS_DEPLOYED:4040
        docker logs --follow weavescope
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    deploy:
      mode: global

  # since it launches a standalone container on each host, to clean up you need to:
  # (use case for having terraform output an ansible inventory)
  # docker stop $(docker ps -a | grep weavescope | awk '{print $1}') && docker rm $(docker ps -a | grep weavescope | awk '{print $1}')

networks:
  net_internal_web_gateway:
    external: true
```