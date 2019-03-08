---
title: Kubernetes
description: 
published: true
date: 2019-03-08T18:08:35.194Z
tags: 
---

---
title: Kubernetes
description: 
published: true
date: 2019-03-08T15:02:06.971Z
tags: 
---

> https://kubernetes.io/

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

# Requerimientos para Kubernetes utilizando kubeadm+

> https://kubernetes.io/docs/setup/independent/install-kubeadm/

### Install CNI plugins (required for most pod network):
```
$ CNI_VERSION="v0.6.0"
$ mkdir -p /opt/cni/bin
$ curl -L "https://github.com/containernetworking/plugins/releases/download/${CNI_VERSION}/cni-plugins-amd64-${CNI_VERSION}.tgz" | tar -C /opt/cni/bin -xz
```

### Install crictl (required for kubeadm / Kubelet Container Runtime Interface (CRI))

```
$ CRICTL_VERSION="v1.11.1"
$ mkdir -p /opt/bin
$ curl -L "https://github.com/kubernetes-incubator/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz" | tar -C /opt/bin -xz
```

### Install kubeadm, kubelet, kubectl and add a kubelet systemd service:

```
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
