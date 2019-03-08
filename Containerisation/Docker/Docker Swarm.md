---
title: Docker Swarm
description: 
published: true
date: 2019-03-08T18:30:37.543Z
tags: 
---


> https://docs.docker.com/engine/swarm/

# Routing mesh

- Por defecto todos los puertos expuestos en un Swarm serán accesibles des de cualquier nodo del mismo.

> More about routing mesh https://docs.docker.com/engine/swarm/ingress/

# Puertos utilizados

- TCP port `2377` for cluster management communications.
- TCP and UDP port `7946` for communication among nodes.
- UDP port `4789` for overlay network traffic.

# Creación de Swarm

## Must

- Cómo cualquier servicio en cluster hay que prevenir el split-brain. Para desarollo se recomienda un mínimo de 3 nodos que hagan de manager y en **producción un mínimo de 5**.

> En caso de split-brain el cluster seguiría funcionado pero no se podría cambiar ninguna configuración del mismo hasta reestablecer el quorum.

## Inicialización

- Creación del primer miembro del Swarm

`$ docker swarm init`

### Creación de tokens

> Sólo válido des de un nodo manager

- Manager

`$ docker swarm join-token manager`

- Woker

`$ docker swarm join-token worker`

### Añadir nodos al swarm

- Sólo hay que ejecutar el output del comando `join-token` en el nodo a incluir al swarm.

# Comandos básicos

- Listar los nodos del Swarm

`$ docker node ls`

## Stack

- Lanzar stack

`$ docker stack deploy -c archivo.yaml nombre_stack`

- Listar los stacks

`$ docker stack ls`

- Listar los servicios

`$ docker service ls`

- Recargar servicio

`$ docker service update nombre_servicio`

- Escalar servicio

`$ docker service scale nombre_servicio=numero_replicas


# Stacks interesantes

- Portainer
- Swarmpit
- Consul
- Traefik