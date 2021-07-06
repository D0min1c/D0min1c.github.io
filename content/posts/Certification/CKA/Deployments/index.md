---
title: "Deployments hands-on"
date: 2021-07-02
description: Sample post with multiple images, embedded video ect.
menu:
  sidebar:
    name: Deployments hands-on
    identifier: Deployments hands-on
    parent: CKA
    weight: 3
hero: images/HO_thumnail.jpg
---
찾아보기 쉽게 자주쓰는 명령어를 정리해두었습니다.
<!--more-->
## Deployments hands-on

1. deployments 상세 정보 확인
- kubectl describe deployment
```
kubectl describe deployment

<--결과값-->
Name:                   frontend-deployment
Namespace:              default
CreationTimestamp:      Fri, 02 Jul 2021 05:01:56 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               name=busybox-pod
Replicas:               4 desired | 4 updated | 4 total | 0 available | 4 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  name=busybox-pod
  Containers:
   busybox-container:
    Image:      busybox888
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      echo Hello Kubernetes! && sleep 3600
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      False   MinimumReplicasUnavailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:  <none>
NewReplicaSet:   frontend-deployment-56d8ff5458 (4/4 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  96s   deployment-controller  Scaled up replica set frontend-deployment-56d8ff5458 to 4
```

2. 틀린 YAML 찾기
```
apiVersion: apps/v1
kind: deployment <-> Deployment 대문자
metadata:
  name: deployment-1
spec:
  replicas: 2
  selector:
    matchLabels:
      name: busybox-pod
  template:
    metadata:
      labels:
        name: busybox-pod
    spec:
      containers:
      - name: busybox-container
        image: busybox888
        command:
        - sh
        - "-c"
        - echo Hello Kubernetes! && sleep 3600
```

정리 Pod > replicaset > deployment