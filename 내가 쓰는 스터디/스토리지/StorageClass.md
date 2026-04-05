## 목차

- [1장 StorageClass 란](#1장-storageclass-란)
  - [1️⃣ StorageClass란](#1️⃣-storageclass란)
  - [2️⃣ 한줄 정리](#2️⃣-한줄-정리)
  - [3️⃣ 왜 필요한가](#3️⃣-왜-필요한가)
  - [4️⃣ 전체 구조](#4️⃣-전체-구조)
  - [5️⃣ StorageClass YAML (전체)](#5️⃣-storageclass-yaml-전체)
  - [13. Dynamic Provisioning 내부 흐름](#13-dynamic-provisioning-내부-흐름)
  - [14. 실제 동작 흐름](#14-실제-동작-흐름)
  - [15. CSI 호출](#15-csi-호출)
  - [16. 실제 Linux에서 일어나는 일](#16-실제-linux에서-일어나는-일)
  - [17. StorageClass 확인](#17-storageclass-확인)
  - [18. 전체 구조 (정리)](#18-전체-구조-정리)
- [2장 nfs 용 StorageClass 만들기](#2장-nfs-용-storageclass-만들기)
  - [준비물](#준비물)
  - [1️⃣ NFS Provisioner 설치](#1️⃣-nfs-provisioner-설치)
  - [2️⃣ NFS StorageClass](#2️⃣-nfs-storageclass)
  - [3️⃣ PVC 생성](#3️⃣-pvc-생성)
  - [4️⃣ NFS 사용하는 Pod](#4️⃣-nfs-사용하는-pod)

---

# 1장 StorageClass 란

## 1️⃣ StorageClass란

```
StorageClass는 어떤 방식으로 스토리지를 생성할지 정의하는 정책이다.

즉
PVC → StorageClass → CSI Driver → 실제 Volume 생성
```

## 2️⃣ 한줄 정리

```
PV를 자동 생성하는 정책
```

## 3️⃣ 왜 필요한가

예전 Kubernetes (Static provisioning)

관리자가 먼저 PV 생성

```
kind: PersistentVolume
```

그리고 PVC 생성

```
kind: PersistentVolumeClaim
```

문제

```
1. 관리자가 PV를 미리 만들어야 함
2. 자동화 불가능
3. Cloud 환경에 부적합
```

그래서 나온 것이

```
Dynamic Provisioning

PVC 생성하면 자동으로 PV 생성
```

이때 사용하는 것이

```
StorageClass
```

## 4️⃣ 전체 구조

전체 구조

```
Pod
 │
 ▼
PVC
 │
 ▼
StorageClass
 │
 ▼
External Provisioner
 │
 ▼
CSI Driver
 │
 ▼
Cloud Disk / NFS / Ceph
```

## 5️⃣ StorageClass YAML (전체)

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass

metadata:
  name: fast-ssd
  namespace: default
  labels: {}
  annotations: {}
  finalizers: {}

# 어떤 드라이버가 볼륨을 생성할지 (Volume 생성하는 driver)
# kubernetes.io/aws-ebs (legacy)
# csi.aws.com
# csi.ceph.com
# csi.nfs.com
provisioner: csi.aws.com

# Driver에게 전달할 옵션
# [AWS]
# type: gp3
# iops: 3000
#
# [Ceph]
# pool: kube-pool
#
# [NFS]
# server: 10.0.0.1
# path: /data
parameters:
  type: gp3
  fsType: ext4

# PVC 삭제 후 Volume 처리 정책
# Delete : Disk 삭제
# Retain : Disk 유지
reclaimPolicy: Delete

# Volume 생성 타이밍
# Immediate : PVC 생성 즉시 PV 생성
# WaitForFirstConsumer : Pod scheduling 후 생성
volumeBindingMode: Immediate

# Linux mount 옵션
# 예: mount -o noatime,nodiratime
mountOptions:
  - noatime
  - nodiratime

# PVC size 변경 허용 (Driver가 resize 수행)
allowVolumeExpansion: true

# 특정 topology에서만 볼륨 생성
# 예시 : zone 제한
allowedTopologies:
  - matchLabelExpressions:
      - key: topology.kubernetes.io/zone
        values:
          - ap-northeast-2a
```

## 13. Dynamic Provisioning 내부 흐름

```
PVC 생성
↓
API Server
↓
external-provisioner 감지
↓
CSI driver 호출
↓
Volume 생성
↓
PV 생성
↓
PVC binding
```

## 14. 실제 동작 흐름

```
PVC 생성
 │
 ▼
API Server
 │
 ▼
external-provisioner (watch PVC)
 │
 ▼
CreateVolume (CSI RPC)
 │
 ▼
Cloud / Storage
 │
 ▼
PV 생성
 │
 ▼
PVC binding
```

## 15. CSI 호출

external-provisioner는 CSI RPC 호출한다.

RPC

```
CreateVolume
DeleteVolume
ControllerPublishVolume
NodeStageVolume
NodePublishVolume
```

## 16. 실제 Linux에서 일어나는 일

Volume 생성 이후

Pod 실행

```
kubelet
↓
CSI Node Plugin
↓
device attach
↓
mount
↓
container runtime mount
```

실제 mount 위치

```
/var/lib/kubelet/plugins/
```

또는

```
/var/lib/kubelet/pods/<uid>/volumes
```

## 17. StorageClass 확인

클러스터에서

```
kubectl get storageclass
```

상세

```
kubectl describe storageclass
```

예

```
NAME                 PROVISIONER
standard             csi.aws.com
nfs-client           nfs.csi.k8s.io
```

## 18. 전체 구조 (정리)

```
Pod
 │
 ▼
PVC
 │
 ▼
StorageClass
 │
 ▼
External Provisioner
 │
 ▼
CSI Driver
 │
 ▼
Storage System
```

---

# 2장 nfs 용 StorageClass 만들기

## 준비물

```
[공통 - Server (NFS Server), Client (WorkerNode)]
dnf install nfs-utils
# nfs 서비스 활성화
systemctl start nfs-server
systemctl enable nfs-server
# 관련 데몬 rpcbind 활성화 (보통 nfs 서비스 활성화되면 자동으로 활성화되긴 함)
systemctl start rpcbind
systemctl enable rpcbind
```

```
[NFS Server]
# nfs 서버용 디렉토리 생성
➜  mkdir /nfsdir

# NFS 환경설정 파일 수정
# [공유할 디렉토리] [허가할 호스트](디렉토리 권한)
➜  vi /etc/exports
예시 : /nfsdir *(rw,sync)

---
* NFS 환경설정 파일 옵션 ( /etc/exports )
ro / rw	        : read only / read write
root_squash     :	클라이언트에서 접근하는 root 무시 -> 서버 상의 nobody로 매핑됨 (default)
no_root_squash	: 클라이언트에서 접근하는 root 허용
secure	        : 클라이언트 마운트 요청 시 포트 번호가 1024 이하인 경우만 허가
noaccess	    : 액세스 거부
sync	        : 파일 시스템이 변경되면 즉시 동기화
---

# 수정내용 반영
➜  exportfs -r

➜  firewall-cmd --permanent --add-service=nfs
➜  firewall-cmd --reload
➜  firewall-cmd --list-all

# NFS 서버 구성 확인
showmount -e [ip]	# NFS 클라이언트에서 NFS 서버의 공유된 정보 확인
exportfs -v			# NFS 서버에서 외부에 공유된 내용 자세히 출력
```

## 1️⃣ NFS Provisioner 설치

먼저 NFS 서버 정보가 필요하다.

예:

```
NFS Server: 192.168.0.50
Export Path: /data/nfs
```

Helm으로 설치하는 것이 가장 간단하다.

```
helm repo add nfs-subdir-external-provisioner \
https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/

helm repo update
```

설치

```
helm install nfs-client nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
 --set nfs.server=192.168.0.50 \
 --set nfs.path=/data/nfs
```

설치되면 이런 Pod가 생긴다.

```
nfs-client-nfs-subdir-external-provisioner
```

이 Pod가 PV를 자동 생성하는 역할을 한다.

## 2️⃣ NFS StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-storage
provisioner: cluster.local/nfs-subdir-external-provisioner
reclaimPolicy: Delete
volumeBindingMode: Immediate
parameters:
  archiveOnDelete: "false"
```

확인

```
kubectl get storageclass
```

## 3️⃣ PVC 생성

이제 PVC만 만들면 PV가 자동 생성된다.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: nfs-storage
  resources:
    requests:
      storage: 1Gi
```

확인

```
kubectl get pv
kubectl get pvc
```

## 4️⃣ NFS 사용하는 Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nfs-test
spec:
  containers:
    - name: app
      image: busybox
      command: ["sleep", "3600"]
      volumeMounts:
        - name: data
          mountPath: /data
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: nfs-pvc
```

Pod 접속

```
kubectl exec -it nfs-test -- sh
```

테스트

```
cd /data
touch test.txt
```

NFS 서버를 보면

```
/data/nfs/pvc-xxxx/
```

디렉토리가 자동으로 생성되어 있다.
