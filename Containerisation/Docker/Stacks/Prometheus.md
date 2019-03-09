---
title: Prometheus
description: 
published: true
date: 2019-03-09T18:24:49.468Z
tags: 
---

```yaml
version: "3.6"

services:
  prometheus:
    image: "prom/prometheus:latest"
    user: root
    command: "--config.file=/prometheus.yml --storage.tsdb.path /data"
    volumes:
      - "prometheus_data:/data"
    configs:
      - prometheus.yml
    networks:
      - net_prometheus
      - net_internal_web_gateway
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.port=9090"
        - "traefik.frontend.rule=Host:prometheus.gp.local" # Set your domain
        - "traefik.docker.network=net_internal_web_gateway"
        - "traefik.backend=prometheus"
      placement:
        constraints:
          - node.platform.os == linux
          - node.role == worker
          - node.labels.tier == T2
      resources:
        limits:
          cpus: '3'
          memory: 3G
      restart_policy:
        condition: on-failure

configs:
  prometheus.yml:
    external: true

volumes:
  prometheus_data:
    driver: local
    driver_opts:
      type: nfs
      o: addr=111.111.111.111,nfsvers=4.1,nolock,soft,rw
      device: ":/prometheus_data"

networks:
  net_prometheus:
    driver: overlay
    external: true
  net_internal_web_gateway:
    driver: overlay
    external: true
```

# Configs

- prometheus.yml

```yaml
global:
  scrape_interval:     15s
  evaluation_interval: 15s

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
    monitor: 'prometheus-stack-monitor'

# Load and evaluate rules in this file every 'evaluation_interval' seconds.
rule_files:
# - "first.rules"
# - "second.rules"

scrape_configs:

  - job_name: 'prometheus'

    scrape_interval: 5s
    scrape_timeout: 5s

    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'gitlab-process'
    scrape_interval: 30s
    scrape_timeout: 10s
    metrics_path: '/process'
    static_configs:
      - targets: ['gitlab:9168']
  - job_name: 'gitlab-database'
    scrape_interval: 30s
    scrape_timeout: 10s
    metrics_path: '/database'
    static_configs:
      - targets: ['gitlab:9168']
  - job_name: 'gitlab-sidekiq'
    scrape_interval: 30s
    scrape_timeout: 10s
    metrics_path: '/sidekiq'
    static_configs:
      - targets: ['gitlab:9168']
  #XXX Broken, see: https://gotfix.com/docker/gitlab/issues/57
  # - job_name: 'gitlab-git'
  #   scrape_interval: 30s
  #   scrape_timeout: 10s
  #   metrics_path: '/git'
  #   static_configs:
  #     - targets: ['gitlab:9168']
  # - job_name: 'gitlab-git-process'
  #   scrape_interval: 30s
  #   scrape_timeout: 10s
  #   metrics_path: '/git_process'
  #   static_configs:
  #     - targets: ['gitlab:9168']
  - job_name: 'gitlab-pages'
    scrape_interval: 30s
    scrape_timeout: 10s
    metrics_path: '/pages'
    static_configs:
      - targets: ['gitlab:9235']
  - job_name: 'gitaly'
    scrape_interval: 5s
    scrape_timeout: 5s
    metrics_path: '/metrics'
    static_configs:
      - targets: ['gitlab:9236']
```