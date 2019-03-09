---
title: Docker Machine
description: 
published: true
date: 2019-03-09T17:05:35.895Z
tags: 
---

> https://docs.docker.com/machine/
> https://docs.docker.com/machine/get-started/

> **100$ free 2 months DigitalOcean referal -> https://m.do.co/c/2cb1a2a5d99b**

# Instalación

`$ base=https://github.com/docker/machine/releases/download/v0.16.0 &&  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/usr/local/bin/docker-machine &&  chmod +x /usr/local/bin/docker-machine`

# Ejemplos básicos

- DigitalOcean

`$ docker-machine create -d digitalocean --digitalocean-access-token=xxx --digitalocean-image=centos-7-x64 --digitalocean-region=lon1 --digitalocean-size "s-2vcpu-2gb" --digitalocean-private-networking --engine-label public=no db01`

- AWS EC2

https://docs.docker.com/machine/examples/aws/

- Desplegar Docker Swarm

https://docs.yugabyte.com/latest/deploy/docker-swarm/