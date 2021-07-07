---
title: "비기너라면 알아두어야 할 ELB"
date: 2021-07-05
description: Sample post with multiple images, embedded video ect.
menu:
  sidebar:
    name: ㄴ ELB의 모든것
    identifier: AWS_ELB
    parent: AWS_Beginners😆
    weight: 5
hero: images/host_thumnail.jpg
---
AWS를 사용하시려는 모든 유저분들이 간단히 보기에 좋은 내용입니다. AWS Docs는 너무 어려워...! 그렇다면 들어오세요.

<!--more-->

## 배경
AWS를 시작하시는 분들을 위해 간단하게 ELB에 대해 설명합니다.

### ELB 개념 (ElasticLoadBalancer)
ELB는 AWS의 로드밸런싱 서비스입니다.
둘 이상의 가용 영역에서 EC2 인스턴스, 컨테이너, IP 주소 등 여러 대상에 걸쳐 수신되는 트래픽을 자동으로 분산합니다. 

기본적으로 비정상 상태의 서버를 감지하여 정상적으로 동작하는 서버에 트래픽을 분산하지만,
만약 헬스체크가 모두 실패라면 모든 타겟에 트래픽을 로드밸런싱하는 특징이 있습니다.
 
3가지 종류의 로드밸런스를 지원하고 최근 1개가 추가되어 총 4가지의 로드밸런서 타입을 지원합니다. (GWLB에 대해서는 나중에 다루겠습니다.)
- CLB (Classic Load Balancer)
제일 오래된 타입으로 기본적으로 4계층, 7계층의 LB를 모두 지원하며, 타겟 그룹아닌 인스턴스를 등록하여 백엔드로 지정합니다.
라우팅 알고리즘
  - TCP 리스너에 대한 라운드 로빈 라우팅 알고리즘 사용
  - HTTP 및 HTTPS 리스너에 대한 최소 미해결 요청 라우팅 알고리즘 사용

- ALB (Application Load Balancer)
7계층 L7 로드밸런서로 SG를 적용하여 클라이언트의 요청에 대해 로드밸런서에서 필터링할 수 있습니다.

라우팅 알고리즘
  - 라운드 로빈
  - 최소 미해결 요청 라우팅

- NLB (Network Load Balancer)
4계층 L4 로드밸런서 입니다. 
라우팅 알고리즘
  - Hash 알고리즘을 사용하여 라우팅합니다. (5tuple)
    - protocol, source IP/port, Destination IP/port, TCP Sequence

### ELB 동작방식
![This is an image](images/lb_6.jpg)

우선 동작방식을 알아보기위해 test-LB들을 생성해보겠습니다.

#### ALB 생성
![This is an image](images/lb_1.jpg)
  - ELB 생성할 때 가용영역은 최소 2개를 선택해야 합니다.

![This is an image](images/lb_2.jpg)
  - 대상그룹에 대해 정의와 라우팅 규칙은 대상 포트는 80으로 설정합니다.
  - ALB의 보안그룹은 우선 동일하게 ANY 80으로 열어두었습니다.
  - 리스너 규칙은 80으로 들어오는 요청에 대해 alb-target-group으로 트래픽이 라우팅되도록 설정했습니다.
  - 대상그룹의 타겟 인스턴스는 미리 생성해둔 아파치 서버 2대를 연결했습니다. (보안그룹과 서버 내 서비스가 정상적으로 동작중인지 확인해주세요.)

![This is an image](images/lb_3.jpg)
  - 정상적으로 타겟의 상태가 Healthy를 보여줍니다. ALB endpoint로 조회가 되는지 확인해봅니다.

#### ALB 테스트
![This is an image](images/lb_4.jpg)
  - 조회가 정상적으로 이뤄집니다. 서버에서 테스트해봅시다.

![This is an image](images/lb_5.jpg)
  - 잘 분산되어 요청이 가는걸 확인했습니다. 

그럼 만약 ALB dns를 nslookup으로 조회하면 백엔드 서버의 IP가 보일까요?

```
nslookup test-alb-842437062.ap-northeast-1.elb.amazonaws.com

<--결과값-->
Server:		10.0.0.2
Address:	10.0.0.2#53

Non-authoritative answer:
Name:	test-alb-842437062.ap-northeast-1.elb.amazonaws.com
Address: 52.193.80.244
Name:	test-alb-842437062.ap-northeast-1.elb.amazonaws.com
Address: 54.248.216.43
```
엥, 분명 ALB DNS를 조회헀는데 EC2와는 전혀 상관없는 IP들이 보입니다.
이때 조회되는 ip들은 ELB의 Node IP입니다.

#### ELB 노드 동작방식
ELB에서는 node의 개념이 있습니다.
Elastic Load Balancing은 활성화된 각 가용 영역에 대해 네트워크 인터페이스를 생성합니다.


ELB에서 바로 타겟에 연결되어있는(혹은 바로 인스턴스로 연결되어있는) 서버로 트래픽을 보내는 구조가 아닌 ELB 생성 시 선택한 가용영역의 서브넷 내 노드를 생성하여 해당 노드를 통해 트래픽을 백엔드 서버로 전송해줍니다. (proxy의 개념)


로드 밸런서에서 가용 영역을 활성화하면 Elastic Load Balancing이 해당 가용 영역에서 로드 밸런서 노드를 생성합니다. 가용 영역에 대상을 등록하지만 가용 영역은 활성화하지 않는 경우 이러한 등록된 대상은 트래픽을 수신하지 않습니다. 

이 노드들은 트래픽이 증가할 때 자동으로 Scale in/out을 수행하며 수시로 IP가 변경됩니다.
따라서 갑작스러운 이벤트로 인해 트래픽이 증가할 것을 예상해서 백엔드 서버의 Scale Up, out을 진행해도 노드의 갯수가 부족할 경우
트래픽은 Drop됩니다. 

