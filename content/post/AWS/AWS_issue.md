---
title: 아니 글쎄 갑자기 VM이 죽었다니까요? - AWS
categories:
  - "cloud"
tags:
  - "aws"
  - "public cloud"
---
AWS/GCP에서 일어나는 VM이 갑자기 뻗거나 리부팅되는 이유에 대해...

<!--more-->

## 배경
GCP 혹은 AWS를 사용하면서 종종 Host Error! 혹은 Instance check Failed를 경험한 적이 있으시죠?
우선 AWS를 기준으로 이슈에 대해 정리해보았습니다.

### AWS System Check failed
 다들 아시다시피 ec2 인스턴스는 **AWS에서 Iaas 방식으로 제공하는 서비스**이기에 사용자가 해당 ec2 인스턴스를 프로비저닝하게 될 경우 선택한 리전과 가용영역의 데이터 센터 내 host computer에서 hypervisor 위에 고객의 요청에 의해 insatnce를 생성하여 제공하는 방식입니다.

여기서 만약 hypervisor와 instance간의 Network이 끊긴다면..? 아니면 Host 장비에 불이 난다면...?

**와장창-!** 말 그대로 인스턴스가 와장창 납니다.

 이러한 갑작스러운 이슈로 인해 발생하는 AWS 하드웨어의 Maintenance Event 혹은 System checkfailed 이슈로 인해 발생하는 예기치않은 이슈에 대해 정리하고 사례에 맞춰 사용자들은 어떻게 대처해야하는지에 경험을 공유하고자 합니다.

## 종류
우선 AWS에서의 EC2 Event 종류와 Cloudwatch에서 확인할 수 있는 Status Check failed 지표는 아래와 같은 내용을 포함합니다.
  - AWS Event 
    - Instance stop : 예약된 시간에 인스턴스가 중지됩니다. 인스턴스를 다시 시작하면 **새 호스트로 마이그레이션됩니다.** 
    
    이러한 유형은 Amazon EBS가 지원하는 인스턴스에만 적용됩니다.
    
    - Instance retirement 
    - Instance reboot
    - System reboot 
    - System maintenance 

### Instance stop 

: 예약된 시간에 인스턴스가 중지됩니다. 인스턴스를 다시 시작하면 **새 호스트로 마이그레이션됩니다.** 

Instance stop이 일어나는 경우는 대게  Host hardware의 폐기 날이 결정된 것입니다.

이 경우, Maintenance 일정 이전에 스스로 stop/start를 진행하여 해소 할 수 있습니다.

두 눈을 크게 뜨고 정확히 봐야합니다.  Instance reboot이 아닌, Stop인 것을요.

Stop되어있는 인스턴스는 어떤 트리거가 발생하기 전까지는 결코 혼자서 벌떡 일어나진 않습니다.

    이러한 유형은 Amazon EBS가 지원하는 인스턴스에만 적용됩니다.

### Instance retirement 
: 예약된 시간에 인스턴스가 Amazon EBS에서 지원되는 경우 중지되거나 인스턴스 스토어에서 지원되는 경우 종료됩니다.

말 그대로 EBS 지원 인스턴스는 stop, 인스턴스스토어 지원 인스턴스는 terminate되는 이벤트입니다.

아직 발생해본적이 없어서 패스하겠습니다. (뭐..다를거 있겠냐만..)

### System maintenance 

: 예약된 시간에 네트워크 또는 전력 유지 관리로 인스턴스가 일시적인 영향을 받을 수 있습니다

네트워크 유지 관리 시에는 예약된 인스턴스의 네트워크 연결이 잠시 동안 끊어집니다. 유지 관리가 완료되면 인스턴스의 네트워크 연결이 평소처럼 복구됩니다.

전력 유지 관리 시에는 예약된 인스턴스가 잠시 동안 오프라인 상태로 전환되었다가 재부팅됩니다. 재부팅 이후에도 인스턴스의 모든 구성 설정은 그대로 유지됩니다.

### Instance reboot 
: 예약된 시간에 인스턴스가 재부팅됩니다.

하이라이트 입니다. 과연 인스턴스 리붓과 시스템 리붓의 차이는 

### System reboot 
: 예약된 시간에 인스턴스의 호스트가 재부팅됩니다.





.

