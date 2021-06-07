---
title: GCP-Cloud Load Balancer...1
categories:
  - "cloud"
tags:
  - "gcp"
  - "google cloud"
---
GCP CLB를 사용해본 뒤 특징과 서비스에 대해 나름대로 풀어봤습니다.

-by Dominic

<!--more-->

## 배경

Load Balancing (이하 LB)는 Client로부터 들어오는 트래픽을 여러 Backend Server (Instance, VM)에 분산하는 기능을 가지고 있습니다. 이를 통해 트래픽에 의해 Backend Server의 리소스가 과다하게 점유되어 터지는 것을 방지하고자 트래픽 즉 부하들을 분산시켜 서비스의 고가용성을 얻음과 동시에 성능적인 이점을 가져갈 수 있습니다.

AWS에서 Elastic Load Balancing로 서비스되는 것처럼 GCP의 로드밸런서는 CLB(CloudLoadBalancer)로 서비스합니다.

두 벤더 모두를 경험해본 결과, AWS는 사용자에게 보다 친절합니다. 하지만 세세한 설정까지 원하는 사용자에게는 다소 불편합니다.

GCP는 일반적인 사용자에게 불친절할 정도로 세세한 설정이 필요합니다. 하지만 티끌까지 optimzing을 원하는 사용자에게는 이보다 좋은  플랫폼은 없다고 느껴집니다.

사용 간 느꼈던 점을 기록하기 앞서 GCP에서 서비스하는 LB의 특징들을 요약해봤습니다. 



(Google Docs가서 보시면 보다 정확한 내용을 알 수 있습니다.)


## CLB의 종류

----

- **HTTP/HTTPS Load Balancing**
- **TCP/UDP Load Balancing**
- **TCP/SSL Proxy Load balancing**
----

CLB는 어느 프로토콜을 지원하는지에 따라 3가지의 종류로 나뉘어집니다. GCP에서의 표기와 다를 수 있습니다.


기능적으로 헷갈리기는 부분이 있어, 제가 보기쉽게 나눠 표기하였습니다.

중요한건 HTTP(S), TCP, TCP proxy LB의 차이를 명확하게 구분지을 줄 알아야 서비스에서 원하는 로드밸런서를 선택할 수 있다는 점입니다.

---

## CLB의 특징

- 프런트엔드 역할을 하는 단일 **anycast IP address**
- AWS의 ELB를 사용하던 유저들은 다소 생소할 수 있습니다. 왜냐면 AWS ELB는 FQDN을 통해 ELB를 제공하기 때문입니다. 

이는 AWS ELB의 Node라는 개념에 의해 어쩔 수 없이 Endpoint로 FQDN을 제공할 수 밖에 없는 AWS의 특징입니다. 
    (상세 내용은 AWS LB에서 ㅎㅎ)
    그러나 GCP CLB는 트래픽을 IP를 통해 받을 수 있도록 프런트엔드에 external IP를 부여할 수 있습니다.


- **백엔드의 자동 지능형 자동 확장**
- 단일 리전에서 애플리케이션을 사용할 때의 리전 부하 분산 
- 전 세계에서 애플리케이션을 사용할 때의 전역 부하 분산 **(Premium 등급의 Network Tier가 필요)**
- CLB 백단에 있는 모든 백엔드가 auto scaling되진 않습니다.
인스턴스 그룹이 관리형(Managed)인지 비관리형(unManaged)인지에 따라 다르게 동작하며 이 부분은 인스터스 그룹에 대한 이해가 필요합니다.
- 캐시된 콘텐츠 전달을 위한 [Cloud CDN](https://cloud.google.com/cdn?hl=ko)과의 통합

![This is an image](/img/session_affi.jpg)


- 기본적으로 5-tuple hash 기반의 부하분산을 지원하지만, Statefull한 애플리케이션을 구성했을 경우를 대비하여 세션 어피니티를 지원한다. 

---

## Cloud Load Balancer 기초

크게 **백엔드 서비스**, **호스트 및 경로규칙(HTTP/s)**, **프런트 엔드**로 나뉘며 각기 항목별 세부적인 항목들이 또 있습니다.

#### Backend
![This is an image](/img/backend_1.JPG)


- 백엔드 유형 / backend port / 인스턴스 그룹 / 밸런싱 모드 / A capacity scaler / 타임아웃 / 세션어피니티 / 헬스체크 (포트 or 경로)


#### 호스트 및 경로 규칙 (HTTP/s)
![This is an image](/img/lb_host.jpg)


- 호스트 / 경로 / 백엔드


#### FrontEnd
![This is an image](/img/lb_front.jpg)


- 프로토콜 / 네트워크 계층 / external IP / Front Port

### HTTP/HTTPS Load Balancing
![This is an image](/img/HTTP_LB.jpg)

7계층에서 동작합니다.

여러가지 백엔드 유형을 지원하며,  **대상 HTTP(S) 프록시**는 클라이언트로부터 요청을 받습니다.



websocket 을 지원합니다.



HTTP(S) 프록시는 트래픽 라우팅을 결정하기 위해 URL 맵을 사용하여 요청을 판단(컨텐츠 기반 부하분산)합니다. 



프록시는 SSL 인증서를 사용하여 통신을 인증할 수도 있습니다.



세션 유지 시간은 600초로 고정되어있습니다. 



(websocket에는 적용되지않지만, 일반적으로 백엔드가 조기에 세션을 끊는 일이 발생하지 않도록 600초보다 길게 KeepAliveTime Out값을 설정하는 것이 좋습니다.)



gPRC 를 지원합니다.....!!!!!!!!

### TCP/UDP Load Balancing
![This is an image](/img/tcp_lb.jpg)


흔히  pass-through LB를 알고 있다면 이미 CLB의 TCP LB를 알고있다고해도 무방합니다.

수신(ingress)/요청(request) 패킷만 lb를 통과  - 송신(engress)/응답(response) 패킷은 lb를 거치지 않고 바로 client로 가는 DSR로 동작합니다.

라우팅 알고리즘은 
### TCP/UDP Load Balancing
![This is an image](/img/tcp_pr_lb.jpg)
라우팅  알고리즘은 workflow는
나머진 밥먹고 쓸래...