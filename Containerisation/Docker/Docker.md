---
title: Docker
description: 
published: true
date: 2019-03-08T20:39:26.800Z
tags: 
---

# Instalación de Docker

> https://docs.docker.com/install/linux/docker-ce/centos/

## Instalación en CentOS


### Utilizando repositorio

- Añadir utilidades básicas

`$ yum install -y yum-utils device-mapper-persistent-data lvm2`

- Añadir repositorio

`$ yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo`

- Instalación de paquetes Docker

`$ yum install docker-ce docker-ce-cli containerd.io`

### Utilizando script

```
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```

### Configurar, iniciar y comprobar

- Añadir tu usuario al grupo `docker` para usar Docker sin ser `sudo`

`$ usermod -aG docker your-user`

- Configurar logs
    - Por defecto Docker loguea el output de los contenedores en archivos en formato JSON. En algunos casos puede que estos archivos incrementen muchíssimo su tamaño.
    - Debido a que se utilizan concentradores de logs (Elasticsearch, influxDB, Prometheus...), aconsejo bajar al "mínimo" el tamaño disponible para los logs locales en los nodos.
    - Esta opción tambien se puede configurar a nivel de servicio.
    
Servicio:

```
logging:
    driver: "json-file"
    options:
        max-size: "50m"
```

Global por nodo:

```
$vi /etc/docker/daemon.json
---------
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

- Arrancar Docker

`$ systemctl start docker`

- Comprobar que funciona correctamente lanzando un contenedor

`$ docker run hello-world`

# Comandos básicos

> https://docs.docker.com/engine/reference/commandline/docker/

- Arrancar contenedor en modo `Daemon`
    - `-d` modo Daemon
    - `-p` exponer puerto `8000` local al puerto `80`interno
    - `-v` montar una ubicación local a una ubicación interna
    - `nginx` imagen a utilizar

`$ docker run -d -p 8000:80 -v /ubicacion_local:/usr/share/nginx/html nginx`

> Aconsejable usar la opción `-P` para mapeos dinámicos de puerto. Se mapeará un puerto empezando por el 30.000 hacia el puerto expuesto 80.

- Arrancar contenedor en modo `Terminal Interactivo`
    - `-ti` modo Terminal Interactivo
    - `-rm` eliminar el contenedor al salir del Terminal Interactivo
    - `centos` imagen a utilizar
    - `--name` nombre que se va a usar para el contenedor
        
`$ docker run -it -rm centos --name MiCentos`

> Salir de un contenedor en modo Terminal Interactivo sin cerrarlo. `Control + P + Q`

- Ejecutar comandos en un contenedor arrancado
    - En este ejemplo ejecutamos una shell en modo Terminal Interactivo.

`$ docker exec -ti nombre_container /bin/bash`

- Parar un contenedor

`$ docker stop nombre_container`

- Borrar un contenedor
    - Con un `-f` paramos el contenedor si está arrancado

`$ docker rm nombre_container`

`$ docker ps -q | xargs docker rm -f`

- Inspeccionar contenedor (info)

`$ docker inspect nombre_container`

- Copiar datos a/de un contenedor

```
$ docker cp nombre_container:/ruta_container ruta_local
$ docker cp ruta_local nombre_container:/ruta_container
```

- Mostrar procesos de un contenedor

`$ docker top nombre_container`

- Mostrar logs de un contenedor
    - `-f` se utiliza como el comando `tail`

`$ docker logs nombre_contenedor`

- Mostrar estadísticas de Docker

`$ docker stats`

# Docker plugis

> https://docs.docker.com/engine/extend/legacy_plugins/

## Instalación de plugins para volumenes vSphere (VMDK)

> https://github.com/vmware/vsphere-storage-for-docker

- Después de las pruebas pertinentes **no recomiendo su uso** en producción.
- Es mucho más facil administrar volumenes, y sus datos, atacando directamente a cabina con NFS o crear tu propio HA NFS Active/Pasive.
- Ultima actualización del plugin (Github) hace casi 1 año.
---

- Este plugin te permite crear volumenes en Datastores de vSphere y montarlos en los nodos de Docker sobre ESXi.
- Cada volumen solo se puede montar sobre un unico nodo de Docker
- En este caso utilizamos el parámetro `"VDVS_SOCKET_GID=233"` donde `233` es el `gid` del grupo `docker` para CoreOS

`$ docker plugin install --grant-all-permissions --alias vsphere vmware/vsphere-storage-for-docker:latest "VDVS_SOCKET_GID=233"`

### Instalación de plugin para volumenes vSphere sobre múltiples nodos de Docker

> https://github.com/vmware/vsphere-storage-for-docker/blob/master/docs/external/vfile-plugin.md

- **No recomiendo su uso**.
- Este plugin actua por encima del plugin de vSphere anterior.
- Crea un contenedor intermedio, al cual monta el volumen original de vSphere y posteriormente comparte un recurso local por Samba con el usuario `root` a los demás nodos de Docker. Es muy difícil/imposible administrar permisos a los archivos del volumen.
---
- Permite utilizar un unico volumen de vSphere en multiple nodos de Docker.

`$ docker plugin install --grant-all-permissions --alias vfile vmware/vfile:latest VFILE_TIMEOUT_IN_SECOND=90 "VDVS_SOCKET_GID=233"`

# Volumenes

> https://docs.docker.com/storage/volumes/

- Mostrar todos los volumenes

`$ docker volume ls`

- Montar volumen

`$ docker run -ti -v nombre_volumen:ruta_mapeo contenedor`

- Montar volumen de vSphere a un nuevo contenedor

`$ docker run --rm -it -v VolumeName@NFSLOCATION:/mnt/myvol --name busybox busybox`

## Creación de volumenes

- Volumen local

`$ docker volume create nombre_volumen`

- Volumen vSphere

```
$ docker volume create --driver=vsphere --name=nombre_volumen -o size=10gb
$ docker volume create --driver=vfile --name=nombre_volumen_compartido -o size=10gb
```

- Volumen NFS

> Utilzar las optiones de montaje NFS `hard/soft` u otras cuando sea preciso

`$ docker volume create --driver local --opt type=nfs --opt o=addr=192.168.1.1,rw --opt device=:/path/to/dir nombre_volumen`

- Hay muchissimos drivers más

# Redes

> https://docs.docker.com/network/

- Mostrar todas la redes

`$ docker network ls`

## Creación de redes

- Red simple 

`$ docker network create --attachable nombre_red`

- Utilización del driver Overlay

`$ docker network create -d overlay nombre_red`
