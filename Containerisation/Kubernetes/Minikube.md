---
title: Minikube
description: 
published: true
date: 2019-03-09T11:16:37.245Z
tags: 
---

`$ minikube version`
`$ minikube start`
`$ kubectl cluster-info`
`$ kubectl get nodes`
`$ kubectl run first-deployment --image=katacoda/docker-http-server --port=80`
`$ kubectl get pods`
`$ kubectl expose deployment first-deployment --port=80 --type=NodePort`
```
$ export PORT=$(kubectl get svc first-deployment -o go-template='{{range.spec.ports}}{{if .nodePort}}{{.nodePort}}{{"\n"}}{{end}}{{end}}')
echo "Accessing host01:$PORT"
curl host01:$PORT
```