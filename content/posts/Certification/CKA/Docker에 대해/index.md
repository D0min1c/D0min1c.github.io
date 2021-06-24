---
title: "Docker에 대해"
date: 2020-06-20
description: Sample post with multiple images, embedded video ect.
menu:
  sidebar:
    name: Docker에 대해
    identifier: Docker에 대해
    parent: CKA
    weight: 10
hero: images/CKA_thumnail.jpg
---
GCP 환경에서 docker를 사용하여 이미지를 빌드하고 실행해보고, 

GCR(Google Container Registry)에 게시해봅시다.
<!--more-->

## 배경
Docker 컨테이너 빌드, 실행과 Docker Hub 및 Google Container Registry에서 Docker 이미지 가져오기

그리고 Dockerfile을 통해 이미지를 생성하는 법에 대해 간단하게 확인해봤습니다.

Docker Network는 k8s(Network) 포스팅을 참고 부탁드립니다.

###  Hello World

GCP에서 제공하는 Cloud shell을 열어 Hello world 컨테이너를 실행합니다.

```
docker run hello-world

<--명령어 결과-->
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
b8dfde127a29: Pull complete
Digest: sha256:9f6ad537c5132bcce57f7a0a20e317228d382c3cd61edae14650eec68b2b345c
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

```

이 컨테이너는 shell에 "Hello from Docker!"라는 메시지를 반환합니다. 결과는 간단하지만 결과가 나오기 전에 이미지를 불러오는 과정을 확인해봅시다.

로컬이미지에 Hello world이미지가 없기에 docker deamon은 docker hub라는 public registry에서 해당 이미지를 pull하여 컨테이너를 생성하고 실행하는 과정을 통해 위 메세지를 반환합니다.

docker hub에서 가져온 이미지를 확인해봅시다.

```
docker images

<--명령어 결과-->
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    d1165f221234   3 months ago   13.3kB
```

여기서 알 수 있는 사실은 로컬에 이미지를 가지고 있지 않아도 docker deamon은 알아서 docker hub라는 public registry에서 이미지를 검색한다는 점입니다.

이미지를 로컬로 Pull하였기에 두번쨰 실행부터는 바로 로컬이미지를 통해 컨테이너를 생성, 실행할 수 있는 것 입니다.
위 이미지는 echo "Hello from Docker!"를 실행하고 사라졌네요 마치 k8s의 jobs같습니다. 실제로 Docker ps -a 를 통해 컨테이너를 확인해보면 컨테이너가 종료되었음을 알 수 있습니다.

```
docker ps -a

<--명령어 결과-->
CONTAINER ID   IMAGE         COMMAND    CREATED              STATUS                          PORTS     NAMES
d36437eeb64a   hello-world   "/hello"   About a minute ago   Exited (0) About a minute ago             sweet_pare
359b95d41706   hello-world   "/hello"   2 minutes ago        Exited (0) 2 minutes ago                  happy_ritchie
```
docker run을 시작할때 옵션으로 --name을 주면 뒤에 무작위로 붙는 Names 라벨을 설정해줄 수 도 있습니다.

### DockerFile build / run Test
Dockerfile을 통해 직접 이미지를 빌드할 수 있습니다. 간단하게 CentOS에 Httpd를 띄우는 Dockerfile을 작성합니다.

```
cat > Dockerfile <<EOF
# CentOS7를 사용합니다.
FROM centos:7

# 컨테이너에서 http 패키지를 설치해줍니다.
RUN yum -y install httpd 

# 현재 디렉토리의 내용을 컨테이너 내부로 복사합니다.
ADD /write-http.sh /write-http.sh

# 컨테이너에서 write-http.sh를 실행할 수있도록 권한을 변경합니다.
RUN chmod 755 /write-http.sh

# 서비스를 위해 컨테이너의 포트 80을 외부에 공개합니다.
EXPOSE 80

# 컨테이너가 시작될 때 write-http.sh 스크립트를 실행합니다.
ENTRYPOINT ["sh","/etc/write-http.sh"]

EOF
```
```
# write-http.sh 내용
#!/bin/sh
/usr/sbin/httpd -D FOREGROUND
```

이미지를 빌드하여 컨테이너에 띄워보면 아래와 같이 컨테이너가 생성됨을 알 수 있다.

```
docker build -t dominic-web:0.1 .

<--명령어 결과-->
Sending build context to Docker daemon  3.072kB
Step 1/6 : FROM centos:7.5.1804
 ---> cf49811e3cdb
Step 2/6 : RUN yum -y install httpd
 ---> Using cache
 ---> b04936c42e77
Step 3/6 : ADD /write-http.sh /write-http.sh
 ---> 79d1a552ea2a
Step 4/6 : RUN chmod 755 /write-http.sh
 ---> Running in 9041f099a392
Removing intermediate container 9041f099a392
 ---> 96ce83e42fce
Step 5/6 : EXPOSE 80
 ---> Running in eb46efbe7e84
Removing intermediate container eb46efbe7e84
 ---> b39def1a021e
Step 6/6 : ENTRYPOINT ["sh","/etc/write-http.sh"]
 ---> Running in 7e258ee5d665
Removing intermediate container 7e258ee5d665
 ---> d3924f695483
Successfully built d3924f695483
Successfully tagged dominic-web:0.1
```

```
docker images
<--명령어 결과-->
REPOSITORY    TAG        IMAGE ID       CREATED          SIZE
dominic-web   0.1        d3924f695483   40 seconds ago   353MB
```

이후 docker run command를 통해 이미지를 컨테이너에 실행시키면 된다.
```
docker run -d -p 80:80 dominic-web:0.1
```
-d 옵션을 안주면 Foreground에서 동작하기에 쉘을 움직을 수가 없으니, -d 옵션으로 도커 컨테이너가 background에서 동작될 수 있도록 합시다!

만약 다른 버전의 이미지를 생성하고 싶다면 간단합니다. 그냥 빌드할때, 태그를 달리해주면 된다.
```
docker build -t dominic-web:0.2 .
```
이후 빌드를하면 기존에 캐시되어있던 레이어는 넘어가고 변경사항이 생긴 레이어에서 수정이 발생함을 알 수 있을 것입니다.

### 결과 !

curl로 확인해보니, httpd default page가 쫙~~나오네요. 다음엔 index.html을 좀 추가해야될듯 ㅎㅎ;

간단하게 docker 테스트 진행해봤습니다.

이후 docker push를 통해 내 GCR로 이미지를 푸쉬하는 부분은 GCP Docs로 대체하겠습니다.

[이미지 내보내기](https://cloud.google.com/container-registry/docs/pushing-and-pulling)

