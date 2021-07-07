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

3가지 종류의 로드밸런스를 지원하고 최근 1개가 추가되어 총 4가지의 로드밸런서 타입을 지원합니다.
- CLB (Classic Load Balancer)
제일 오래된 타입으로 기본적으로 4계층, 7계층의 LB를 모두 지원하며, 타겟 그룹아닌 인스턴스를 등록하여 백엔드로 지정합니다.

- ALB (Application Load Balancer)
7계층 L7 로드밸런서로 Request에 대한 동작은 아래와 같습니다.
  1. 우선 순위에 따라 리스너 규칙을 평가하여 적용할 규칙을 결정
  2. 규칙 작업의 대상 그룹에서 대상을 선택합니다.
  3. 리스너의 규칙에 의해 애플리케이션 트래픽의 콘텐츠를 기반으로 다른 대상 그룹에 요청을 라우팅할 수 있습니다.
  (대상이 여러 개의 대상 그룹에 등록이 된 경우에도 각 대상 그룹에 대해 독립적으로 라우팅이 수행됩니다.) 
  
기본 라우팅 알고리즘은 라운드 로빈이며 요청이 적은 백엔드로 요청을 보내는 라우팅 알고리즘으로 수정이 가능합니다.
또한 방화벽 정책을 통해 백엔드까지의 요청을 로드밸런서에서 필터링할 수 있습니다.

- NLB (Network Load Balancer)
4계층 L4 로드밸런서로 Request에 대한 동작은 아래와 같습니다.
  1. 

- GLB (Gateway Load Balancer)


기본적으로 ELB에서는 node의 개념이 있습니다.