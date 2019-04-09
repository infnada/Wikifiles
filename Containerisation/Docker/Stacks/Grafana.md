---
title: Grafana
description: Grafana
published: true
date: 2019-03-09T18:23:36.301Z
tags: 
---

```yaml
version: "3.6"

services:
  grafana:
    image: "grafana/grafana:latest"
    user: root
    environment:
      GF_PATHS_CONFIG: "/grafana.ini"
    configs:
      - grafana.ini
    volumes:
      - "grafana_data:/data"
    networks:
      - net_grafana
      - net_redis
      - net_internal_web_gateway
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.port=3000"
        - "traefik.docker.network=net_internal_web_gateway"
        - "traefik.backend=grafana"
        - "traefik.frontend.rule=Host:grafana.gp.local"
      placement:
        constraints:
          - node.platform.os == linux
          - node.role == worker
          - node.labels.tier == T2
      restart_policy:
        condition: on-failure

volumes:
  grafana_data:
    driver: local
    driver_opts:
      type: nfs
      o: addr=111.111.111.111,nfsvers=4.1,nolock,soft,rw
      device: ":/grafana_data"

configs:
  grafana.ini:
    external: true

networks:
  net_grafana:
    driver: overlay
    external: true
  net_internal_web_gateway:
    driver: overlay
    external: true
  net_redis:
    driver: overlay
    external: true
```

# Configs

- grafana.ini

```ini
[database]
path = "/data/grafana.db"

[session]
provider = "redis"
provider_config = "addr=redis:6379,prefix=grafana:,password=TU_REDIS_SUPER_PASSWORD"
```
