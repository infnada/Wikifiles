---
title: Kubernetes
description: Kubernetes
published: true
date: 2019-04-09T13:42:52.613Z
tags: 
---

> https://kubernetes.io/

# To read
> https://kubernetes.io/docs/reference/kubectl/cheatsheet/
> https://kubernetes.io/docs/concepts/
> Network comparison: https://chrislovecnm.com/kubernetes/cni/choosing-a-cni-provider/
> Serie de tutoriales de Oscar Mas sobre K8s: https://www.jorgedelacruz.es/2017/11/28/kubernetes-introduccion-kubernetes/
> HELM https://www.digitalocean.com/community/tutorials/an-introduction-to-helm-the-package-manager-for-kubernetes
> CNAB & Duffle: https://github.com/deislabs/duffle/tree/master/docs
> **Istio: https://istio.io/docs/setup/kubernetes/quick-start/**
> Deployment Strategies in K8s (hay más maneras): https://container-solutions.com/kubernetes-deployment-strategies/

> https://hub.docker.com/editions/enterprise/docker-ee-trial/trial

# Puertos utilizados

### Master node(s)

| Protocol | Direction | Port Range | Purpose                 | Used By                   |
|----------|-----------|------------|-------------------------|---------------------------|
| TCP      | Inbound   | 6443*      | Kubernetes API server   | All                       |
| TCP      | Inbound   | 2379-2380  | etcd server client API  | kube-apiserver, etcd      |
| TCP      | Inbound   | 10250      | Kubelet API             | Self, Control plane       |
| TCP      | Inbound   | 10251      | kube-scheduler          | Self                      |
| TCP      | Inbound   | 10252      | kube-controller-manager | Self                      |

### Worker node(s)

| Protocol | Direction | Port Range  | Purpose               | Used By                 |
|----------|-----------|-------------|-----------------------|-------------------------|
| TCP      | Inbound   | 10250       | Kubelet API           | Self, Control plane     |
| TCP      | Inbound   | 30000-32767 | NodePort Services**   | All                     |

# Requerimientos para Kubernetes utilizando kubeadm

> https://kubernetes.io/docs/setup/independent/install-kubeadm/

### Install CNI plugins (required for most pod network):
```sh
$ CNI_VERSION="v0.6.0"
$ mkdir -p /opt/cni/bin
$ curl -L "https://github.com/containernetworking/plugins/releases/download/${CNI_VERSION}/cni-plugins-amd64-${CNI_VERSION}.tgz" | tar -C /opt/cni/bin -xz
```

### Install crictl (required for kubeadm / Kubelet Container Runtime Interface (CRI))

```sh
$ CRICTL_VERSION="v1.11.1"
$ mkdir -p /opt/bin
$ curl -L "https://github.com/kubernetes-incubator/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz" | tar -C /opt/bin -xz
```

### Install kubeadm, kubelet, kubectl and add a kubelet systemd service:

```sh
$ RELEASE="$(curl -sSL https://dl.k8s.io/release/stable.txt)"

$ mkdir -p /opt/bin
$ cd /opt/bin
$ curl -L --remote-name-all https://storage.googleapis.com/kubernetes-release/release/${RELEASE}/bin/linux/amd64/{kubeadm,kubelet,kubectl}
$ chmod +x {kubeadm,kubelet,kubectl}

$ curl -sSL "https://raw.githubusercontent.com/kubernetes/kubernetes/${RELEASE}/build/debs/kubelet.service" | sed "s:/usr/bin:/opt/bin:g" > /etc/systemd/system/kubelet.service
$ mkdir -p /etc/systemd/system/kubelet.service.d
$ curl -sSL "https://raw.githubusercontent.com/kubernetes/kubernetes/${RELEASE}/build/debs/10-kubeadm.conf" | sed "s:/usr/bin:/opt/bin:g" > /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
$ systemctl enable --now kubelet
```
# Bash autocomplete

https://kubernetes.io/docs/tasks/tools/install-kubectl/#enabling-shell-autocompletion

# Lanzar contenedores

- Básico

`$ kubectl run http --image=katacoda/docker-http-server:latest --replicas=1`
`$ kubectl get deployments`
`$ kubectl describe deployment http`

- Exponer contenedor básico

`$ kubectl expose deployment http --external-ip="172.17.0.66" --port=8000 --target-port=80`
`$ curl http://172.17.0.66:8000`

- Exponer contenedor al lanzarlo

`$ kubectl run httpexposed --image=katacoda/docker-http-server:latest --replicas=1 --port=80 --hostport=8001`
`$ curl http://172.17.0.66:8001`

> Expone el Pod via Docker Port Mapping. Por lo tanto no se verá el servicio listado usando `$ kubectl get svc`

# Escalar contenedores

`$ kubectl scale --replicas=3 deployment http`
`$ kubectl get pods`

> Al arrancar los nuevos Pods estos serán añadodos al servicio de Load Balancing.

`$ kubectl describe svc http`
`$ curl http://172.17.0.66:8000`

# Lanzar Deployments

```yaml
vi deployment.yaml
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webapp1
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: webapp1
    spec:
      containers:
      - name: webapp1
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
```

`$ kubectl create -f deployment.yaml`
`$ kubectl get deployment`
`$ kubectl describe deployment webapp1`

## Actualizar Deployments

- Modifica el archivo anterior y pon `replicas: 4`

`$ kubectl apply -f deployment.yaml`
`$ kubectl get pods`
`$ curl host01:30080`

# Lanzar Servicios

```yaml
vi service.yaml
---
piVersion: v1
kind: Service
metadata:
  name: webapp1-svc
  labels:
    app: webapp1
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30080
  selector:
    app: webapp1
```

`$ kubectl create -f service.yaml`
`$ kubectl get svc`
`$ kubectl describe svc webapp1-svc`
`$ curl host01:30080`
