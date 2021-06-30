---
title: "세대를 교체중입니다. -AWS"
date: 2021-06-30
description: Sample post with multiple images, embedded video ect.
menu:
  sidebar:
    name: AWS이슈 (Host error)
    identifier: AWS(Host_issue)
    parent: AWS
    weight: 10
hero: images/host_thumnail.jpg
---
AWS EC2 세대교체에 대한 경험을 메모해두었습니다.
<!--more-->

## 배경
AWS에서 t2와 같은 구세대 인스턴스를 최신 세대의 인스턴스로 마이그레이션하는 고객들의 기억을 되살리며 메모해둡니다.

EC2 instance의 가상화 기술은 Xen PV 으로 시작하여 Xen HVM 을거쳐 Nitro 까지 발전을 거듭해왔습니다
AWS의 인스턴스는 크게 Xen 기반의 구세대 인스턴스와 Nitro 시스템 (KVM)기반의 신세대 인스턴스로 나뉩니다.

### 무슨 차이가 있는지?
짧고 굵게 설명하자면, 이전 세대 대비 저렴한 비용 그와 반대되는 성능적 차이가 있습니다.
메모리, CPU의 할당을 관리하고 대부분의 워크로드에있어서 베어 메탈과 거의 유사한 성능을 제공하는 하이퍼바이저라고 설명합니다.

5세대 인스턴스들은 모두 Nitro 시스템이 도입되어 있기에 이전 세대의 인스턴스와는 다른 드라이버들이 존재합니다.

- 로컬 NVMe 스토리지 – 새로운 C5d, M5d 및 베어 메탈 EC2 인스턴스에는 Xen 가상화 I3 및 F1 인스턴스에도 사용되는 Nitro 로컬 NVMe 스토리지 구성 요소가 포함됩니다. 이 구성 요소는 PCI 인터페이스를 통해 고속 로컬 스토리지에 대한 직접 액세스를 제공하며, 모든 데이터를 전용 하드웨어를 사용하여 투명하게 암호화합니다. 또한 스토리지 디바이스와 EC2 인스턴스를 하드웨어 레벨에서 분리하여 베어 메탈 인스턴스에서 로컬 NVMe 스토리지의 이점을 활용할 수 있도록 합니다.

- Nitro 보안 칩 – AWS 서버 설계에 포함되는 구성 요소로, 하드웨어 리소스를 지속적으로 모니터링하고 보호하며 시스템을 부팅할 때마다 독립적으로 펌웨어를 확인합니다.
- Nitro 하이퍼바이저 – 메모리 및 CPU 할당을 관리하고 대부분의 워크로드에 베어 메탈과 거의 유사한 성능(Netflix의 Brendan Gregg는 이 성능을 1% 미만으로 벤치마킹함)을 제공하는 대기 휴지 상태의 씬 하이퍼바이저입니다.
- 네트워킹 – 각 VPC(가상 프라이빗 클라우드) 내의 소프트웨어 정의 네트워크에 대한 하드웨어 지원, 향상된 네트워킹 및 탄력적 네트워크 어댑터(ENA)를 제공합니다.
- 탄력적 블록 스토리지 – CPU 집약형 암호화 작업을 포함한 하드웨어 EBS 처리 기능을 제공합니다.
스토리지, 네트워킹 및 보안 기능을 하드웨어로 이동하면 베어 메탈 및 가상 인스턴스 유형에서 다음과 같은 이점을 실현할 수 있습니다.
- 가상 인스턴스의 경우 하이퍼바이저의 역할이 크게 감소하므로 모든 호스트의 CPU 성능 및 메모리를 게스트 운영 체제에 제공할 수 있습니다.
- 베어 메탈 인스턴스는 하드웨어에 완벽하게 액세스할 수 있을 뿐 아니라 가상 EC2 인스턴스와 동일한 유연성 및 기능 세트(예: CloudWatch 지표, EBS 및 VPC)를 활용할 수 있습니다.

아래 AWS링크를 참고 부탁드립니다.

