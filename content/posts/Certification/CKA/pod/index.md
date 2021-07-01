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
찾기보기 쉽게 자주쓰는 명령어를 정리해두었습니다.
<!--more-->
## Pod hands-on
1. 클러스터 내 파드 확인
kubectl get pod -o wide (해석좀)

2. 특정이미지를 통한 pod 생성
kubectl run nginx --image=nginx (--restart=Never 옵션을 주면 리붓되지않음)

3. pod의 상세 내용 확인 (실패로그 yaml 선언내용 등등 상세 내용 확인 가능)
kubectl describe pod pod명

4. 클러스터 노드구성 확인
kubectl get nodes

5. pod 삭제
kubectl delete pod

6. pod 업데이트 
kubectl edit pod pod명
