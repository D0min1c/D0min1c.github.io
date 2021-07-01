---
title: "Pod hands-on"
date: 2021-07-01
description: Sample post with multiple images, embedded video ect.
menu:
  sidebar:
    name: Pod hands-on
    identifier: Pod hands-on
    parent: CKA
    weight: 1
hero: images/HO_thumnail.jpg
---
찾아보기 쉽게 자주쓰는 명령어를 정리해두었습니다.
<!--more-->
## Pod hands-on

1. 특정이미지를 통한 pod 생성
- kubectl run nginx --image=nginx (--restart=Never 옵션을 주면 리붓되지않음)
```
kubectl run nginx --image=nginx --restart=Never
<--결과값-->
pod/nginx created
```


2. 클러스터 내 파드 확인
- kubectl get pod -o wide (--selector 옵션으로 특정 셀럭터기준으로 출력 가능) 
```
kubectl get pod -o wide
<--결과값-->
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE                                               NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          4s    10.244.1.3   ip-172-31-41-149.ap-northeast-1.compute.internal   <none>           <none>

```

3. pod의 상세 내용 확인 (실패로그 yaml 선언내용 등등 상세 내용 확인 가능)
- kubectl describe pod pod명
```
kubectl describe pod nginx
<--결과값-->
Name:         nginx
Namespace:    default
Priority:     0
Node:         ip-172-31-41-149.ap-northeast-1.compute.internal/172.31.41.149
Start Time:   Thu, 01 Jul 2021 07:25:12 +0000
Labels:       run=nginx
Annotations:  <none>
Status:       Running
IP:           10.244.1.3
IPs:
  IP:  10.244.1.3
Containers:
  nginx:
    Container ID:   docker://204dc60cdb5e56a5ada0088920503dcfeb907276030cf0cdbc3bb75b9188c8e5
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:47ae43cdfc7064d28800bc42e79a429540c7c80168e8c8952778c0d5af1c09db
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 01 Jul 2021 07:25:14 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-bjkdq (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-bjkdq:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  55s   default-scheduler  Successfully assigned default/nginx to ip-172-31-41-149.ap-northeast-1.compute.internal
  Normal  Pulling    55s   kubelet            Pulling image "nginx"
  Normal  Pulled     53s   kubelet            Successfully pulled image "nginx" in 1.87586268s
  Normal  Created    53s   kubelet            Created container nginx
  Normal  Started    53s   kubelet            Started container nginx

```

4. 클러스터 노드구성 확인
- kubectl get node

```
kubectl get node
<--결과값-->
NAME                                               STATUS   ROLES                  AGE   VERSION
ip-172-31-35-14.ap-northeast-1.compute.internal    Ready    control-plane,master   2d    v1.21.1
ip-172-31-37-29.ap-northeast-1.compute.internal    Ready    <none>                 2d    v1.21.1
ip-172-31-41-149.ap-northeast-1.compute.internal   Ready    <none>                 2d    v1.21.1
```

5. pod 삭제
- kubectl delete pod
```
kubectl delete pod nginx
<--결과값-->
pod "nginx" deleted
```

6. pod 업데이트 
- kubectl edit pod pod명
 내용 수정하면 파드가 업데이트 된다.
```
kubectl edit pod test-nginx-1
<--결과값-->
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2021-07-01T07:31:48Z"
  name: test-nginx1
  namespace: default
  resourceVersion: "247954"
  uid: 5b6b89de-c0e4-4730-90a9-6c2de5468972
spec:
  containers:
  - image: nginx:1 -> nginx:2
    imagePullPolicy: IfNotPresent
    name: test-nginx1
    ports:
    - containerPort: 80
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-9lg6m
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: ip-172-31-41-149.ap-northeast-1.compute.internal
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-9lg6m
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
-----상태 / host IP / uptime 등등 중략 -----

```