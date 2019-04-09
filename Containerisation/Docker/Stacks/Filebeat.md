---
title: Filebeat
description: Filebeat
published: true
date: 2019-03-09T18:23:29.448Z
tags: 
---

> Reemplazar `&#123;&#123;` por `{{` .... ya que sino no me renderiza correctamente este HTML

```yaml
version: "3.6"

services:

  # How to Tune Elastic Beats Performance: A Practical Example with Batch Size, Worker Count, and More
  # https://www.elastic.co/blog/how-to-tune-elastic-beats-performance-a-practical-example-with-batch-size-worker-count-and-more?blade=tw&hulk=social
  filebeat:
    image: docker.elastic.co/beats/filebeat:6.6.0
    # https://github.com/docker/swarmkit/issues/1951
    hostname: "&#123;&#123;.Node.Hostname}}-filebeat"
    # Need to override user so we can access the log files, and docker.sock
    user: root
    networks:
      - net_internal_web_gateway
    configs:
      - source: filebeat.yml
        target: /usr/share/filebeat/filebeat.yml
    volumes:
      - filebeat:/usr/share/filebeat/data
      - /var/run/docker.sock:/var/run/docker.sock
      # This is needed for filebeat to load container log path as specified in filebeat.yml
      - /var/lib/docker/containers/:/var/lib/docker/containers/:ro

      # # This is needed for filebeat to load jenkins build log path as specified in filebeat.yml
      # - /var/lib/docker/volumes/jenkins_home/_data/jobs/:/var/lib/docker/volumes/jenkins_home/_data/jobs/:ro

      # This is needed for filebeat to load logs for system and auth modules
      - /var/log/:/var/log/:ro
      # This is needed for filebeat to load logs for auditd module
      # - /var/log/audit/:/var/log/audit/:ro
    environment:
      - ELASTICSEARCH_HOST=https://elasticsearch.gp.local # Set you elastic domain
      - KIBANA_HOST=https://kibana.gp.local # Set you kibana domain
      - ELASTICSEARCH_USERNAME=YOUR_ELASTIC_USERNAME
      - ELASTICSEARCH_PASSWORD=YOUR_ELASTIC_PASSWORD
    # disable strict permission checks
    command: ["--strict.perms=false"]
    deploy:
      mode: global

networks:
  net_internal_web_gateway:
    external: true

volumes:
  filebeat:

configs:
  filebeat.yml:
    external: true
```

# Configs

- filebeat.yml

```yaml
# https://github.com/elastic/beats/blob/master/filebeat/filebeat.reference.yml

filebeat.modules:
  - module: system
    syslog:
      enabled: true
    auth:
      enabled: true
  - module: auditd
    log:
      # Does not look like Auditd is supported in Alpine linux: https://github.com/linuxkit/linuxkit/issues/52
      enabled: false

filebeat.inputs:
  - type: docker
    enabled: true
    containers:
      stream: all # can be all, stdout or stderr
      ids:
        - '*'
    # exclude_lines: ["^\\s+[\\-`('.|_]"]  # drop asciiart lines
    # multiline.pattern: "^\t|^[[:space:]]+(at|...)|^Caused by:"
    # multiline.match: after

#========================== Filebeat autodiscover ==============================
# See this URL on how to run Apache2 Filebeat module: # https://www.elastic.co/guide/en/beats/filebeat/current/running-on-docker.html
filebeat.autodiscover:
  providers:
    - type: docker
      # https://www.elastic.co/guide/en/beats/filebeat/current/configuration-autodiscover-hints.html
      # This URL alos contains instructions on multi-line logs
      hints.enabled: true

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
  hosts: ["https://elasticsearch.gp.local:443"] # Set you elasticsearch domain
  username: YOUR_ELASTIC_USERNAME
  password: YOUR_ELASTIC_PASSWORD
  ssl:
    verification_mode: none

#============================== Dashboards =====================================
setup.dashboards:
  enabled: true

#============================== Kibana =========================================
setup.kibana:
  host: "https://kibana.gp.local:443" # Set you kibana domain
  username: YOUR_ELASTIC_USERNAME
  password: YOUR_ELASTIC_PASSWORD
  ssl:
    verification_mode: none

#============================== Xpack Monitoring ===============================
xpack.monitoring:
  enabled: true
  elasticsearch:
```
