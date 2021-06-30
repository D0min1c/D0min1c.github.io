---
title: "k8s(Volume)"
date: 2021-06-29
description: Sample post with multiple images, embedded video ect.
menu:
  sidebar:
    name: k8s(Volume)
    identifier: k8s(Volume)
    parent: Kubernetes
    weight: 10
hero: images/CKA_thumnail.jpg
---
CKA 취득을 위해 이론적으로 공부한 내용들을 간단하게 기록하였습니다.
<!--more-->

## 배경
컨테이너의 파일은 호스트의 파일 시스템에 마운트되어 있다. 컨테이너는 이미지로부터 만들어지므로 크게 다음의 호스트 파일 경로에 마운트된다.

- 이미지의 파일 내용을 저장하는 호스트 파일 경로(이미지는 실제로 레이어로 구성되므로 여러 경로 사용)
- 컨테이너 구동 이후 변경 사항을 저장하기 위한 호스트 파일 경로

이 외에 --mount 옵션을 이용해서 호스트 파일 시스템이나 도커 볼륨에 마운트 할 수 있다.

docker inspect 명령어를 사용하면 호스트 파일 시스템에 마운트된 경로를 확인할 수 있다.

그러면 쿠버네티스는 어떤 개념의 볼륨들이 있고, 어떻게 사용하는지 궁금해졌다.

### 도커 볼륨 
컨테이너 내의 디스크는 임시 디스크이다. 그렇기에 컨테이너가 터질때마다 컨테이너 내 데이터가 사라질 수 밖에 없다.

그렇기에 도커 볼륨을 이용해서 host directory에서 데이터를 관리하는 방법이 생겼다.
이렇게 되면 컨테이너를 지워도 도커 볼륨과 컨테이너를 연결한다면 데이터는 그대로 유지할 수 있게되었다.
그렇지만 결국 도커 볼륨을 사용한다는 것은 도커에 의해 관리된다는 걸 뜻할 수 있다 (어떤 컨테이너에 연결되어있는지 등 관리가 편하다.)
```
docker volume create
```
위 명령어로 볼륨을 생성하면 생성된 볼륨은 호스트 디렉토리인 /var/lib/docker/volumes/ 에 저장된다.

말고 바로 Host bind Mount하여 사용하는 방식도 있다.
이 방법은 기능적으로는 docker volume과 유사하지만 도커의 관리 없이 Host Directory와 마운트를 하다 보니

컨테이너에서 호스트의 파일 시스템에 접근하여 컨테이너에 지정된 파일이 아닌 다른 파일을 삭제/추가/수정할 수 있다는 것이 단점이다.
 
이 기능은 호스트 시스템의 비 Docker 프로세스에 영향을 주는 것을 포함하여 보안에 영향을 미칠 수 있기에 권장하지 않고있다.

이외에도 메모리에 데이터를 저장하는 tmpfs 마운트 방식도 있다. 마치 쿠버의 secret처럼 메모리에 저장되며 휘발성을 따라간다.

여기까진 도커의 볼륨 개념을 정리했다면 쿠버에서는 어떤 유형의 볼륨을 지원하는지 체크해보자.

### k8s 볼륨
파드는 여러 볼륨 유형을 동시에 사용할 수 있다. 임시 볼륨 유형은 파드의 수명을 갖지만, **퍼시스턴트 볼륨**은 파드의 수명을 넘어 존재한다. 

파드가 더 이상 존재하지 않으면, 쿠버네티스는 임시(ephemeral) 볼륨을 삭제하지만, 
퍼시스턴트(persistent) 볼륨은 삭제하지 않는다. 볼륨의 종류와 상관없이, 파드 내의 컨테이너가 재시작되어도 데이터는 보존된다.

볼륨을 사용하려면, .spec.volumes 에서 파드에 제공할 볼륨을 지정하고 .spec.containers[*].volumeMounts 의 컨테이너에 해당 볼륨을 마운트할 위치를 선언한다.

#### 퍼시스턴트 볼륨과 퍼시스턴트 볼륨 클래임
퍼시스턴트볼륨 (PV)은 관리자가 프로비저닝하거나 스토리지 클래스를 사용하여 동적으로 프로비저닝한 클러스터의 스토리지이다. 노드가 클러스터 리소스인 것처럼 PV는 클러스터 리소스이다. PV는 Volumes와 같은 볼륨 플러그인이지만, PV를 사용하는 개별 파드와는 별개의 라이프사이클을 가진다. 이 API 오브젝트는 NFS, iSCSI 또는 클라우드 공급자별 스토리지 시스템 등 스토리지 구현에 대한 세부 정보를 담아낸다.

퍼시스턴트볼륨클레임 (PVC)은 사용자의 스토리지에 대한 요청이다. 파드와 비슷하다. 파드는 노드 리소스를 사용하고 PVC는 PV 리소스를 사용한다. 파드는 특정 수준의 리소스(CPU 및 메모리)를 요청할 수 있다. 클레임은 특정 크기 및 접근 모드를 요청할 수 있다(예: ReadWriteOnce, ReadOnlyMany 또는 ReadWriteMany로 마운트 할 수 있음. AccessModes )

정리해보자면 퍼시스턴트 볼륨은 그냥 디스크 그 자체이다. 어떤 파드에 어느정도의 볼륨을 파티셔닝할지는 퍼시스턴트 볼륨 클래임을 통해 생성하고 접근 방식을 지정한다.

