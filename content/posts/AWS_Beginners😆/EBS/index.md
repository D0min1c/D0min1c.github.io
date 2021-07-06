---
title: "비기너라면 알아두어야 할 Storage"
date: 2021-07-05
description: Sample post with multiple images, embedded video ect.
menu:
  sidebar:
    name: 1-1 EBS의 모든 것
    identifier: AWS_Storage
    parent: AWS_Beginners😆
    weight: 2
hero: images/host_thumnail.jpg
---
AWS를 사용하시려는 모든 유저분들이 간단히 보기에 좋은 내용입니다. AWS Docs는 너무 어려워...! 그렇다면 들어오세요.

<!--more-->

## 배경
AWS를 시작하시는 분들을 위해 간단하게 AWS 상 서비스되는 Storage 서비스들에 대해 정리해보았습니다.

### AWS Storage 개념 (ElasticBlockService) (AKA. EBS)
AWS에서는 여러 스토리지 서비스를 제공합니다. 그중 EC2를 생성함에 있어서 루트 디바이스가 되는 Elastic Block Storage에 대해 알아보고자합니다.
기본적으로 EBS는 EC2 에서 사용하도록 설계된 사용하기 쉬운 *원격* 블록 스토리지 서비스입니다.
EBS 볼륨은 AZ(가용 영역) 내에서 복제되기에 장애로부터 99.999%의 가용성을 보장합니다. 

(사실 99.999% 면, 연간 13초정도는 connection이 끊겨도 AWS에서는 책임이 없단 소립니다.)

### EBS Architecture

![This is an image](images/ebs.jpg)

큰 그림은 위와 같습니다.

Host computer 내 1대의 인스턴스가 올라와있고, 인스턴스 내 볼륨은 각 sda1,sdh,sdj,sdb가 마운트 되어있습니다.
- sda1 : Root Device Volume으로 EBS가 마운트되어있습니다.
- sdh  : Data volume으로 EBS가 마운트되어있습니다.
- sdj  : Data volume으로 EBS가 마운트되어있습니다.
- sdb  : Data volume으로 EBS가 아닌 Instance Store가 마운트되어있습니다. (!)

Insatnce Store는 Host Computer에 물리적으로 연결되어있는 디스크입니다. 휘발성 스토리지이기에  버퍼, 캐시, 스크래치 데이터 및 기타 임시 콘텐츠와 같이 자주 변경되는 정보의 임시 스토리지나 로드가 분산된 웹 서버 풀과 같은 여러 인스턴스상에서 복제되는 데이터에 가장 적합합니다.

비교 | EBS | InstanceStore
-----|------|------
데이터 보존 | 영구적 | 휘발성
분리| 가능 | 불가능
인스턴스 종료 시 | 유지 가능 | 유지 불가능

### EBS 성능 (ElasticBlockService) (AKA. EBS)
EBS 볼륨의 성능을 측정할 때는 관련된 측정 단위와 성능 계산 방법을 이해해야 합니다.
IOPS를 기준으로 성능을 좌우합니다. 아래 AWS에서 제공하는 성능비교 링크를 통해 각 유형이 어떤 타입과 성능을 제공하는지 확인할 수 있습니다.

[AWS-EBS 볼륨유형](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/ebs-volume-types.html)


### 프리티어
마지막으로 AWS에서는 신규 가입 고객에게 가입한 날로부터 12개월, 혹은 상시로 무료 제공하는 일부 서비스가 있습니다.
이를 "프리티어"라고 부르기는 합니다만, Free라고해서 방심하시면 요금폭탄을 맞기 딱 좋기에 서비스를 생성하기 전 요금을 꼭 확인해보고 생성하시길 바랍니다.

### 모니터링
EBS를 사용함에 있어 주 모니터링 메트릭은 저의 경우 Read/WriteOps와 QueueLength를 가장 먼저 살펴봅니다.
실제 발생하는 IOPS의 양과 처리되지 못하여 Queue에 쌓이는 요청이 늘어날 경우 볼륨의 크기를 키워 IOPS를 늘리거나 타입을 변경하여 해소하곤 했던 기억이 있습니다. (현재는 비용보단 성능에 집중하여 모든 서비스 볼륨은 io2로 사용하고 있습니다.)

### 결론
AWS Storage에 대해 살펴보았습니다. 이밖에도 I/O 버스트 크레딧과 IOPS 계산방법 등이 있으나, 여타 다른 블로그에서 잘 설명되어왔기에
비교적 서비스 핵심적인 내용만 메모해두었습니다. 