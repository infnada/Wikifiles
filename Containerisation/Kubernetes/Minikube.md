---
title: Minikube
description: Minikube
published: true
date: 2019-04-09T13:43:36.010Z
tags: 
---

> Reemplazar `&#123;&#123;` por `{{` .... ya que sino no me renderiza correctamente este HTML

`$ minikube version`
`$ minikube start`
`$ kubectl cluster-info`
`$ kubectl get nodes`
`$ kubectl run first-deployment --image=katacoda/docker-http-server --port=80`
`$ kubectl get pods`
`$ kubectl expose deployment first-deployment --port=80 --type=NodePort`
```
$ export PORT=$(kubectl get svc first-deployment -o go-template='&#123;&#123;range.spec.ports}}&#123;&#123;if .nodePort}}&#123;&#123;.nodePort}}&#123;&#123;"\n"}}&#123;&#123;end}}&#123;&#123;end}}')
echo "Accessing host01:$PORT"
curl host01:$PORT
```
