---
title: Swarmpit
description: 
published: true
date: 2019-03-09T18:21:17.524Z
tags: 
---

```yaml
version: '3.3'

services:
  app:
    image: swarmpit/swarmpit:latest
    environment:
      - SWARMPIT_DB=http://db:5984
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      - net_swarmpit
      - net_internal_web_gateway
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.port=8080"
        - "traefik.docker.network=net_internal_web_gateway"
        - "traefik.backend=swarmpit"
        - "traefik.frontend.rule=Host:swarmpit.gp.local" # Set your domain
      resources:
        limits:
          cpus: '0.50'
          memory: 1024M
        reservations:
          cpus: '0.25'
          memory: 512M
      placement:
        constraints:
          - node.role == manager

  db:
    image: couchdb:2.3.0
    volumes:
      - crouchdb_data:/opt/couchdb/data
    networks:
      - net_swarmpit
    deploy:
      resources:
        limits:
          cpus: '0.30'
          memory: 512M
        reservations:
          cpus: '0.15'
          memory: 256M
      placement:
        constraints:
          - node.role == manager

  agent:
    image: swarmpit/agent:latest
    environment:
      - DOCKER_API_VERSION=1.35
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      - net_swarmpit
    deploy:
      mode: global
      resources:
        limits:
          cpus: '0.10'
          memory: 64M
        reservations:
          cpus: '0.05'
          memory: 32M

networks:
  net_swarmpit:
    driver: overlay
    attachable: true
  net_internal_web_gateway:
    driver: overlay
    external: true

volumes:
  crouchdb_data:
    driver: local
    driver_opts:
      type: nfs
      o: addr=111.111.111.111,nfsvers=4.1,nolock,soft,rw
      device: ":/crouchdb_data"

```