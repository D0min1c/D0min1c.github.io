---
title: "etcd backup&restore"
date: 2021-07-01
description: Sample post with multiple images, embedded video ect.
menu:
  sidebar:
    name: etcd backup&restore
    identifier: etcd backup&restore
    parent: CKA
    weight: 1
hero: images/HO_thumnail.jpg
---
찾기보기 쉽게 자주쓰는 명령어를 정리해두었습니다.
<!--more-->
## etcd ?
etcd 는 분산 시스템에서 사용할 수 있는 분산형 키-값 (key-value) 저장소.
 - 클러스터의 DB역할을 하며, 설정 값이나 클러스터의 상태를 저장하는 서버.
 - 클러스터의 정보와 데이터를 가지고 있기 떄문에 백업에 대한 이해와 작업방법을 알고 있어야 함 (<- 오늘해볼 것)

## etcd backup
