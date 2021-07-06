---
title: "서버가 갑자기 박살나셨습니다. -GCP"
date: 2021-06-23
description: Sample post with multiple images, embedded video ect.
menu:
  sidebar:
    name: GCP 이슈 (갑.서.박)
    identifier: GCP(issue-1)
    parent: GCP
    weight: 10
hero: images/host_thumnail.jpg
---
GCP에서 발생하는 host error 외 이상한 경험에 대해 간략하게 적어두었습니다.
<!--more-->

## 배경
지난번 AWS이슈에 이어 GCP에서의 이슈에 대해 정리해보았습니다.
### 우선 Host error에 대해
GCP의 Compute Engine은 소프트웨어 또는 하드웨어 업데이트와 같은 호스트 시스템 이벤트가 발생하더라도 가상 머신 인스턴스가 계속 실행될 수 있게 해주는 라이브 마이그레이션 기능을 제공합니다.

Compute Engine은 사용자가 VM을 재부팅할 필요 없이 동일 영역에서 실행 중인 인스턴스를 또 다른 호스트로 라이브 마이그레이션합니다.

 이렇게 해서 Google은 사용자의 VM에 영향을 주지 않으면서도 인프라를 보호하고 안정적인 상태로 유지하는 데 반드시 필요한 유지보수를 수행할 수 있습니다. (GPU가 연결된 인스턴스는 불가능하다는 점 !)

GCP도 역시 Host Error가 발생합니다. 자동으로 올라오지만, 마찬가지로 모니터링 및 트리거를 걸어 Reboot try 로직을 걸어두심이 좋습니다.

다시한번 말하지만 세상에 100% 안죽는 서버는 없습니다.  ~~(사장님 이젠 죽을거 같습니다)~~

그렇게 때문에 항상 DR 프로세스에 대한 준비는 해두어야합니다. 이것은 클라우드 위의 인프라도 마찬가지 입니다.

클라우드는 공동 책임 모델 등 SLA를 명확하게 보여주기에 어느정도의 가용성을 보장하는지가 알려줍니다.

가령 99.999%를 보장하는 서비스는 최소 1년에 0.001%의 보장할 수 없는 서비스 중단시간이 발생할 수 있다는 소리입니다.

31536초면 약 9시간쯤 되겠네요.

역시나 운영비를 늘려 서버를 고가용성으로 구성하는것이 가장 바람직하지만 백업과 모니터링에 집중하여 일정부분 타협이 가능할 것 같습니다.

## 실제로 이런 일이 있습니다. (가용영역의 리소스 부족)

GCP, AWS를 막론하고 내가 원하는 가용영역에 리소스가 부족하는 경우가 있습니다.
이때 서버 생성이 불가함은 물론이거니와, 이미 있던 내 서버들이 갑작스레 Host error등으로 죽었는데, host hardware의 리소스가 부족해서
VM migration이 발생하지 않으면 어떻게 될까요? 어떻게 되긴 뭘 어떻게 됩니까. 큰일났지 !_!_!__!_!😫😫😫

### 해결 방안 -1 (zone을 바꿔보자)
쉽게 생각하면 그렇습니다. VM들을 다른 가용 가능한 zone으로 이동시키면 됩니다.

AWS에서는 스냅샷 혹은 AMI를 통해 인스턴스를 재 구성하였다면 GCP에서는 GCP CLI에서 영역 간 인스턴스 이동을 지원합니다.
옮기는 동안 서버에서 생성한 인스턴스와 디스크의 일부 속성이 변경됩니다.

[변경되는 속성](https://cloud.google.com/compute/docs/instances/moving-instance-across-zones?hl=ko#moving-an-instance-automatically)

이 방법으로 이관하지 못한 이유가 있습니다.
  1. 비관리형 인스턴스 그룹를 타겟으로한 LB를 운영하고 있습니다.

    - 비관리형 인스턴스 그룹은 영역끼리 그루핑해야하기에 같은 그룹의 인스턴스를 전부 이관해야합니다.
    그렇다고 관리형 인스턴스 그룹은 되느냐? 그것도 아닙니다. 관리형 인스턴스 그룹의 구성일 경우 서브넷 변경까지 필요합니다.

  2. UEFI 기반 VM는 위 CLI를 통해 자동으로 존 변경이 불가합니다.

    - 수동으로 진행하면 됩니다.
      대략적인 작업 흐름은 아래와 같습니다. (수동, 자동 동일)
      1. 원본 인스턴스에 연결된 영구 디스크의 스냅샷을 만듭니다.
      2. 대상 영역에 영구 디스크의 사본을 만듭니다.
      3. 외부 및 내부 IP 주소:
      동일한 리전 내의 영역 간에 인스턴스를 이동하고 임시 IP 주소를 보존하려면 인스턴스에 할당된 임시 IP 주소를 고정 IP 주소로 임시 승격한 다음 대상 영역에 만든 새 VM 인스턴스에 할당합니다.
      인스턴스를 리전 간에 이동하는 경우에는 VM 인스턴스의 다른 IP 주소를 선택해야 합니다.
      4. 대상 영역에 새 인스턴스를 만들고 부팅합니다. 서로 다른 리전 간에 이동하는 경우에는 새로운 인스턴스의 새로운 서브네트워크도 선택해야 합니다.
      5. 새로운 영구 디스크를 새로운 인스턴스에 연결합니다.
      6. 새 인스턴스에 외부 IP 주소를 할당합니다. 필요한 경우 주소의 수준을 다시 임시 외부 IP 주소로 낮춥니다.
      7. 스냅샷, 원본 디스크, 원본 인스턴스를 삭제합니다.
      
  3. 두개 이상의 볼륨을 마운트 하고 있습니다.

결론적으로 우리는 비관리형 인스턴스 그룹으로 그룹핑해놓은 두개이상의 볼륨이 마운트된 UEFI 기반의 VM 구성이기에 CLI를 통해 이관은 불가했습니다. 또한 당장 해당 리전을 사용하는 vm의 갯수가 수 백대는 되기에 현실적으로 어려움이 있습니다.

### 해결 방안 -2 (타입을 바꿔보자)
그래서 이 방법을 택했습니다.
모든 인스턴스에 기본적으로 인스턴스 reboot이나 host error 발생 시, 알람을 걸어두었습니다.
혹시라도 Host error 이후 정상화가 되지 않는 특정 존의 인스턴스 경우 이벤트로그를 통해 원인을 확인하고 만약 리소스 부족이 원인이라면 
해당 존에서 가용할 수 있는 타입으로 인스턴스를 변경하는 것으로 처리하는 것이지요.

## 결론

그래서 결론이 없는 문제입니다. 사고를 미연에 방지하고자한다면 해당 리전/가용영역에 미리 리소스를 선점해두는 방법밖에 없을 것 같습니다.