이런 경우를 대비하여 AWS에 미리 Node를 warm-up시켜놓을 수 있습니다. pre-warm 요청을 통해서 말이죠.
방법에 대해서는 다른 블로그들에서 친절히 설명되어있기에 따로 다루진 않겠습니다.

#### ALB 옵션사항
1. Sticky Session 옵션을 활성화 할 경우 cookie값을 이용하여 세션을 고정시켜서 요청이 유지될 수 있도록 고정하는 것이 가능합니다.
2. ALB의 Idle Connection Timeout 값을 조정해야한다. 만약 Timeout 시간 내 송수신이 정상적으로 이뤄지지 않을 경우, 503 에러를 발생시킬 수 있기 때문에 백엔드 서버의 아파치에서 keepalive 값은 ALB의 TImeout보다 길게 설정되어있어야 합니다.

#### NLB 생성
마찬가지로 NLB를 생성해줍니다.
![This is an image](images/nlb_1.jpg)
  - 특이사항으로 NLB에서 가용영역을 지정할 때는 Elastic IP를 통해 IP를 고정할 수 있습니다.
    이말인 즉슨 NLB는 노드 IP를 고정하여 사용하는게 가능합니다.
  - NLB는 별도의 LB 방화벽이 없습니다. 서버의 방화벽이 모든 트래픽의 필터링을 제어합니다.
   
#### NLB 테스트
![This is an image](images/nlb_2.jpg)
  - 정상적으로 타겟의 상태가 Healthy를 보여줍니다. NLB endpoint로 조회가 되는지 확인해봅니다.

![This is an image](images/nlb_3.jpg)
  - 응답이 정상적으로 돌아오긴 합니다만, 로드밸런싱 그러니까 새로고침을 반복해도 2번서버가 안나옵니다. 이유는 아래와 같습니다.

#### NLB 동작방식
NLB의 알고리즘은 위에서 설명한대로 5-tuple hash 알고리즘을 사용합니다.
그렇기에 Idle Connection Timeout인 350초 이후에도 동일한 source에서 동일한 요청이 들어올 경우 동일한 서버로 요청이 유지됩니다.
ALB는 Idle Connection Timeout 수정이 가능하지만 NLB는 350초로 고정입니다.

이론적으론 그렇습니다만, 사실 조금 더 복잡하게 생각이 필요합니다.

비기너에선 좀 더 서비스에 익숙해진 뒤 딥다이브가 필요한 내용이기에 궁금하시면 아래 링크를 타고 들어가 리눅서님의 포스팅을 참고해주세요.

[리눅서님 - AWS-NLB-Sticky-sessions-timeout](https://linuxer.name/2020/08/aws-nlb-sticky-sessions-timeout/)

또한 타겟 그룹의 대상이 인스턴스ID를 사용하는지와 IP주소를 사용하는지에 따라 백엔드에 제공되는 클라이언트 IP값이 다릅니다.
이는 response Return 방식이 다르기 때문인데, 여타 유명 기술블로그를 보면 NLB는 DSR 구조라고 설명하고 있습니다.

반은 맞고 반은 틀립니다.

Instance ID로 지정한 Taget Group의 경우 DSR 방식으로 동작하여 Response 시에 EC2 Instance는 직접 Client에게 패킷을 전달해야하기에
EC2 인스턴스는 IGW, NAT 등을 통해 아웃바운드(Outbound) 통신을 허용해야합니다만

반면 IP로 지정한 Target Group의 경우 Response Traffic 또한 LB를 타고 나가기에 변동사항은 없습니다.

### 추가적으로 알아두면 좋은 내용
1. 교차영역 로드밸런싱
일반적으로 몇개의 영역(노드)를 사용하여 LB를 구성하든 트래픽 분배의 기준은 Zone입니다.

그렇다면 생각해봅시다. a존의 2개의 인스턴스가 있고 b존에 8개의 인스턴스가 있다면 총량 100%의 트래픽을
a존 50% (인스턴스 당 25%), b존 50%(인스턴스 당 6.25%) 분배받습니다. a존의 백엔드 인스턴스가 받게되는 트래픽이 b존에 비해 너무 가중됩니다.

이떄 교차영역 로드밸런싱을 활성화 하게 될 경우 트래픽 분배의 기준이 인스턴스로 변경되어 트래픽은 모든 인스턴스에 균등하게 10% 배분됩니다.

Application Load Balancer에서는 교차 영역 로드 밸런싱이 항상 사용됩니다.


Network Load Balancer에서는 기본적으로 교차 영역 로드 밸런싱이 비활성화됩니다. 


로드 밸런서를 생성한 후 언제든지 교차 영역 로드 밸런싱을 활성화하거나 비활성화할 수 있습니다.

### 결론
ELB의 3가지 타입에 대해 알아봤습니다. (GatewayLB(GWLB)는 추후 따로 알아보겠습니다.)

보통 LB를 선택하는 조건은 아래와 같습니다.

NLB – TCP 트래픽 로드 밸런싱에 이상적인 NLB는 초저 대기 시간을 유지하면서 초당 수백만 건의 요청을 처리 할 수 ​​있습니다. NLB는 가용성 영역 당 하나의 고정 IP 주소를 사용하면서 갑작스럽고 불안정한 트래픽 패턴을 처리하도록 최적화되어 있습니다.

ALB – HTTP 및 HTTPS 트래픽의 고급 로드 밸런싱에 이상적인 ALB는 마이크로서비스 및 컨테이너 기반 응용 프로그램을 포함한 최신 응용 프로그램 아키텍처를 지원하는 고급 요청 라우팅을 제공합니다.

CLB - 쓸 이유가 없습니다. 