---
title: Auditbeat
description: Auditbeat
published: true
date: 2019-04-09T13:54:20.043Z
tags: 
---

> Reemplazar `&#123;&#123;` por `{{` .... ya que sino no me renderiza correctamente este HTML

```yaml
version: "3.6"

# https://www.elastic.co/guide/en/beats/auditbeat/current/auditbeat-overview.html
# Does not look like Auditd is supported in Alpine linux: https://github.com/linuxkit/linuxkit/issues/52

services:

  auditbeat:
    image: docker.elastic.co/beats/auditbeat:6.6.0
    # https://github.com/docker/swarmkit/issues/1951
    hostname: "&#123;&#123;.Node.Hostname}}-auditbeat"
    # Need to override user so we can access the log files, and docker.sock
    user: root
    # https://www.elastic.co/guide/en/beats/auditbeat/current/running-on-docker.html#_special_requirements
    # PID and CAP_ADD options are ignored as they are Not yet available in swarm mode at the moment.
    # Eagerly waiting for Docker 19.06 release which will bring --privileged flag to Docker
    # Swarm Mode https://github.com/moby/moby/issues/24862#issuecomment-451594187
    # support for capabilities https://github.com/moby/moby/pull/38380
    pid: host
    cap_add:
      - AUDIT_CONTROL
      - AUDIT_READ
    networks:
      - net_internal_web_gateway
    configs:
      - source: auditbeat.yml
        target: /usr/share/auditbeat/auditbeat.yml
    volumes:
      - auditbeat:/usr/share/auditbeat/data
      - /var/log:/var/log:ro
    environment:
      - ELASTICSEARCH_HOST=https://elasticsearch.gp.local # Set your elastic domain
      - KIBANA_HOST=https://kibana.gp.local # Set your kibana domain
      - ELASTICSEARCH_USERNAME=YOUR_ELASTIC_USERNAME
      - ELASTICSEARCH_PASSWORD=YOUR_ELASTIC_PASSWORD
    command: ["--strict.perms=false"]
    deploy:
      mode: global

networks:
  net_internal_web_gateway:
    external: true
    name: host

volumes:
  auditbeat:

configs:
  auditbeat.yml:
    external: true
```

# Configs

- auditbeat.yml

```yaml
# https://github.com/elastic/beats/blob/master/filebeat/filebeat.reference.yml

auditbeat.modules:

  - module: auditd
    audit_rules: |
      -w /etc/passwd -p wa -k identity
      -a always,exit -F arch=b32 -S open,creat,truncate,ftruncate,openat,open_by_handle_at -F exit=-EPERM -k access
  - module: file_integrity
    paths:
      - /bin
      - /usr/bin
      - /sbin
      - /usr/sbin
      - /etc

  - module: system
    datasets:
      - host # General host information, e.g. uptime, IPs
      - user # User information
    period: 1m
    user.detect_password_changes: true

  - module: system
    datasets:
      - process # Started and stopped processes
      - socket  # Opened and closed sockets
    period: 1s

#================================ Processors ===================================
processors:
  - add_cloud_metadata: ~
# - add_docker_metadata: ~
# - add_locale: ~
# - add_host_metadata:
#     netinfo.enabled: true
# - add_process_metadata: ~

#========================== Elasticsearch output ===============================
output.elasticsearch:
  hosts: ["https://elasticsearch.gp.local:443"] # Set your elasticsearch domain
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