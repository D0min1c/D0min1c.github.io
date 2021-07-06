---
title: "비기너라면 알아두어야 할 AMI"
date: 2021-07-05
description: Sample post with multiple images, embedded video ect.
menu:
  sidebar:
    name: 1-2 AMI의 모든 것
    identifier: AWS_AMI
    parent: AWS_Beginners😆
    weight: 3
hero: images/host_thumnail.jpg
---
AWS를 사용하시려는 모든 유저분들이 간단히 보기에 좋은 내용입니다. AWS Docs는 너무 어려워...! 그렇다면 들어오세요.

<!--more-->

## 배경
AWS를 시작하시는 분들을 위해 간단하게 AMI에 대해 정리해보았습니다.

### AMI 개념 (Amazon Machine Image) (AKA. AMI)

![This is an image](images/ami.jpg)

Amazon 머신 이미지(AMI)는 인스턴스를 시작하는 데 필요한 정보를 제공합니다. 인스턴스를 시작할 때 AMI를 지정해야 합니다.
제공되는 템플릿 AMI를 사용하여 커스텀한 서버나 애플리케이션 인스턴스 자체를 AMI로 풀백업하여 보관하는것도 가능합니다.

### AMI 사용사례
모든 인스턴스의 시작은 AMI에서부터 시작합니다.
또한 AMI의 경우 EBS 스냅샷을 포함하여 풀백업 (OS셋팅까지도)되기에 백업해두는 용도로 많이 사용하게 됩니다.
AMI로 표준 이미지를 생성해두고 해당 이미지를 통해 여러 서버를 복제하거나 Autoscaling과 연동하여 사용할 수 있습니다.

![This is an image](images/ami_backup.jpg)


혹은 AMI를 리전 간 복제하여 다른 리전에, 다른계정에 공유하여 다른 계정 내 VPC에도 이미지를 불러 사용할 수 있습니다.

### AMI 요금
인스턴스 스토어 지원 AMI의 경우 AMI 스토리지 및 인스턴스 사용 및 Amazon S3에 AMI 저장에 대해 요금이 부과됩니다. 
Amazon EBS 기반 AMI를 사용하는 경우 인스턴스 사용, EBS 볼륨 스토리지 및 사용, AMI를 EBS 스냅샷으로 저장에 대해 요금이 부과됩니다.

### 결론 
결론적으로 AMI는 인스턴스의 풀백업 이미지입니다. 이를 통해 서버 규격에 대한 표준을 정하여두고 버저닝하여 관리하는 것이 용이합니다.
또한 만든 AMI를 AWS Marketplace로 판매할 수 있기도합니다.
