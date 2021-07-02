---
title: "Namespace hands-on"
date: 2021-07-01
description: Sample post with multiple images, embedded video ect.
menu:
  sidebar:
    name: Namespace hands-on
    identifier: Namespace hands-on
    parent: CKA
    weight: 1
hero: images/HO_thumnail.jpg
---
찾아보기 쉽게 자주쓰는 명령어를 정리해두었습니다.
<!--more-->
# Namespce
네임스페이스는 여러 개의 팀이나, 프로젝트에 걸쳐서 많은 사용자가 있는 환경에서 사용하도록 만들어졌다. 사용자가 거의 없거나, 수 십명 정도가 되는 경우에는 네임스페이스를 전혀 고려할 필요가 없다. 네임스페이스가 제공하는 기능이 필요할 때 사용하도록 하자.

네임스페이스는 이름의 범위를 제공한다. 리소스의 이름은 네임스페이스 내에서 유일해야하지만, 네임스페이스를 통틀어서 유일할 필요는 없다. 네임스페이스는 서로 중첩될 수 없으며, 각 쿠버네티스 리소스는 하나의 네임스페이스에만 있을 수 있다.

쿠버네티스는 처음에 네 개의 초기 네임스페이스를 갖는다.

- default : 다른 네임스페이스가 없는 오브젝트를 위한 기본 네임스페이스
- kube-system : 쿠버네티스 시스템에서 생성한 오브젝트를 위한 네임스페이스
- kube-public : 이 네임스페이스는 자동으로 생성되며 모든 사용자(인증되지 않은 사용자 포함)가 읽기 권한으로 접근할 수 있다. 이 네임스페이스는 주로 전체 클러스터 중에 공개적으로 드러나서 읽을 수 있는 리소스를 위해 예약되어 있다. 이 네임스페이스의 공개적인 성격은 단지 관례이지 요구 사항은 아니다.
- kube-node-lease : 클러스터가 스케일링될 때 노드 하트비트의 성능을 향상시키는 각 노드와 관련된 리스(lease) 오브젝트에 대한 네임스페이스

--namespace플래그를 사용하면된다.
## Namespace hands-on
1. namespace 조회
- kubectl get namespace
```
kubectl get namespace
<--결과값-->
NAME              STATUS   AGE
default           Active   7m23s
dev               Active   54s
finance           Active   54s
kube-node-lease   Active   7m26s
kube-public       Active   7m26s
kube-system       Active   7m27s
manufacturing     Active   54s
marketing         Active   54s
prod              Active   54s
research          Active   54s
```
2. namespace 내 파드의 개수 확인하기
- kubectl get pod --namespace 네임스페이스 명
```
kubectl get pod --namespace research
NAME    READY   STATUS             RESTARTS   AGE
test-1   0/1     CrashLoopBackOff   5          5m8s
test-2   0/1     CrashLoopBackOff   5          5m8s
```

3. 특정 네임스페이스 내 Pod 실행시키기
kubectl run pod명 --image=이미지명 --namespace 네임스페이스 명
```
kubectl run redis --image=redis --namespace finance
<--결과값-->      
kubectl get pod --namespace finance 
NAME      READY   STATUS    RESTARTS   AGE
redis     1/1     Running   0          62s
```
4. 특정 네임스페이스 내 Pod 실행시키기
kubectl run pod명 --image=이미지명 --namespace 네임스페이스 명
```
kubectl run redis --image=redis --namespace finance
<--결과값-->
kubectl get pod --namespace finance 
NAME      READY   STATUS    RESTARTS   AGE
redis     1/1     Running   0          62s
```

5. 네임스페이스 간 파드의 통신
네임스페이스를 하나의 서브넷이라고 생각하면 좀 더 편한거같다.
가령 파드는 기본적으로 격리되지않아 파드간 통신이 원활하지만 네임스페이스를 통해 특정 파드를 선택하는 네트워크 폴리시가 있다면 해당 파드는 네트워크 폴리시에 의해 격리된다. 기본정책이 all deny