---
title: CKA 취득해보자 - 2021-06-15 ~
categories:
  - "CKA"
tags:
  - "k8s"
  - "CKA"
  - "Kubernetes"
---
2021-06-15부터 CKA 취득을 위해 맨땅에 헤딩해본 피나는 결과 기록
<!--more-->

## 배경
CKA를 취득해 쿠버네티스에 입문해보자..!

##  CKA

작년 11월쯤인가.. CKA할인소식을 듣고 냅다 구매했다가 까먹고 있었다.
이제 슬..만료일이 다가오는거 같으니, 준비해보자.

## 범위

총 5가지 과목을 준비해야한다. 참고로 모든 시험은 핸즈온! 이다.

1. Cluster Architecture, Installation & COnfiguration (25%)
2. Workloads & Scheduling (15%)
3. Services & Networking (20%)
4. Storage (10%)
5. Troubleshooting (30%)

으어,, 재밌겠다.. ~~(군침)~~ 아래부터는 라이브하게 그날 공부한것들을 정리할 예정이다.

### 2021-06-15 공부한 내용
```
환경 : Amazon Linux 2 (t2 패밀리)
amazon-linux-extras install | docker 패키지 설치
service docker start | 도커 서비스 시작
```
- 도커 설치 직후 상태 
![This is an image](/img/docker_info.jpg)

#### 기본 개념
- 기본적인 docker network 구조
![This is an image](/img/docker_net.jpg)
이미지 출처 : [커피고래님 블로그](https://coffeewhale.com/k8s/network/2019/04/19/k8s-network-01/)

Host[eth0 -> docker0(bridge) -> veth0 -> container]
컨테이너를 새로이 배포할때마다 컨테이너에게 veth0라는 가상 네트워크 인터페이스를 할당하여 Docker0에 연결
linked virtual ethernet device pair로 컨테이너와 Bridge 간 연결을 한다.
- eth0 : 실제 내 서버 (VM)에 붙어있는 네트워크 인터페이스 (그냥 physical network interface)
- docker 0 : netmask는 /16으로 설정되었다. (ip도 자동할당)
  - 자동할당이지만 DHCP로 받은 IP는 아니고 Docker 내부로직에 의해 할당받은 IP이다. ~~(DHCP서버가 없는데 어떻게받아요 ㅠ)~~
  - virtual ethernet bridge의 역할을 한다. (brctl 명령어로 bridge id와 STP모드 그리고 현재 연결되어있는 컨테이너 갯수를 interface로 확인 할 수 있다.)
  - 가상의 bridge(L2)container가 하나 생성되면 이 bridge에 container의 interface가 하나씩 binding 된다.
  따라서, 같은 도커 호스트 내 컨테이너들은 이 docker0 덕분에 서로의 IP를 알면 통신이 가능한 상태이다. (localhost 통신 가능)
- veth0 : container에 할당되는 가상 네트워크 인터페이스 (namespace를 통해 독립적인 network 할당)

![This is an image](/img/docker_0.jpg)

- k8s를 이루는 기본 단위 : pods (컨테이너들의 집합, 음...AWS에서 하나의 VPC로 이해하면 될랑가..)
  - Pods 내 컨테이너들은 기본적으로 localhost 통신

#### k8s 아키텍쳐

![This is an image](/img/k8s_archi.jpg)
- Master node와 worker Node로 구성됨
  - master node 구성 : API server, Schedular, Controller, etcd (클러스터의 상태, 컨테이너설정, 네트워킹 구성 등 관리)
  - Worker node 구성 : kubelet, kube-proxy
- k8s 동작 방식
  - API server를 통해 API를 노출한다. Kubectl이라는 로컬 클라이언트를 사용하여 API와 통신한다.
  - Schedular는 이러한 API들을 적절한 노드로 할당해주는 스케줄링 작업을 담당한다.
  - 각 worker node들에서 kubelet은 컨테이너 실행 요청을 수신하고 필요한 리소스들을 관리한다.
  - 각 worker node들에서 kube-proxy는 컨테이너의 네트워크를 담당한다.

