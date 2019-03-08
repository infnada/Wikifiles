---
title: Heartbeat
description: 
published: true
date: 2019-03-08T23:13:58.161Z
tags: 
---

> Reemplazar `&#123;&#123;` por `{{` .... ya que sino no me renderiza correctamente este HTML

```
version: "3.6"

services:

  # How to Tune Elastic Beats Performance: A Practical Example with Batch Size, Worker Count, and More
  # https://www.elastic.co/blog/how-to-tune-elastic-beats-performance-a-practical-example-with-batch-size-worker-count-and-more?blade=tw&hulk=social
  heartbeat:
    image: docker.elastic.co/beats/heartbeat:6.6.0
    # https://github.com/docker/swarmkit/issues/1951
    hostname: "&#123;&#123;.Node.Hostname}}-heartbeat"
    # Need to override user so we can access the log files, and docker.sock
    user: root
    networks:
      - net_internal_web_gateway
    configs:
      - source: heartbeat.yml
        target: /usr/share/heartbeat/heartbeat.yml
    volumes:
      - heartbeat:/usr/share/heartbeat/data
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - ELASTICSEARCH_HOST=https://elasticsearch.gp.local # Set you elastic domain
      - KIBANA_HOST=https://kibana.gp.local # Set you kibana domain
      - ELASTICSEARCH_USERNAME=YOUR_ELASTIC_USERNAME
      - ELASTICSEARCH_PASSWORD=YOUR_ELASTIC_PASSWORD
    # disable strict permission checks
    command: ["--strict.perms=false"]
    deploy:
      placement:
        constraints:
          - node.platform.os == linux
          - node.role == worker
          - node.labels.tier == T2

networks:
  net_internal_web_gateway:
    external: true

volumes:
  heartbeat:

configs:
  heartbeat.yml:
    external: true
```

# Configs

- heartbeat.yml

```
################### Heartbeat Configuration Example #########################

# This file is a full configuration example documenting all non-deprecated
# options in comments. For a shorter configuration example, that contains
# only some common options, please see heartbeat.yml in the same directory.
#
# You can find the full configuration reference here:
# https://www.elastic.co/guide/en/beats/heartbeat/index.html

############################# Heartbeat ######################################


# Configure monitors
heartbeat.monitors:
- type: icmp # monitor type `icmp` (requires root) uses ICMP Echo Request to ping
             # configured hosts

  # Monitor name used for job name and document type.
  #name: icmp

  # Enable/Disable monitor
  #enabled: true

  # Configure task schedule using cron-like syntax
  schedule: '*/5 * * * * * *' # exactly every 5 seconds like 10:00:00, 10:00:05, ...

  # List of hosts to ping
  hosts: ["8.8.8.8", "8.8.4.4"]

  # Configure IP protocol types to ping on if hostnames are configured.
  # Ping all resolvable IPs if `mode` is `all`, or only one IP if `mode` is `any`.
  ipv4: true
  ipv6: true
  mode: any

  # Total running time per ping test.
  timeout: 16s

  # Waiting duration until another ICMP Echo Request is emitted.
  wait: 1s

  # The tags of the monitors are included in their own field with each
  # transaction published. Tags make it easy to group servers by different
  # logical properties.
  tags: ["docker-hosts"]

  # Optional fields that you can specify to add additional information to the
  # monitor output. Fields can be scalar values, arrays, dictionaries, or any nested
  # combination of these.
  #fields:
  #  env: staging

  # If this option is set to true, the custom fields are stored as top-level
  # fields in the output document instead of being grouped under a fields
  # sub-dictionary. Default is false.
  #fields_under_root: false

  # NOTE: THIS FEATURE IS DEPRECATED AND WILL BE REMOVED IN A FUTURE RELEASE
  # Configure file json file to be watched for changes to the monitor:
  #watch.poll_file:
    # Path to check for updates.
    #path:

    # Interval between file file changed checks.
    #interval: 5s

- type: tcp # monitor type `tcp`. Connect via TCP and optionally verify endpoint
            # by sending/receiving a custom payload

  # Monitor name used for job name and document type
  #name: tcp

  # Enable/Disable monitor
  #enabled: true

  # Configure task schedule
  schedule: '@every 5s' # every 5 seconds from start of beat

  # configure hosts to ping.
  # Entries can be:
  #   - plain host name or IP like `localhost`:
  #       Requires ports configs to be checked. If ssl is configured,
  #       a SSL/TLS based connection will be established. Otherwise plain tcp connection
  #       will be established
  #   - hostname + port like `localhost:12345`:
  #       Connect to port on given host. If ssl is configured,
  #       a SSL/TLS based connection will be established. Otherwise plain tcp connection
  #       will be established
  #   - full url syntax. `scheme://<host>:[port]`. The `<scheme>` can be one of
  #     `tcp`, `plain`, `ssl` and `tls`. If `tcp`, `plain` is configured, a plain
  #     tcp connection will be established, even if ssl is configured.
  #     Using `tls`/`ssl`, an SSL connection is established. If no ssl is configured,
  #     system defaults will be used (not supported on windows).
  #     If `port` is missing in url, the ports setting is required.
  hosts: ["elasticsearch:9200"]

  # Configure IP protocol types to ping on if hostnames are configured.
  # Ping all resolvable IPs if `mode` is `all`, or only one IP if `mode` is `any`.
  ipv4: true
  ipv6: true
  mode: any

  # List of ports to ping if host does not contain a port number
  # ports: [80, 9200, 5044]

  # Total test connection and data exchange timeout
  #timeout: 16s

  # Optional payload string to send to remote and expected answer. If none is
  # configured, the endpoint is expected to be up if connection attempt was
  # successful. If only `send_string` is configured, any response will be
  # accepted as ok. If only `receive_string` is configured, no payload will be
  # send, but client expects to receive expected payload on connect.
  #check:
    #send: ''
    #receive: ''

  # SOCKS5 proxy url
  # proxy_url: ''

  # Resolve hostnames locally instead on SOCKS5 server:
  #proxy_use_local_resolver: false

  # TLS/SSL connection settings:
  #ssl:
    # Certificate Authorities
    #certificate_authorities: ['']

    # Required TLS protocols
    #supported_protocols: ["TLSv1.0", "TLSv1.1", "TLSv1.2"]

  # NOTE: THIS FEATURE IS DEPRECATED AND WILL BE REMOVED IN A FUTURE RELEASE
  # Configure file json file to be watched for changes to the monitor:
  #watch.poll_file:
    # Path to check for updates.
    #path:

    # Interval between file file changed checks.
    #interval: 5s


