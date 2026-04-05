## 목차

- [1장 PersistentVolume 이란](#1장-persistentvolume-이란)
  - [1️⃣ PersistentVolume (PV)](#1️⃣-persistentvolume-pv)
  - [2️⃣ 왜 PV가 필요한가](#2️⃣-왜-pv가-필요한가)
  - [3️⃣ PV 역할](#3️⃣-pv-역할)
  - [4️⃣ Kubernetes 구조에서 위치](#4️⃣-kubernetes-구조에서-위치)
  - [5️⃣ 어디서 실행되거나 저장되는가](#5️⃣-어디서-실행되거나-저장되는가)
  - [6️⃣ 실제 Linux에서 어디 존재하는가](#6️⃣-실제-linux에서-어디-존재하는가)
    - [1️⃣ Kubernetes Object](#1️⃣-kubernetes-object)
    - [2️⃣ 실제 storage](#2️⃣-실제-storage)
      - [hostPath PV (Node filesystem)](#hostpath-pv-node-filesystem)
      - [NFS PV (remote server)](#nfs-pv-remote-server)
      - [CSI PV (external storage)](#csi-pv-external-storage)
  - [7️⃣ PV 생성 과정](#7️⃣-pv-생성-과정)
  - [8️⃣ PV controller 역할](#8️⃣-pv-controller-역할)
  - [9️⃣ PV lifecycle](#9️⃣-pv-lifecycle)
  - [🔟 PV 상태 설명](#-pv-상태-설명)
    - [Available](#available)
    - [Bound](#bound)
    - [Released](#released)
    - [Failed](#failed)
  - [11. 실제 mount는 누가 하나](#11-실제-mount는-누가-하나)
  - [12. Linux mount 과정](#12-linux-mount-과정)
  - [13. Node에서 실제 mount 위치](#13-node에서-실제-mount-위치)
  - [14. PV YAML 모든 옵션](#14-pv-yaml-모든-옵션)
  - [15. Volume Source](#15-volume-source)
    - [예제 : hostPath](#예제--hostpath)
    - [예제 : NFS](#예제--nfs)
      - [NFS (Network File System) 설치 시](#nfs-network-file-system-설치-시)
    - [예제 : CSI](#예제--csi)
  - [16. Linux에서 확인](#16-linux에서-확인)
  - [17. PV 전체 구조](#17-pv-전체-구조)

---

# 1장 PersistentVolume 이란

## 1️⃣ PersistentVolume (PV)

한 줄 정리

```
클러스터 전체에서 사용할 수 있는 영구 저장소 리소스

또는

Kubernetes 클러스터에서 사용할 수 있는 실제 스토리지를 나타내는 리소스
```

## 2️⃣ 왜 PV가 필요한가

지금까지 본 Volume

| 타입      | 특징                     |
| --------- | ------------------------ |
| emptyDir  | Pod 삭제하면 데이터 삭제 |
| hostPath  | Node에 종속              |
| ConfigMap | 설정 데이터 (아직 안봄)  |
| Secret    | 보안 데이터 (아직 안봄)  |

문제

```
Pod가 죽어도 데이터 유지해야 한다
```

예

```
database
uploads
logs
user data
```

```
그래서 만든 것이 PersistentVolume
```

## 3️⃣ PV 역할

```
PV의 목적은 스토리지와 Pod를 분리 이다.
```

즉

```
Application
Pod
Storage

이 세 개를 독립적으로 관리할 수 있다.
```

## 4️⃣ Kubernetes 구조에서 위치

```
PV는 cluster level resource다.
```

구조

```
Kubernetes Cluster

API Server
   │
   ▼
PersistentVolume (PV)
   │
   ▼
Storage Backend
```

즉

```
PV는 Node에 속하지 않는다.
```

## 5️⃣ 어디서 실행되거나 저장되는가

PV 자체는 실행되는 프로세스가 아니다.

```
PV는 API Object이다.

즉 저장 위치는 etcd이다.
```

## 6️⃣ 실제 Linux에서 어디 존재하는가

PV는 두 부분으로 나뉜다.

### 1️⃣ Kubernetes Object

API Server → etcd

```
kubectl get pv
```

### 2️⃣ 실제 storage

스토리지 종류에 따라 다르다.

#### hostPath PV (Node filesystem)

```
/data/pv-storage
```

#### NFS PV (remote server)

```
192.168.1.100:/exports
```

구조

```
기본 구조는 매우 단순하다.

        NFS Server
      192.168.0.10
      /data

        ▲   ▲   ▲
        │   │   │

node1       node2       node3
mount       mount       mount
/data       /data       /data
```

#### CSI PV (external storage)

예

```
AWS EBS
Ceph
```

## 7️⃣ PV 생성 과정

PV 생성 흐름

```
kubectl apply
     │
     ▼
API Server
     │
     ▼
etcd 저장
     │
     ▼
PV controller 감시
```

```
PV controller는 kube-controller-manager안에 있다.
```

## 8️⃣ PV controller 역할

controller 위치

```
kube-controller-manager
```

PV 관련 controller

```
persistentvolume-controller
```

코드 위치

```
pkg/controller/volume/persistentvolume
```

controller 역할

```
PV lifecycle 관리
binding 관리
reclaim policy 처리
```

## 9️⃣ PV lifecycle

```
Create
   │
   ▼
Available
   │
   ▼
Bound
   │
   ▼
Released
   │
   ▼
Failed
```

## 🔟 PV 상태 설명

### Available

```
사용 가능
```

### Bound

```
PVC 연결됨
```

### Released

```
PVC 삭제됨
```

### Failed

```
스토리지 문제
```

## 11. 실제 mount는 누가 하나

```
PV는 자체적으로 mount 하지 않는다.
mount는 kubelet이 한다.
```

kubelet 흐름

```
Pod 생성
   │
   ▼
PVC 확인
   │
   ▼
PV 확인
   │
   ▼
Volume plugin
   │
   ▼
Linux mount
```

## 12. Linux mount 과정

NFS PV

```
mount -t nfs 192.168.1.100:/data /var/lib/kubelet/pods/.../
```

hostPath PV

```
mount --bind /data /var/lib/kubelet/pods/.../
```

CSI PV

```
CSI driver mount 수행
```

## 13. Node에서 실제 mount 위치

```
/var/lib/kubelet/pods/<pod-uid>/volumes/
```

예

```
/var/lib/kubelet/pods/
  93af21
     volumes
        kubernetes.io~csi
        kubernetes.io~nfs
```

확인

```
mount | grep kubelet
```

## 14. PV YAML 모든 옵션

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
  namespace: default
  labels: {}
  annotations: {}
  finalizers: {}

spec:
  # PV가 제공할 스토리지 용량을 정의한다.
  # PVC가 PV를 요청할 때 이 값을 기준으로 매칭된다.
  capacity:
    storage: 10Gi

  # Filesystem : 즉, ext4 xfs 같은 파일 시스템으로 마운트
  # Block (Raw block device) : 컨테이너에 /dev/sdX 형태로 제공 (예: DB에서 사용)
  volumeMode: Filesystem

  # 볼륨을 몇 개의 노드 / Pod가 사용할 수 있는지
  # ReadWriteOnce : 한 노드에서만 읽기/쓰기 가능 다만, 여러 Pod 가능 (같은 node)
  # ReadOnlyMany : 여러 노드에서 읽기만 가능
  # ReadWriteMany : 여러 노드 읽기쓰기
  # ReadWriteOncePod : 오직 하나의 Pod만 사용
  accessModes: ReadWriteOnce

  # PVC가 삭제됐을 때 PV 데이터를 어떻게 처리할지
  # 즉, PVC 삭제 후 PV 처리 정책
  # Retain : 데이터 유지 (관리자가 수동 삭제)
  # Delete : 스토리지도 같이 삭제
  # Recycle (deprecated) : 자동 정리 (rm -rf)
  persistentVolumeReclaimPolicy: Retain

  # 이 PV가 속한 StorageClass 지정
  storageClassName: manual

  # Linux mount 옵션 (mount 할 때 옵션 전달)
  # 예시 : mount -t nfs -o hard,nfsvers=4.1
  mountOptions:
    - hard
    - nfsvers=4.1

  # PVC 연결 정보 (PV가 어떤 PVC와 연결됐는지)
  claimRef:
    namespace: default
    name: myclaim

  # 특정 Node에서만 사용 (이 PV를 특정 노드에서만 사용하도록 제한)
  nodeAffinity:
    required:
      nodeSelectorTerms: <node>
```

## 15. Volume Source

PV는 여러 storage 타입을 지원한다.

대표

```
hostPath
nfs
iscsi
cephfs
rbd
csi
local
awsElasticBlockStore
gcePersistentDisk
```

### 예제 : hostPath

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-hostpath

spec:
  capacity:
    storage: 10Gi

  volumeMode: Filesystem

  accessModes:
    - ReadWriteOnce

  persistentVolumeReclaimPolicy: Retain

  storageClassName: manual

  hostPath:
    path: /data/pv
```

### 예제 : NFS

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs

spec:
  capacity:
    storage: 20Gi

  accessModes:
    - ReadWriteMany

  persistentVolumeReclaimPolicy: Retain

  storageClassName: nfs

  nfs:
    path: /exports
    server: 192.168.1.100
```

#### NFS (Network File System) 설치 시

```
[공통 - Server, Client 다운]
dnf install nfs-utils
# nfs 서비스 활성화
systemctl start nfs-server
systemctl enable nfs-server
# 관련 데몬 rpcbind 활성화 (보통 nfs 서비스 활성화되면 자동으로 활성화되긴 함)
systemctl start rpcbind
systemctl enable rpcbind
```

```
[Server]
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

### 예제 : CSI

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-csi

spec:
  capacity:
    storage: 100Gi

  accessModes:
    - ReadWriteOnce

  storageClassName: csi-storage

  csi:
    driver: ebs.csi.aws.com
    volumeHandle: vol-123456
```

## 16. Linux에서 확인

PV 확인

```
kubectl get pv
```

상세 정보

```
kubectl describe pv pv-hostpath
```

Node mount 확인

```
mount | grep kubelet
```

kubelet volume

```
ls /var/lib/kubelet/pods
```

## 17. PV 전체 구조

```
PersistentVolume (API Object)
        │
        ▼
Storage Backend
        │
        ▼
Volume Plugin
        │
        ▼
Linux mount
        │
        ▼
Container filesystem
```
