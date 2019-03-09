---
title: Single
description: 
published: false
date: 2019-03-09T16:28:49.270Z
tags: 
---

> https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/


```
$ kubeadm init
$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
$ kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard.yaml


$ kubectl apply -f "https://cloud.weave.works/k8s/scope.yaml?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

```
$ kubeadm join 172.17.4.10:6443 --token 3si408.wexib33sx2vfopgl --discovery-token-ca-cert-hash sha256:b2e3e58472d0fac1a1f075271c88306a6c2bfa46ce66463aa490c33f21648c83
```

Kubernetes Dashboard:

```
https://172.17.4.10:31115/#!/pod/weave/weave-scope-app-6cbf5dbc45-2stv2?namespace=_all
$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin | awk '{print $1}')
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi10b2tlbi01bG00eiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJhZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImM3MzViMmZiLTNkNmQtMTFlOS1hMDU5LTAwNTA1Njg0MmU2MCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTphZG1pbiJ9.gOwBbx3x7NrrwaVsfPfSALEXuOC-XKnxX0aZsu4tHD101skP689A-HquQjzMDJhZ7iHHV9BwaMLB_nX-9QcrOD8an5M0uvcxMG9mvrpNjvhZE-6ZsbWI1zpCMgCgHCtsL2lt8pW9E7zH5U7HFx2-0H-Pi1G4fZz13IyXXt-CjzhpUL5DjHfIZjPlqirzP5rb9jMnHV29nUpvc63YnG-PI1FTctGXszUN-z2e_p2BvKNsq7i7J5GT_cbyBs3vIzzFtfl2lRMbUlezYO_4oLJ6dBRT9J8Ey8lXJmNcceWRqBPQDlbdZpI4XKwVqeYiY1VIReN_q-IQnW6RVJYM1w0KxQ
```