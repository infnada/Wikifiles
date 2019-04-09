---
title: Packetbeat
description: Packetbeat
published: true
date: 2019-03-09T18:24:41.821Z
tags: 
---

> Reemplazar `&#123;&#123;` por `{{` .... ya que sino no me renderiza correctamente este HTML

```yaml
version: "3.6"

services:

  packetbeat:
    image: docker.elastic.co/beats/packetbeat:6.6.0
    # https://github.com/docker/swarmkit/issues/1951
    hostname: "&#123;&#123;.Node.Hostname}}-packetbeat"
    user: root
    networks:
      - net_internal_web_gateway
    configs:
      - source: packetbeat.yml
        target: /usr/share/packetbeat/packetbeat.yml
    volumes:
      - packetbeat:/usr/share/packetbeat/data
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - ELASTICSEARCH_HOST=https://elasticsearch.gp.local # Set your elastic domain
      - KIBANA_HOST=https://kibana.gp.local # Set your kibana domain
      - ELASTICSEARCH_USERNAME=YOUR_ELASTIC_USERNAME
      - ELASTICSEARCH_PASSWORD=YOUR_ELASTIC_PASSWORD
      # Eagerly waiting for Docker 19.06 release which will bring --privileged flag to Docker
      # Swarm Mode https://github.com/moby/moby/issues/24862#issuecomment-451594187
      # support for capabilities https://github.com/moby/moby/pull/38380
    cap_add:
      - NET_RAW
      - NET_ADMIN
    command: ["--strict.perms=false"]
    deploy:
      mode: global

networks:
  net_internal_web_gateway:
    external: true
    name: host

volumes:
  packetbeat:

configs:
  packetbeat.yml:
    external: true
```

# Configs

- packetbeat.yml

```yaml
#============================== Network device ================================
packetbeat.interfaces.device: any

#================================== Flows =====================================
packetbeat.flows:
  timeout: 30s
  period: 10s

#========================== Transaction protocols =============================
packetbeat.protocols:
  - type: http
    ports: [80, 8080, 5000]
    send_headers: true
    send_all_headers: true

  - type: tls
    ports: [443]
    send_certificates: false

#=========================== Monitored processes ==============================
packetbeat.procs:
  enabled: false
  monitored:
    - process: pgsql
      cmdline_grep: postgres

#================================ Processors ===================================
# For example, you can use the following processors to keep the fields that
# contain CPU load percentages, but remove the fields that contain CPU ticks
# values:
processors:
  - include_fields:
      fields: ["cpu"]
  - drop_fields:
      fields: ["cpu.user", "cpu.system"]
  # The following example drops the events that have the HTTP response code 200:
  - drop_event:
      when:
        equals:
          http.code: 200
  # The following example enriches each event with metadata from the cloud provider about the host machine.
  - add_docker_metadata:
      host: "unix:///var/run/docker.sock"
  - add_cloud_metadata: ~
  - add_locale: ~

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
