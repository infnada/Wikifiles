---
title: Traefik
description: Traefik
published: true
date: 2019-03-09T18:22:39.339Z
tags: 
---

> https://docs.traefik.io/user-guide/examples/
> Mejor utilizar `ports` en modo `host`

```yaml
version: "3.4"

services:

  traefik:
    image: traefik
    command:
      - "--api"
      - "--acme"
      - "--acme.storage=/etc/traefik/acme/acme.json"
      - "--acme.entryPoint=https"
      - "--acme.httpChallenge.entryPoint=http"
      - "--acme.onHostRule=true"
      - "--acme.onDemand=false"
      - "--acme.acmeLogging=true"
      - "--acme.email=TU_CORREO" # Set your email
      - "--docker"
      - "--docker.swarmmode=true"
      - "--docker.watch=true"
      - "--docker.exposedbydefault=false"
      - "--docker.domain=TU_DOMINIO" # Set your domain
    configs:
      - source: traefik.toml
        target: /etc/traefik/traefik.toml
      - source: local_server.key
        target: /local_server.key
      - source: local_server.pem
        target: /local_server.pem
    volumes:
      - traefik_acme:/etc/traefik/acme
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - net_internal_web_gateway
      - net_traefik
    ports:
      - 80:80
      - 443:443
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.port=8080"
        - "traefik.docker.network=net_internal_web_gateway"
        - "traefik.frontend.rule=Host:traefik.gp.local" # Set you domain
      mode: global
      placement:
        constraints:
          - node.role == manager
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure

networks:
  net_internal_web_gateway:
    driver: overlay
    external: true
  net_traefik:
    driver: overlay
    external: true

volumes:
  traefik_acme:
    driver: local
    driver_opts:
      type: nfs
      o: addr=111.111.111.111,nfsvers=4.1,nolock,soft,rw
      device: ":/traefik_acme"

configs:
  local_server.pem:
    external: true
  local_server.key:
    external: true
  traefik.toml:
    external: true
```

# Configs

- local_server.pem
```
-----BEGIN CERTIFICATE-----
XXXXXXX
-----END CERTIFICATE-----
```

- local_server.key <-- Esto debería de ser un `Secret`
```
-----BEGIN PRIVATE KEY-----
XXXXXXX
-----END PRIVATE KEY-----
```

- traefik.toml
```toml
defaultEntryPoints = ["http", "https"]

[entryPoints]
  [entryPoints.http]
  address = ":80"
    [entryPoints.http.redirect]
    entryPoint = "https"
  [entryPoints.https]
  address = ":443"
    [entryPoints.https.tls]
      [[entryPoints.https.tls.certificates]]
      certFile = "/local_server.pem"
      keyFile = "/local_server.key"
```
