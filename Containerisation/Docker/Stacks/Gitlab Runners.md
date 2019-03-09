---
title: Gitlab Runners
description: 
published: true
date: 2019-03-09T18:25:08.262Z
tags: 
---

```yaml
version: "3.6"

services:
  gitlab:
    image: "gitlab/gitlab-ce"
    volumes:
      - "gitlab_data:/var/opt/gitlab"
      - "gitlab_config:/etc/gitlab"
    ports:
      - "22:22"
    configs:
      - source: "gitlab.rb"
        target: "/etc/gitlab/gitlab.rb"
    secrets:
       - GITLAB_ROOT_PASSWORD
       - REDIS_MASTER_PASSWORD
       - POSTGRES_GITLAB_PASSWORD
    networks:
      - gitlab
      - net_internal_web_gateway
      - net_redis
      - net_postgres
      - net_prometheus
    deploy:
      placement:
        constraints:
          - node.platform.os == linux
          - node.role == worker
          - node.labels.tier == T2
      labels:
        - "traefik.enable=true"
        - "traefik.port=80"
        - "traefik.frontend.rule=Host:gitlab.gp.local, registry.gitlab.gp.local" # Set yout domains
        - "traefik.docker.network=net_internal_web_gateway"
        - "traefik.backend=gitlab"
      restart_policy:
        condition: on-failure

  gitlab-runner:
    image: gitlab/gitlab-runner:alpine
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - gitlab
    configs:
      - source: "gitlab_runner.toml"
        target: "/etc/gitlab-runner/config.toml"
      - source: "local_server.pem"
        target: "/etc/gitlab-runner/certs/gitlab.gp.local.crt"
    deploy:
      mode: replicated
      replicas: 3
      placement:
        constraints:
          - node.platform.os == linux
          - node.role == worker
          - node.labels.tier == T2
      restart_policy:
        condition: on-failure


volumes:
  gitlab_data:
    driver: local
    driver_opts:
      type: nfs
      o: addr=111.111.111.111,nfsvers=4.1,nolock,soft,rw
      device: ":/gitlab_data"
  gitlab_config:
    driver: local
    driver_opts:
      type: nfs
      o: addr=111.111.111.111,nfsvers=4.1,nolock,soft,rw
      device: ":/gitlab_config"

configs:
  gitlab.rb:
    external: true
  gitlab_runner.toml:
    external: true
  local_server.pem:
    external: true

secrets:
   GITLAB_ROOT_PASSWORD:
     external: true
   POSTGRES_GITLAB_PASSWORD:
     external: true
   REDIS_MASTER_PASSWORD:
     external: true

networks:
  gitlab:
  net_internal_web_gateway:
    driver: overlay
    external: true
  net_prometheus:
    driver: overlay
    external: true
  net_redis:
    driver: overlay
    external: true
  net_postgres:
    driver: overlay
    external: true
```

# Configs

- gitlab.rb

```ruby
# gitlab.rb

external_url 'https://gitlab.gp.local' # Set your domain
registry_external_url 'https://registry.gitlab.gp.local' # Set your domain
gitlab_rails['registry_host'] = "registry.gitlab.gp.local" # Set your domain

nginx['redirect_http_to_https'] = false
nginx['listen_port'] = '80'
nginx['listen_https'] = false
registry_nginx['listen_port'] = '80'
registry_nginx['listen_https'] = false

registry['enable'] = true
gitlab_rails['registry_enabled'] = true


gitlab_rails['initial_root_password'] = File.read('/run/secrets/GITLAB_ROOT_PASSWORD')

# Disable services
postgresql['enable'] = false
redis['enable'] = false
prometheus['enable'] = false
postgres_exporter['enable'] = false
redis_exporter['enable'] = false

# Postgres settings
gitlab_rails['db_adapter'] = "postgresql"
gitlab_rails['db_encoding'] = "unicode"

# database service will be named "postgres" in the stack
gitlab_rails['db_host'] = "tasks.postgres"
gitlab_rails['db_database'] = "gitlab"
gitlab_rails['db_username'] = "gitlab"
gitlab_rails['db_password'] = File.read('/run/secrets/POSTGRES_GITLAB_PASSWORD')

# Redis settings
# redis service will be named "redis" in the stack
gitlab_rails['redis_host'] = "tasks.redis"
gitlab_rails['redis_password'] = File.read('/run/secrets/REDIS_MASTER_PASSWORD')

# Prometheus exporters
node_exporter['listen_address'] = '0.0.0.0:9100'
gitlab_monitor['listen_address'] = '0.0.0.0'
gitaly['prometheus_listen_addr'] = "0.0.0.0:9236"
gitlab_workhorse['prometheus_listen_addr'] = "0.0.0.0:9229"

gitlab_rails['monitoring_whitelist'] = ['0.0.0.0/0', '10.0.16.0/24']
```

- gitlab_runner.toml

```toml
concurrent = 1

[[runners]]
  name = "docker"
  url = "http://gitlab/"
  token = "YOUR_RUNNER_TOKEN"
  limit = 0
  executor = "docker"
  builds_dir = "/root"
  [runners.docker]
    image = "docker:stable"
    privileged = true
    disable_entrypoint_overwrite = false
    disable_cache = false
    volumes = ["/cache"]
    [runners.docker.sysctls]
      "net.ipv4.ip_forward" = "1"
```

- local_server.pem

```
-----BEGIN CERTIFICATE-----
XXXXXXX
-----END CERTIFICATE-----
```

# Secrets

- Just some passwords