---
title: CKA를 취득하기 위해 알아두어야 할 것들
categories:
  - "CKA"
tags:
  - "k8s"
  - "CKA"
  - "Kubernetes"
---
2021-06-15부터 CKA 취득을 위해 이론적으로 공부한 내용들을 데일리로 기록하였습니다.
<!--more-->

## 배경
CKA라는 핑계를 통해, 쿠버네티스에 입문해보자

##  CKA

작년 11월쯤인가.. CKA할인소식을 듣고 냅다 구매했다가 까먹고 있었다.
이제 슬..만료일이 다가오는거 같으니, 준비해보자.

## 범위

총 5가지 과목을 준비해야한다. 참고로 모든 시험은 핸즈온으로 구성되어있다고 한다.

1. Cluster Architecture, Installation & COnfiguration (25%)
- 역할 기반 액세스 제어(RBAC) 관리
- Kubeadm을 사용하여 기본 클러스터 설치
- 고 가용성 Kubernetes 클러스터 관리
- Kubernetes 클러스터를 배포하기위한 기본 인프라 프로비저닝
- Kubeadm을 사용하여 Kubernetes 클러스터에서 버전 업그레이드 수행
- 기타 백업 및 복원 구현

2. Workloads & Scheduling (15%)
- 배포 및 롤링 업데이트 및 롤백 수행 방법 이해
- ConfigMaps 및 Secrets를 사용하여 애플리케이션 구성
- 애플리케이션 확장 방법 파악
- 강력한 자가 복구 애플리케이션 구현에 사용되는 기본 요소 이해
- 리소스 제한이 포드 스케줄링에 미치는 영향 이해
- 매니페스트 관리 및 일반적인 템플릿 도구에 대한 인식

3. Services & Networking (20%)
- 클러스터 노드의 호스트 네트워킹 구성 이해
- 포드 간의 연결 이해
- Cluster IP, NodePort, LoadBalancer 서비스 유형 및 엔드 포인트 이해
- 수신(Ingress) 컨트롤러 및 수신(Ingress) 리소스 사용 방법 파악
- CoreDNS 구성 및 사용 방법 이해
- 적절한 컨테이너 네트워크 인터페이스 플러그인 선택

4. Storage (10%)
- 스토리지 클래스, 영구 볼륨 이해
- 볼륨 모드, 액세스 모드 및 볼륨 회수 정책 이해
- 영구 볼륨 클레임 기본 이해
- 영구 스토리지로 애플리케이션 구성 방법 이해

5. Troubleshooting (30%)

- 클러스터 및 노드 로깅 평가
- 응용 프로그램 모니터링 방법 이해
- 컨테이너 stdout 및 stderr 로그 관리
- 응용 프로그램 오류 문제 해결
- 클러스터 구성 요소 오류 문제 해결
- 네트워킹 문제 해결

라이브하게 그날 공부한것들을 정리할 예정이다.

아래부터는 kubeadm으로 AWS EC2에 Kubenetes v1.21.1 설치 히스토리 기록 

```
1M2K 셋팅 
공통 설정

#####호스트 명 변경 및 호스트 추가#####

hostnamectl set-hostname k8s-*

/etc/hosts

172.31.4.180 k8s-master
172.31.40.213 k8s-worker-1
172.31.39.84 k8s-worker-2

##### selinux 모드확인 ######
getenforce
Disabled

##### Swap 메모리 확인 #####
free

##### 컨테이너 런타임 설치 (docker 예제) #####

yum install -y docker
systemctl start docker  && docker info | grep -i version
Server Version: 20.10.4
 Cgroup Version: 1
 containerd version: 05f951a3781f4f2c1911b05e61c160e9c30eaa8e
 runc version: 12644e614e25b05da6fd08a38ffa0cfe1903fdec
 init version: de40ad0
 Kernel Version: 4.14.232-176.381.amzn2.x86_64

##### 컨테이너 런타임 설치 후 cgroup 관리 주체를 systemd로 변경 #####
overlay2는 리눅스 커널 4.0 이상 또는 3.10.0-514 버전 이상을 사용하는 RHEL 또는 CentOS를 구동하는 시스템에서 선호하는 스토리지 드라이버 

tee /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}

EOF
####Docker 서비스에 대한 systemd 드롭 인 디렉토리 생성#####

mkdir -p /etc/systemd/system/docker.service.d

####데몬 리로드 및 restart [드라이버 셋팅값 확인]####
systemctl daemon-reload && systemctl restart docker && systemctl enable docker
docker info | grep -i driver
Storage Driver: overlay2
Logging Driver: json-file
Cgroup Driver: systemd

####iptables가 브릿지된 트래픽을 보도록 설정####

cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system

####kubeadm, kubelet, kubectl 패키지 설치를 위해 repo설정 ####
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=0
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

yum install -y kubelet kubeadm kubectl
sudo yum install -y tc
systemctl enable kubelet && sudo systemctl start kubelet
-------------------------------템플릿 생성 완료 및 1차 백업-------------------------
템플릿 AMI로 worker 노드 배포 [sudo kubeadm join 명령어로 Master node에 join]
-------------------------------Master node 셋팅---------------------------------
kubeadm init --pod-network-cidr 192.168.0.0/16 [Calico 를 CNI로 쓸예정이기에]

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

CNI설치(칼리코)
kubectl apply -f https://docs.projectcalico.org/v3.13/manifests/calico.yaml

```

결과 :

![This is an image](/img/k8s_join.jpg)

#### kubeadm 설치 (v1.21|calico)

|Node-hostname|CPU|Memory|Storage|
|------|------|----|----|
|k8s-master|2vcpu|4GB|20GB|
|k8s-worker-1|1vcpu|2GB|20GB|
|k8s-worker-2|1vcpu|2GB|20GB|

![This is an image](/img/kube_info.jpg)