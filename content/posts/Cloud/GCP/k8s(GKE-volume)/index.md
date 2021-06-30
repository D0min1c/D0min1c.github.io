---
title: "k8s(GKE-volume)"
date: 2021-06-26
description: Sample post with multiple images, embedded video ect.
menu:
  sidebar:
    name: k8s(GKE-volume)
    identifier: k8s(GKE-volume)
    parent: GCP
    weight: 10
hero: images/CKA_thumnail.jpg
---
GKE를 생성해서 kubectl에 익숙해져보자.
<!--more-->
### GCP volume
GCP에서의 볼륨 옵션은 그들의 네트워킹 구조와 마찬가지로 zonal, Regional을 선택할 수 있다.
오늘 해볼 것은 Regional SSD Persistent Disk를 생성하여 각 영역에 있는 K8s 노드에 붙여볼 예정이다.

### 리소스 생성
