---
title: Netdata Stream
description: 
published: true
date: 2019-03-08T23:21:38.592Z
tags: 
---

> Reemplazar `&#123;&#123;` por `{{` .... ya que sino no me renderiza correctamente este HTML

```
version: '3.6'

services:
  netdata-client:
    image: netdata/netdata
    hostname: "&#123;&#123;.Node.Hostname}}"
    configs:
      - source: "netdata_slave_stream.conf"
        target: "/etc/netdata/stream.conf"
      - source: "netdata_slave_config.conf"
        target: "/etc/netdata/netdata.conf"
    cap_add:
      - SYS_PTRACE
    security_opt:
      - apparmor:unconfined
    environment:
      - PGID=999
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - net_netdata
    deploy:
      mode: global
      placement:
        constraints: [node.role == worker]

  netdata-central:
    image: netdata/netdata
    hostname: netdata-control
    configs:
      - source: "netdata_master_stream.conf"
        target: "/etc/netdata/stream.conf"
    cap_add:
      - SYS_PTRACE
    security_opt:
      - apparmor:unconfined
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - net_netdata
      - net_internal_web_gateway
    deploy:
      mode: replicated
      replicas: 1
      labels:
        - "traefik.enable=true"
        - "traefik.port=19999"
        - "traefik.docker.network=net_internal_web_gateway"
        - "traefik.backend=netdata"
        - "traefik.frontend.rule=Host:netdata.gp.local" # Set your domain
      placement:
        constraints: [node.role == manager]

networks:
  net_netdata:
    driver: overlay
    attachable: true
  net_internal_web_gateway:
    driver: overlay
    external: true

configs:
  netdata_master_stream.conf:
    external: true
  netdata_slave_stream.conf:
    external: true
  netdata_slave_config.conf:
    external: true
```

# Configs

- netdata_master_stream.conf

```
[11111111-2222-3333-4444-555555555555]
    # enable/disable this API key
    enabled = yes

    # one hour of data for each of the slaves
    default history = 3600

    # do not save slave metrics on disk
    default memory = ram

    # alarms checks, only while the slave is connected
    health enabled by default = auto
```

- netdata_slave_stream.conf

```
[stream]
    # stream metrics to another netdata
    enabled = yes

    # the IP and PORT of the master
    destination = netdata-central:19999

    # the API key to use
    api key = 11111111-2222-3333-4444-555555555555
```

- netdata_slave_config.conf

```
[global]
    # disable the local database
    memory mode = none

[health]
    # disable health checks
    enabled = no
```
