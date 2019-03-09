---
title: Portainer
description: 
published: true
date: 2019-03-09T18:21:07.713Z
tags: 
---

- Portainer se queda inaccesible cuando un nodo del cluster no puede, por ejemplo, acceder a un volumen NFS o de vSphere... o vete a saber por cual motivo. Algo que nunca me ha pasado con Swarmpit.

```yaml
version: '3.7'

services:
  agent:
    image: portainer/agent
    environment:
      # REQUIRED: Should be equal to the service name prefixed by "tasks." when
      # deployed inside an overlay network
      AGENT_CLUSTER_ADDR: tasks.agent
      # AGENT_PORT: 9001
      # LOG_LEVEL: debug
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
    networks:
      - agent_network
    deploy:
      mode: global
      placement:
        constraints:
          - node.platform.os == linux
      restart_policy:
        condition: on-failure

  portainer:
    image: portainer/portainer
    command: -H tcp://tasks.agent:9001 --tlsskipverify
    ports:
      - 9000
    volumes:
      - portainer_data:/data
    networks:
      agent_network:
      net_internal_web_gateway:
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.port=9000"
        - "traefik.docker.network=net_internal_web_gateway"
        - "traefik.frontend.rule=Host:portainer.gp.local" # Set you domain
        - "traefik.frontend.priority=1"
        - "traefik.frontend.entryPoints=http,https"
        - "traefik.frontend.redirect.entryPoint=https"
        - "traefik.backend=portainer"
        - "traefik.backend.loadbalancer.swarm=true"
        - "traefik.backend.loadbalancer.method=drr"
        - "traefik.backend.loadbalancer.stickiness=true"
      mode: replicated
      replicas: 1
      placement:
        constraints:
          - node.role == manager
      resources:
        limits:
          cpus: '1'
          memory: 1G
      restart_policy:
        condition: on-failure

networks:
  agent_network:
    driver: overlay
    attachable: true
  net_internal_web_gateway:
    driver: overlay
    external: true

volumes:
  portainer_data:
    driver: local
    driver_opts:
      type: nfs
      o: addr=111.111.111.111,nfsvers=4.1,nolock,soft,rw
      device: ":/portainer_data"

```