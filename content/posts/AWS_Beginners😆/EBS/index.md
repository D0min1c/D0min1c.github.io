---
title: "비기너라면 알아두어야 할 Storage"
date: 2021-07-05
description: Sample post with multiple images, embedded video ect.
menu:
  sidebar:
    name: ㄴ EBS의 모든 것
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

### 모니터링 및 백업
EBS는 스냅샷 기능을 통해 백업을 지원합니다.
EBS 스냅샷은 특정 시간에 찍은 볼륨의 이미지 사본입니다. 최초의 스냅샷, 즉 이미지 사본은 전체 백업이고, 이후의 스냅샷은 블록-수준의 증분 백업방식을 사용합니다.


![This is an image](images/ebs_snap.jpg)

AWS 제공해주는 스냅샷 이해에 도움이되는 이미지입니다.
해석해보자면 이렇습니다.

1. 최초 10GB의 볼륨을 스냅샷합니다 (최초 이기에 풀백업 10GB)
2. 이후 10GB의 데이터 중 4GB의 데이터가 변경되었기에 이떄 스냅샷을 생성하면 10GB를 전부 생성하는 것이 아닌, 변경된 4GB에 대한 스냅샷만을 생성합니다. (이전에 생성해둔 스냅샷에서 변경사항이 없는 6GB의 데이터를 유지합니다.) 총 용량은 여전히 10GB입니다.
3. 10GB에서 2GB의 볼륨을 추가하여 총 볼륨량이 12GB가 되었습니다. 마찬가지로 이 2GB에 대해서만 증분 복제합니다.

결론적으로 3의 과정을 거치면서 생긴 스냅샷은 최초 스냅샷 10GB+2번에서 변경된 데이터 4GB+ 3번에서 추가된 2GB =16GB가 스냅샷으로 백업되어 있는 것 입니다.

그렇기에 중복되는 데이터의 백업을 줄여 총 백업 데이터 량에서 비용 효율적인 백업을 할 수 있습니다.
이 이미지는 S3에서 하나의 객체로 저장되지만, 실제 사용자가 볼수 없는 AWS 관리형 S3에 저장됩니다.


EBS를 사용함에 있어 주 모니터링 메트릭은 저의 경우 Read/WriteOps와 QueueLength를 가장 먼저 살펴봅니다.
실제 발생하는 IOPS의 양과 처리되지 못하여 Queue에 쌓이는 요청이 늘어날 경우 볼륨의 크기를 키워 IOPS를 늘리거나 타입을 변경하여 해소하곤 했던 기억이 있습니다. (현재는 비용보단 성능에 집중하여 모든 서비스 볼륨은 io2로 사용하고 있습니다.)

### 결론
AWS Storage에 대해 살펴보았습니다. 이밖에도 I/O 버스트 크레딧과 IOPS 계산방법 등이 있으나, 여타 다른 블로그에서 잘 설명되어왔기에
비교적 서비스 핵심적인 내용만 메모해두었습니다. 