- type: http # monitor type `http`. Connect via HTTP an optionally verify response

  # Monitor name used for job name and document type
  #name: http

  # Enable/Disable monitor
  #enabled: true

  # Configure task schedule
  schedule: '@every 5s' # every 5 seconds from start of beat

  # Configure URLs to ping
  urls: ["https://portainer.gp.local", "https://swarmpit.gp.local", "https://gitlab.gp.local", "https://traefik.gp.local", "https://grafana.gp.local", "https://kibana.gp.local", "https://prometheus.gp.local", "https://consul.gp.local", "https://weavescope.gp.local", "https://netdata.gp.local"]

  # Configure IP protocol types to ping on if hostnames are configured.
  # Ping all resolvable IPs if `mode` is `all`, or only one IP if `mode` is `any`.
  ipv4: true
  ipv6: true
  mode: any

  ssl.verification_mode: none

  # Configure file json file to be watched for changes to the monitor:
  #watch.poll_file:
    # Path to check for updates.
    #path:

    # Interval between file file changed checks.
    #interval: 5s

  # Optional HTTP proxy url.
  #proxy_url: ''

  # Total test connection and data exchange timeout
  #timeout: 16s

  # Optional Authentication Credentials
  #username: ''
  #password: ''

  # TLS/SSL connection settings for use with HTTPS endpoint. If not configured
  # system defaults will be used.
  #ssl:
    # Certificate Authorities
    #certificate_authorities: ['']

    # Required TLS protocols
    #supported_protocols: ["TLSv1.0", "TLSv1.1", "TLSv1.2"]

  # Request settings:
  #check.request:
    # Configure HTTP method to use. Only 'HEAD', 'GET' and 'POST' methods are allowed.
    #method: "GET"

    # Dictionary of additional HTTP headers to send:
    #headers:

    # Optional request body content
    #body:

  # Expected response settings
  #check.response:
    # Expected status code. If not configured or set to 0 any status code not
    # being 404 is accepted.
    #status: 0

    # Required response headers.
    #headers:

    # Required response contents.
    #body:

    # Parses the body as JSON, then checks against the given condition expression
    #json:
    #- description: Explanation of what the check does
    #  condition:
    #    equals:
    #      myField: expectedValue


  # NOTE: THIS FEATURE IS DEPRECATED AND WILL BE REMOVED IN A FUTURE RELEASE
  # Configure file json file to be watched for changes to the monitor:
  #watch.poll_file:
    # Path to check for updates.
    #path:

    # Interval between file file changed checks.
    #interval: 5s


heartbeat.scheduler:
  # Limit number of concurrent tasks executed by heartbeat. The task limit if
  # disabled if set to 0. The default is 0.
  #limit: 0

  # Set the scheduler it's timezone
  #location: ''

#================================ Processors ===================================
processors:
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
  - add_locale:
      format: offset
  - add_host_metadata:
      netinfo.enabled: true

#-------------------------- Elasticsearch output -------------------------------
output.elasticsearch:
  hosts: ["https://elasticsearch.gp.local:443"] # Set you elasticsearch domain
  username: YOUR_ELASTIC_USERNAME
  password: YOUR_ELASTIC_PASSWORD
  ssl:
    verification_mode: none

#============================== Dashboards =====================================
setup.dashboards:
  enabled: true

#============================== Kibana =====================================
setup.kibana:
  host: "https://kibana.gp.local:443" # Set you kibana domain
  username: YOUR_ELASTIC_USERNAME
  password: YOUR_ELASTIC_PASSWORD
  ssl:
    verification_mode: none

#================================ Logging ======================================
logging.to_files: false

#============================== Xpack Monitoring =====================================
xpack.monitoring:
   enabled: true
   elasticsearch:
```