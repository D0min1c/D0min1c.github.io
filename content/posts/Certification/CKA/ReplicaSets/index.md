---
title: "ReplicaSets hands-on"
date: 2021-07-01
description: Sample post with multiple images, embedded video ect.
menu:
  sidebar:
    name: ReplicaSets hands-on
    identifier: ReplicaSets hands-on
    parent: CKA
    weight: 1
hero: images/HO_thumnail.jpg
---
찾아보기 쉽게 자주쓰는 명령어를 정리해두었습니다.
<!--more-->
## ReplicaSets hands-on

1. 현재 replicaSets 조회
- kubectl get replicaset
```
kubectl get replicaset
<--결과값-->
NAME              DESIRED(원하는 갯수)   CURRENT   READY   AGE
new-replica-set   4                        4         0       70s
```
2. replcatsets 상세 구성 확인
- kubectl describe replicaset replicaset명 
```
kubectl describe replicaset new-replica-set
<--결과값-->
Name:         new-replica-set
Namespace:    default
Selector:     name=busybox-pod
Labels:       <none>
Annotations:  <none>
Replicas:     4 current / 4 desired
Pods Status:  0 Running / 4 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  name=busybox-pod
  Containers:
   busybox-container:
    Image:      busybox777
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      echo Hello Kubernetes! && sleep 3600
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  3m46s  replicaset-controller  Created pod: new-replica-set-tx56t
  Normal  SuccessfulCreate  3m46s  replicaset-controller  Created pod: new-replica-set-24jhd
  Normal  SuccessfulCreate  3m46s  replicaset-controller  Created pod: new-replica-set-5xnp7
  Normal  SuccessfulCreate  3m46s  replicaset-controller  Created pod: new-replica-set-qgzbt
  ```

3. replicaset으로 생성된 pod들은 pod를 삭제해도 라이플 사이클에 의해 다시 살아나니, replicaset 자체를 지워야됨
- kubectl delete replicaset new-replica-set

4. Replicaset yaml로 생성 (아래 파일 수정)
```
apiVersion: v1 <-> apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-1
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
```

4. Replicaset yaml로 생성 (아래 파일 수정)
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-2
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend <-> tier가 일치하지 않음
  template:
    metadata:
      labels:
        tier: nginx <-> frontend
    spec:
      containers:
      - name: nginx
        image: nginx
```


5. Replicaset 롤링업데이트
- kubectl edit replicaset replicaset명
```
kubectl edit replicaset replicaset명
<내용 수정 후>
kubectl delete pod pod명 (새로 구성된 내용을 생성하기 위해 구버전 pod를 삭제함)
```

6. Replicaset 스케일 조절 (확장) (위 edit 명령어로 갯수를 늘려도되지만 scale 명령어 사용해봄)
- kubectl scale rs replicaset명 --replicas=8
```
kubectl scale rs new-replica-set --replicas=8
```

7. Replicaset 스케일 조절 (축소) (위 edit 명령어로 갯수를 늘려도되지만 scale 명령어 사용해봄)
- kubectl scale rs replicaset명 --replicas=2
```
kubectl scale rs new-replica-set --replicas=2
```