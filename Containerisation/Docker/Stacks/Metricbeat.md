---
title: Metricbeat
description: 
published: true
date: 2019-03-09T18:24:10.606Z
tags: 
---

> Reemplazar `&#123;&#123;` por `{{` .... ya que sino no me renderiza correctamente este HTML

```yaml
version: "3.6"

services:

  metricbeat:
    image: docker.elastic.co/beats/metricbeat:6.6.0
    # https://github.com/docker/swarmkit/issues/1951
    hostname: "&#123;&#123;.Node.Hostname}}-metricbeat"
    user: root
    networks:
      - net_internal_web_gateway
    configs:
      - source: metricbeat.yml
        target: /usr/share/metricbeat/metricbeat.yml
    volumes:
      - /proc:/hostfs/proc:ro
      - /sys/fs/cgroup:/hostfs/sys/fs/cgroup:ro
      - /:/hostfs:ro
      - /var/run/docker.sock:/var/run/docker.sock
      - metricbeat:/usr/share/metricbeat/data
    environment:
      - ELASTICSEARCH_HOST=https://elasticsearch.gp.local # Set your elastic domain
      - KIBANA_HOST=https://kibana.gp.local # Set your kibana domain
      - ELASTICSEARCH_USERNAME=YOUR_ELASTIC_USERNAME
      - ELASTICSEARCH_PASSWORD=YOUR_ELASTIC_PASSWORD
    # disable strict permission checks
    command: ["--strict.perms=false", "-system.hostfs=/hostfs"]
    deploy:
      mode: global

networks:
  net_internal_web_gateway:
    external: true
    # https://www.elastic.co/guide/en/beats/metricbeat/current/running-on-docker.html#monitoring-host
    name: host

volumes:
  metricbeat:

configs:
  metricbeat.yml:
    external: true
```

# Configs

- metricbeat.yml

```yaml
# https://github.com/elastic/beats/blob/master/metricbeat/metricbeat.reference.yml

#-------------------------------- Autodiscovery -------------------------------
# Autodiscover allows you to detect changes in the system and spawn new modules as they happen.
metricbeat.autodiscover:
  providers:
    - type: docker
      # https://www.elastic.co/guide/en/beats/metricbeat/current/configuration-autodiscover-hints.html
      hints.enabled: true

metricbeat.modules:
  #------------------------------- System Module -------------------------------
  - module: system
    metricsets: ["cpu", "load", "memory", "network", "process", "process_summary", "core", "diskio", "socket"]
    processes: ['.*']
    process.include_top_n:
      by_cpu: 5
      by_memory: 5
    period: 10s
    cpu.metrics:  ["percentages"]
    core.metrics: ["percentages"]

  - module: system
    period: 1m
    metricsets:
      - filesystem
      - fsstat
    processors:
      - drop_event.when.regexp:
          system.filesystem.mount_point: '^/(sys|cgroup|proc|dev|etc|host|lib)($|/)'

  - module: system
    period: 15m
    metricsets:
      - uptime

  #------------------------------- Docker Module -------------------------------
  - module: docker
    metricsets: ["container", "cpu", "diskio", "healthcheck", "info", "memory", "network"]
    hosts: ["unix:///var/run/docker.sock"]
    period: 10s

#================================ Processors ===================================
processors:
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
  - add_locale:
      format: offset
  - add_host_metadata:
      netinfo.enabled: true

#========================== Elasticsearch output ===============================
output.elasticsearch:
  hosts: ["https://elasticsearch.gp.local:443"] # Set your elastic domain
  username: YOUR_ELASTIC_USERNAME
  password: YOUR_ELASTIC_PASSWORD
  ssl:
    verification_mode: none

#============================== Dashboards =====================================
setup.dashboards:
  enabled: true

#============================== Kibana =========================================
setup.kibana:
  host: "https://kibana.gp.local:443" # Set your kibana domain
  username: YOUR_ELASTIC_USERNAME
  password: YOUR_ELASTIC_PASSWORD
  ssl:
    verification_mode: none

#============================== Xpack Monitoring ===============================
xpack.monitoring:
  enabled: true
  elasticsearch:
```