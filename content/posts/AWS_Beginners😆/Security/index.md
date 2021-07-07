---
title: "비기너라면 알아두어야 할 NACL과 SG"
date: 2021-07-05
description: Sample post with multiple images, embedded video ect.
menu:
  sidebar:
    name: ㄴ NACL, SG의 모든것
    identifier: AWS_security
    parent: AWS_Beginners😆
    weight: 5
hero: images/host_thumnail.jpg
---
AWS를 사용하시려는 모든 유저분들이 간단히 보기에 좋은 내용입니다. AWS Docs는 너무 어려워...! 그렇다면 들어오세요.

<!--more-->

## 배경
AWS를 시작하시는 분들을 위해 간단하게 NACL과 SG에 대해 설명합니다.

### NACL 개념 (Network access control list)
VPC와 함께 기본적으로 제공되는 NACL은 VPC 내 서브넷 내부와 외부의 트래픽을 제어하기 위한 계층입니다.
서브넷은 한번에 한개의 ACL만을 적용받을 수 있습니다만, ACL은 여러 서브넷에 적용할 수 있습니다.
기본 제공되는 NACL의 경우 모든 IN/Out bound 트래픽에 대해 허용하고 있습니다.

### SG 개념 (SecurityGroup)
NACL과 다르게 SG는 각 인스턴스의 방화벽 역할을 합니다. 
기본적으로 모든 Inbound 트래픽에 대해 ALL deny 상태로, Outbound 트래픽에 대해 ALL allow 상태로 생성됩니다.

NACL과 SG의 가장 큰 차이는
```
Stateless인가, Stateful인가
```

### Stateless, Stateful?

#### Stateless
상태 비저장으로 트래픽에 대한 정보를 저장하지 않습니다.
요청과 응답은 트래픽의 상태와 상관없이 각각 inbound 와 outbound 규칙을 따릅니다.

#### Stateful
상태 저장으로 트래픽에 대한 정보를 저장합니다.
요청과 응답은 Outbound 규칙과 상관없이 Inbound 트래픽을 허용했다면 이에 대한 응답은 허용됩니다.


예를 들어, NACL을 통해 80 port에 대한 Inbound 허용을 하였다면, 80에 대한 요청을 서버는 받을 수는 있어도 그에 대한 response를 내보낼 순 없습니다. 이 경우 휘발성 포트에 대한 outbound 허용이 되어있어야 통신이 가능합니다.

허나 SG를 통해 80 port에 대한 Inbound 허용을 하였다면, Outbound 정책이 어찌됐건 들어온 요청에 대한 Response는 언제든 클라이언트에게 송신됩니다.

#### 결론
NACL, SG에 대해 알아봤습니다. 두 정책 모두 활용하시는 방법은 권장해드리지 않습니다.
SG만으로도 충분하기도하고 NACL까지 세부적으로 설정해뒀는데 나중에 이슈 발생하면 트러블 슈팅을 하는데 복잡도가 증가하고
관리포인트가 너무 많아집니다.
둘 중 하나의 정책만으로도 충분히 기능을 발휘할 수 있습니다.

AWS의 NACL과 SG에 대해 알아봤습니다.