퍼시스턴트볼륨클레임(PersistentVolumeClaim)을 사용하도록 파드를 설정해보자.

### PV-PVC 정적 프로비저닝

그전에 우선 /mnt/data에 index.html파일을 하나 생성해두었다.
```
vi test-pv.yaml
----------------------------
apiVersion: v1
kind: PersistentVolume
metadata:
  name: test-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```
퍼시스턴스 볼륨 구성 내용은 위와 같다. (10Gi, R/W_Once, Path= /mnt/data)
스토리지 클래스 네임을 정의하여 퍼시스턴트볼륨클래임의 바인딩을 사는데 사용한다.

일단 PV를 생성하고 조회해보자.

```
kubectl apply -f test-pv.yaml
kubectl get pv
<--결과값-->
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
test-pv-volume   10Gi        RWO            Retain           Available           manual                  1m
```
PV를 조회했더니 뭔가 항목이 많이보인다. 각 항목이 의미하는 바는 아래와 같다.

  1. NAME             : PV 명
  2. CAPACITY         : PV 용량
  3. ACCESS MODES     : 접근 모드 설정 (단일노드 r/w/멀티노드 r,r/w)
  4. RECLAIM POLICY   : 회수 정책 (retain, Recycle, Delete)
  5. STATUS           : 상태
  6. CLAIM            : 현재 바인딩되어있는 클레임
  7. STORAGECLASS     : 특정 클래스의 PV를 가지면 해당 클래스에 요청하는 PVC만 바인딩된다.

```
vi test-pv-claim.yaml
----------------------------
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```
퍼시스턴스 볼륨 클래임 구성 내용은 위와 같다. (5Gi)
마찬가지로 퍼시스턴트 볼륨 클래임을 생성하고 조회해보자.

```
kubectl apply -f test-pv-claim.yaml
kubectl get pvc
<--결과값-->
NAME            STATUS   VOLUME           CAPACITY   ACCESS MODES   STORAGECLASS   AGE
test-pv-claim   Bound    test-pv-volume   5Gi        RWO            manual         7s
```

다시 pv를 조회해보면, Status와 CLAIM이 업데이트된걸 볼 수 있다.
```
kubectl get pv
<--결과값-->
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS   REASON   AGE
test-pv-volume   5Gi        RWO            Retain           Bound    default/test-pv-claim   manual                   1m

```

그럼 이제 생성한 클레임을 파드에 붙여보자.

```
apiVersion: v1
kind: Pod
metadata:
  name: test-pv-pod
spec:
  volumes:
    - name: test-pv-storage
      persistentVolumeClaim:
        claimName: test-pv-claim
  containers:
    - name: test-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: test-pv-storage
```
```
kubectl apply -f test-pv-pod.yaml
kubectl get pod test-pv-pod
<--결과값-->
윽, 캡쳐를 못했다. 혹여 파드가 pending상태라면 kubectl describe를 통해 파드의 상태를 확인해보면 로그가 딱 보일것이다. 
```
exec 명령어로 쉘에 접근하여 실제 hostpath(/mnt/data)볼륨으로부터 index.html를 제공받았는지 체크해보자.
```
kubectl exec -it test-pv-pod -- /bin/bash
  curl http://localhost/
   Hello from Kubernetes storage
  exit
```
host에 생성해둔 index.html이 잘불러와진다.


### 동적 프로비저닝
동적 볼륨 프로비저닝의 구현은 storage.k8s.io API 그룹의 StorageClass API 오브젝트를 기반으로 한다. 클러스터 관리자는 볼륨을 프로비전하는 볼륨 플러그인 (프로비저너라고도 알려짐)과 프로비저닝시에 프로비저너에게 전달할 파라미터 집합을 지정하는 StorageClass 오브젝트를 필요한 만큼 정의할 수 있다.

그러기 위해선 스토리 클래스 오브젝트를 사전에 생성해 두어야 한다.

#### 동적 프로비저닝 오브젝트 예시 (Googlecloud)
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
```
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```
#### 동적 프로비저닝 사용
 사용자는 PersistentVolumeClaim 오브젝트의 storageClassName 필드를 사용해야 한다. 
 이 필드의 값은 관리자가 구성한 StorageClass 의 이름과 일치해야 한다.

예를 들어 "fast" 스토리지 클래스를 선택하려면 다음과 같은 PersistentVolumeClaim 을 생성한다.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim1
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast
  resources:
    requests:
      storage: 30Gi
```

이런 식으로 클레임을 구성하면  자동으로 볼륨이 프로비저닝된다.
 
 그렇담 애초에 스토리지 클래스가 지정되지않은 모든 클래임들이 자동으로 동적 프로비저닝을 사용하면 관리하기 얼마나 편할까?
 master-cluster에서 설정이 가능하다.

 storageclass.kubernetes.io/is-default-class 어노테이션을 추가해서 특정 StorageClass 를 기본으로 표시할 수 있다. 기본 StorageClass 가 클러스터에 존재하고 사용자가 storageClassName 를 지정하지 않은 PersistentVolumeClaim 을 작성하면, DefaultStorageClass 어드미션 컨트롤러가 디폴트 스토리지 클래스를 가리키는 storageClassName 필드를 자동으로 추가한다.

 끝 ~_~