[AWS NitroSystem-1](https://aws.amazon.com/ko/blogs/korea/amazon-ec2-update-additional-instance-types-nitro-system-and-cpu-options/)
[AWS NitroSystem-2](https://aws.amazon.com/ko/ec2/nitro/)

구구절절한것들 말고 실질적으로 이전 세대에서 신세대 인스턴스로 옮기는데 확인이 필요한 부분은 아래 두가지 입니다.
1. Network
ENA를 도입했습니다. 기본적은 네트워크모듈인 ENA (Elastic Network Adapter)는 기존 ENI보다 향상된 네트워킹 성능을 제공합니다.
가령, 네트워크 지연시간을 줄이고, PPS 성능이 높아진 모듈이라고 생각하면 편할듯합니다.

2. Storage
Nitro 시스템 기반 인스턴스에서는 EBS 볼륨이 NVMe(Non-Volatile Memory express) 블록 디바이스로 표시됩니다.

NVMe는 PCIe 버스를 통해 작동 하므로 특성은 하드디스크가 아닌 고속 메모리에 가깝다고 할 수 있습니다.

NVMe 볼륨을 사용하기에 그에맞는 드라이버가 필요합니다.

### 옮기는데 필요한 것들
답은 전혀입니다. 간단하게 옮길 수 있습니다. 그전에 몇가지만 확인하면 됩니다.

- [check point] 
  1. 내 인스턴스의 버전 확인
   내 인스턴스의 uptime을 확인해야 합니다. 기본적으로 4세대 인스턴스에는 최신 세대 인스턴스 유형을 지원하는 드라이버 및 구성이  포함되어있지 않았습니다. 2018년 8월 이후의 인스턴스에는 최신 세대 인스턴스 유형을 지원하는 드라이버 및 구성이 포함됩니다. 
  
  그렇기에 필요하다면 드라이버들은 수동으로 설치해줘야 합니다.
  2. Window 인스턴스를 업그레이드하려는 경우
   인스턴스에서 어느 네트워크 드라이버가 실행되고 있는지 확인해야 합니다. PV 네트워크 드라이버는 사용자가 원격 데스크톱을 사용하여 인스턴스에 액세스할 수 있게 해줍니다. Windows Server 2008 R2부터 인스턴스는 AWS PV, intel Network Adapter 또는 Enhanced Networking 드라이버를 사용합니다. Windows Server 2003 및 Windows Server 2008의 인스턴스는 Citrix PV 드라이버를 사용합니다.

  3. Linux 인스턴스를 업그레이드 하는 경우
   ENA(Elastic Network Adapter) enaSupport 속성이 활성화되어있는지 확인해야합니다.

  4. Linux 인스턴스는 그냥 NitroInstanceChecks script 스크립트 실행하시는게 제일 편합니다.
  AWS에서 제공하는 스크립트를 실행하고 업그레이드를 위해 무엇이 필요한지 확인하면됩니다.

[NitroInstanceChecks링크](https://github.com/awslabs/aws-support-tools/tree/master/EC2/NitroInstanceChecks)

### 모든 변경사항을 발생시키기 전 AMI 풀백업은 필수
그럼에도 업그레이드가 모종의 이유로 실패할 경우를 대비하여 항상 백업본을 유지한 채 작업을 진행합시다.
실제로 점검시간 내 작업이 완료되지않아 이슈가 발생한 케이스가 있었습니다만, 당연히 될거라고 생각하여 백업본이 없었기에,
루트볼륨을 테스트인스턴스에 연결해서 수정한 사례가 있었습니다. 작업 시간이 두배..ㅠㅠ

### 특이사항
간혹 5세대 인스턴스의 NVMe에서 timeout에 의해 I/O작업이 멈춘 이슈도 있었습니다.
대충 이런 순서로 장애가 발생했던 것으로 기억합니다. (하도오래되서 ㅎㅎ;)
Instance Hardware 이슈 발생 -> 이슈타임 간 EBS 이슈 발생 -> NVMe Timeout ->Instance Hardware 이슈 해소 -> NVMe timeout으로 인한 부팅 실패 -> 상태유지 -> 담당자 아침에 출근하여 리붓으로 해소 

일단은 부트파라미터에서 nvme.io_timeout값을 최대값으로 변경하여 이후 이슈는 없었지만 
악재의 악재가 겹치는 상황이였기에 당시 꽤나 당황스러운 이슈였습니다.

뭔가 더 있었던거 