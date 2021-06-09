---
title: GCP-Cloud Load Balancer(2) 간단테스트
categories:
  - "cloud"
tags:
  - "gcp"
  - "google cloud"
---
GCP CLB 1편 포스팅에 이어 기능들을 모두 테스트해보고 검증한 내용입니다.

<!--more-->

## 배경
앞서 1편에서 각기 LB들의 서비스 특징과 workflow에 대해 정리했다면
이번 글은 실제 서비스에서 GCP LB를 사용해보면서 진짜(?) 기능이 동작하는지, 실제로 핸즈온하며 체크해볼 예정입니다.

```
목차
[1. HTTP/s LB 테스트]
[2. TCP LB 테스트]
[3.TCP porxy LB 테스트]
```
---

### **[인스턴스 그룹 생성]**
![This is an image](/img/ig.jpg)

기본적으로 백엔드 서비스는 UIG로 생성할 예정이며 단일 존 내부에 두대의 Web service를 띄워 확인해보겠습니다.

### **[상태 검사 생성]**
![This is an image](/img/hc_set.jpg)

우선 Http-80으로 상태 검사를 진행할 예정입니다. 고고

## HTTP/s LB-Test

### LB 생성
### **[HTTP Backend 생성]**
![This is an image](/img/Http_back.jpg)

### **[HTTP 검토 및 확인]**
![This is an image](/img/Http_last.jpg)

나머지 LB들도 동일하게 구성하면 되기에 configuration에 대한 내용은 여기까지...! Test 를 하러갑시다.

 (TCP LB는 그룹이 아닌 인스턴스 Pool로 등록 / TCP LB생성 간 단일리전으로 선택하면 TCP proxy가 아닌 TCP LB로 구성을 도와줍니다.)

### Test 1. Health check 확인 
![This is an image](/img/test_hc.jpg)

LB를 생성하고 나면 정상상태의 인스턴스가 확인되지 않는다. 당연히 80포트의 데몬이 떠있질 않으니, 헬스체크가 안되는겁니다.
우선 인스턴스 방화벽에서 35.191.0.0/16,130.211.0.0/22 80 Ingress를 허용해준 뒤, 얼른 서버로 접근하여 httpd를 띄워봅시다~

---

![This is an image](/img/http_index.jpg)

http 데몬을 올린 뒤, 설정해둔 상태검사 포인트만큼 수십초를 기다리면 금방 헬스체크가 되는 모습을 볼 수 있습니다. 
![This is an image](/img/http_hc_g.jpg)
![This is an image](/img/HTTP_test.jpg)

### Loadbalancing Test
부하분산이 정상적으로 이뤄지고 있는지 확인해봅시다.
F5를 다다다닷 누르기는 싫어서 curl을 실행하는 간단한 스크립트를 통해 확인해본 결과,
```
#!/bin/bash

number=0
while [ $number -le 50 ]
do
curl 34.98.79.168 >> $(date +"%y%m%d").txt
sleep 3
  ((number++))
done

```

Web 1번 22회, Web 2 29회가 확인됩니다. 왜 정확히 반반 나뉘지 않는걸까요?

**-# GCP의 설명 #-**

**"- 기본적으로 균등하게 부하를 분산하지만, 아주 작은 수의 요청은 균등하게 분배되지 않을 수 있습니다"**

...흠..🤣🤣🤣🤣🤣🤣 
### Access log 확인

그럼 실제 서버에서는 어떻게 요청이 오는지 Access.log를 통해 확인해보겠습니다.
![This is an image](/img/access_log_1.jpg)

HC 대역인 35.191.0.0/16으로 Healthcheck와 Client의 요청과 결과가 잘 찍혀있네요.

왜 HC 대역이 access log에 남는걸까요?
실제 부하 분산 트래픽의 소스 IP 주소는 상태 확인 프로브 IP 범위와 같기 때문입니다.

부하 분산기는 수신 연결을 종료한 후 부하 분산기에서 백엔드로 새 연결을 엽니다. 

이후 target proxy에서 URL map을 통해 백엔드 서비스쪽으로  reverse proxy하기에 백엔드 서비스는 실제 Source IP를 알 수 없는겁니다.

Source IP가 필요하다면 X-forward-for Header를 통해 확인 할 수 있습니다.

### HTTP backend session affinity

![This is an image](/img/cookie_2.jpg)

생성해놨던 HTTP LB의 백엔드 구성에서 cookie를 생성해줍니다.
생성된 쿠키 어피니티가 설정되면 LB가 첫 번째 요청에서 쿠키를 생성합니다. 
LB는 동일한 쿠키를 사용하는 각 후속 요청에서 같은 백엔드 VM 또는 엔드포인트로 요청을 전달합니다.

![This is an image](/img/cookie.jpg)

GCLB라는 cookie가 확인됩니다. 쿠키 값을 통해 고정 세션을 유지할 수 있습니다. 영원히 세션이 고정되는것은 아닙니다.
Timeout으로 설정해둔 (HTTP는 Keep AliveTimeout값이 제한 시간 값을 따라갑니다.) 30초 후엔 새로운 세션이 연결됩니다.

![This is an image](/img/cookie3.jpg)

위 결과, (31초 후 curl -i 동작) 31초마다 세션이 바뀌는걸 확인할 수 있습니다.
 

### HTTP backend keep-alive time



## TCP proxy LB-Test

## TCP LB-Test







