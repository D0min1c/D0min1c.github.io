---
title: GCP-Cloud Load Balancer...(내용정리)
categories:
  - "cloud"
tags:
  - "gcp"
  - "google cloud"
---
GCP CLB를 사용해본 뒤,  서비스의 특징에 대해 사용 후 정리해봤습니다.



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
    그러나 GCP CLB는 트래픽을 IP를 통해 받을 수 있도록 프런트엔드에 external IP를 부여할 수 있습니다.
![This is an image](/img/HC.jpg)

- Global Healthcheck가 존재합니다. (백엔드 서비스의 방화벽에서 필수적으로 Ingress를 허용해줘야 한다. )


- **백엔드의 자동 지능형 자동 확장 **

  - LB는 SDN 형태로 Software 적으로 만들어진 것이며, 기존의 물리적인 장비가 갖고 있던 하드웨어적 처리 성능 한계를 갖지는 않습니다.
  -  SDN의 특성 상, 개념적인 하나의 LB가 하나의 Software Process를 의미하지 않으며, 필요에 따라 (백엔드 구성에 맞물려) 실제의 Data Plane을 담당하는 구성요소는 수평 확장되는 구조입니다.
  -  위의 특성들에 의해 전반적으로 성능의 한계는 Backend Services의 규모/구성에 의해 결정되는 요소가 오히려 더 크다고 볼 수 있습니다.
  -  병목현상이 일어난다면 AWS는 node에 대한 이슈까지도 염두해두지만 GCP에서는 QPS에 대한 측정 자료나 Backend latency, Total latency 등의 값을 참고하는 것이 이슈를 해소하기 좋습니다.


