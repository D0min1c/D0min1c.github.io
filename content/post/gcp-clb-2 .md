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



## TCP LB-Test










## TCP proxy LB-Test