- 단일 리전에서 애플리케이션을 사용할 때의 리전 부하 분산 
- 전 세계에서 애플리케이션을 사용할 때의 전역 부하 분산 **(Premium 등급의 Network Tier가 필요)**
- CLB 백단에 있는 모든 백엔드가 auto scaling되진 않습니다.
인스턴스 그룹이 관리형(Managed)인지 비관리형(unManaged)인지에 따라 다르게 동작하며 이 부분은 인스터스 그룹에 대한 이해가 필요합니다.
- 캐시된 콘텐츠 전달을 위한 [Cloud CDN](https://cloud.google.com/cdn?hl=ko)과의 통합

![This is an image](/img/session_affi.jpg)


- 기본적으로 5-tuple hash 기반의 부하분산을 지원하지만, Statefull한 애플리케이션을 구성했을 경우를 대비하여 세션 어피니티를 지원합니다. 세션 어피니티는 백엔드가 정상이고 용량이 있는 한 동일한 클라이언트의 모든 요청을 동일한 백엔드로 전송합니다.

---

## Cloud Load Balancer 기초

크게 **백엔드 서비스**, **호스트 및 경로규칙(HTTP/s)**, **프런트 엔드**로 나뉘며 각기 항목별 세부적인 항목들이 있습니다.

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

- 7계층에서 동작합니다.
- 여러가지 백엔드 유형을 지원하며,  **대상 HTTP(S) 프록시**는 클라이언트로부터 요청을 받습니다.
- HTTP(S) 프록시는 트래픽 라우팅을 결정하기 위해 URL 맵을 사용하여 요청을 판단(컨텐츠 기반 부하분산)합니다. 
- 프록시는 SSL 인증서를 사용하여 통신을 인증할 수도 있습니다.
- 세션 유지 시간은 600초로 고정되어있습니다. 

(websocket에는 적용되지않지만, 일반적으로 백엔드가 조기에 세션을 끊는 일이 발생하지 않도록 600초보다 길게 KeepAliveTime Out값을 설정하는 것이 좋습니다.)

- websocket을 지원합니다.
- gPRC를 지원합니다. (AWS ALB에도 2020년 11월 추가됐습니다.)
- QUIC를 지원합니다 . (Quick UDP Internet Connections / HTTP3 )



### TCP/UDP Load Balancing
![This is an image](/img/tcp_lb.jpg)

- 4 계층에서 동작합니다.
- pass-through LB 입니다. (글로벌 서비스를 위해 백엔드에서 모든 Ingress IP/port에 대한 허용이 필요합니다. )
- 수신(ingress)/요청(request) 패킷만 lb를 통과  - 송신(engress)/응답(response) 패킷은 lb를 거치지 않고 바로 client로 가는 DSR로 동작합니다.
- Proxy가 아닙니다. 
  - 부하 분산된 패킷은 소스 IP가 변경되지 않은 백엔드 VM에서 수신됩니다.
  - 부하 분산된 연결은 백엔드 VM에 의해 종료됩니다.
  - 백엔드 VM의 응답은 부하 분산기를 통하지 않고 클라이언트에 직접 전달됩니다. (위에서 설명한 DSR이 이것입니다.)
- 연결 추적 테이블과 구성 가능한 일관된 해싱 알고리즘을 사용하여 트래픽이 백엔드 VM에 분산되는 방식을 결정합니다.
- 백엔드 VM이 비정상 상태여도 TCP 패킷에 응답하면 세션을 다른 백엔드 VM으로 넘기지 않습니다. 

### TCP/SSL proxy Load Balancing
![This is an image](/img/tcp_pr_lb.jpg)

- TCP 연결을 통해 들어오는 트래픽이 부하 분산 레이어에서 종료된 후 TCP 또는 SSL을 통해 사용 가능한 가장 가까운 백엔드로 전달됩니다.
- 자동으로 트래픽을 사용자와 가장 가까운 백엔드로 라우팅합니다.
- TCP 80 or 8080 Port를 지원하지 않습니다. (HTTP/s LB로 쓰면됩니다.)
- TCP 프록시 부하 분산기는 역방향 프록시 부하 분산기입니다. 부하 분산기는 수신 연결을 종료한 후 부하 분산기에서 백엔드로 새 연결을 엽니다. 역방향 프록시 기능은 Google 프런트엔드(GFE)에서 제공합니다.



## 사용하면서 배운 것들

### 1. LB를 사용할 때 백엔드 서비스 구성은 생각보다 난해한 포인트들이 있다.

우선 LB의 백엔드 구성 간 선택지는 기본적으로 백엔드 서비스, 버킷, TCP에는 인스턴스 Pool을 직접 연결하기도 하지만
저의 경우 GCP 내의 VM으로 트래픽을 전달받아야하기 때문에 백엔드 서비스의 인스턴스 그룹 혹은 인스턴스 Pool로 셋팅합니다.

이때 그룹의 종류가 두가지로 나뉘는데 앞서 1편에서 말했던 인스턴스 그룹의 두 종류인 비관리형(Unmanaged Instance Group)과 관리형(Managed Instance Group)  그룹입니다.

1-1 **MIG**는 단일 region 내 멀티 zone에 VM 인스턴스를 분산함으로써 single zone failure 에 대응하여 워크로드의 고가용성을 높일 수 있습니다.

예를 들어, zonal failure 가 발생하거나 특정 zone 내에 위치한 인스턴스들에 장애 발생시, regional MIG 에 포함된 다른 zone에서 실행되는 인스턴스가 트래픽을 처리함으로써 고가용성을 구성할 수 있습니다. 

---

**[MIG 의 특징]**

- 애플리케이션 로드를 멀티 zone 으로 분산


- 최대 2,000개의 인스턴스까지 관리 가능 (zonal MIG의 2배)


- 멀티 zone 을 사용함으로써 zonal failure 및 단일 zone 내 인스턴스 그룹의 오작동과 같은 시나리오로부터 보호


- zonal failure 발생 혹은 zone 내 인스턴스 장애시, regional MIG에 포함된 다른 zone 에서 트래픽을 처리

---

1-2 **UIG**는 단순히 같은 zone내의 인스턴스들을 집합화 시킨 server pool이라고 이해하면 됩니다.

인스턴스 템플릿이 필요하지 않습니다. 단지 같은 zone내의 인스턴스라면 그룹화시켜 LB를 통해 트래픽을 받을 수 있습니다.

UIG를 사용할지 MIG를 사용할지는 어떤 서비스를 제공하는지에 따라 다릅니다.
[제가 근무하는 인프라실에서는 UIG로 구성합니다.]

MIG는 동일한 구성의 인스턴스를 여러 개 만들기 위한 것입니다. 기존 인스턴스 템플릿을 업데이트하거나 생성된 인스턴스 템플릿을 변경할 수 없습니다. 구성을 변경해야 하는 경우 새 인스턴스 템플릿을 만듭니다. 

그러나 UIG는 전혀다른 스펙과 구성의 인스턴스들을 그룹핑 시킬 수 있습니다.

따라서 게임서비스의 특성 상 순간적인 트래픽이 발생하기보단 꾸준한 유저들의 동접 수를 대응하기에 예상 트래픽의 최대치를 기준으로 서버를 생성하고 일정기간 모니터링하여 optimizing하는 방법으로 GCP를 활용하고 있습니다. 


그러나 항상 single zone failure에 대한 고민을 하고 있습니다. (좋은 방법 있을까요? (°∀°)b)

---

### 2. Global HealthCheck..?

AWS를 사용할때는 ALB의 sg를 참조하여 백엔드 EC2의 보안그룹을 유연하게 구성했다면, 다행히 GCP LB에서는 LB에 node 개념이 없습니다.

다만, Global HC라고하는  백엔드에 연결하는 전역 및 리전별 상태 확인 시스템을 제공합니다.

각 연결 시도를 *프로브*라고 부르고, 각 상태 확인 시스템을 *프로버*라고 부릅니다. 

### 3. 인스턴스와 인스턴스 그룹 재사용

하나의 인스턴스는 동시에 여러 인스턴스 그룹에 그룹핑될수 없습니다. 만약 한대의 인스턴스 내에 여러 서비스가 올라가있는 상태라면 Instance Group으로 나눌수는 없으니, 백엔드 포트를 서비스 마다 열어두고, 한대의 인스턴스 그룹을 여러 LB로 연결하는 방식으로 재사용 해야합니다.

위 기능들은 다음 포스팅에서 검증해서 보여드릴 예정입니다.

감사합니다.