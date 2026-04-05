## 목차

- [1장 CSI Driver 란?](#1장-csi-driver-란)
  - [1️⃣ CSI Driver란](#1️⃣-csi-driver란)
  - [2️⃣ 한줄 정리](#2️⃣-한줄-정리)
  - [3️⃣ 왜 CSI가 만들어졌나](#3️⃣-왜-csi가-만들어졌나)
  - [4️⃣ CSI 전체 구조](#4️⃣-csi-전체-구조)
  - [5️⃣ CSI Driver 구성 요소](#5️⃣-csi-driver-구성-요소)
  - [6️⃣ Controller Plugin](#6️⃣-controller-plugin)
  - [7️⃣ Node Plugin](#7️⃣-node-plugin)
  - [8️⃣ CSI Sidecar](#8️⃣-csi-sidecar)
  - [9️⃣ CSI RPC API](#9️⃣-csi-rpc-api)
    - [주요 RPC 설명](#주요-rpc-설명)
      - [CreateVolume](#createvolume)
      - [ControllerPublishVolume](#controllerpublishvolume)
      - [NodeStageVolume](#nodestagevolume)
      - [NodePublishVolume](#nodepublishvolume)
  - [🔟 실제 Kubernetes 동작 흐름](#-실제-kubernetes-동작-흐름)
  - [11. 실제 Linux에서 일어나는 일](#11-실제-linux에서-일어나는-일)
  - [12. 실제 Linux mount 구조](#12-실제-linux-mount-구조)
  - [13. CSI Driver 확인](#13-csi-driver-확인)
  - [14. CSI 관련 리소스](#14-csi-관련-리소스)
  - [15. 실제 디렉토리](#15-실제-디렉토리)
- [2장 CSI Driver 설치](#2장-csi-driver-설치)
  - [1. 설치 시 생성되는 Kubernetes 리소스](#1-설치-시-생성되는-kubernetes-리소스)
  - [2. Controller Pod 구조](#2-controller-pod-구조)
  - [3. Node Plugin 구조](#3-node-plugin-구조)
  - [4. CSI Driver 리소스](#4-csi-driver-리소스)
  - [5. CSIDriver YAML](#5-csidriver-yaml)
  - [6. Linux 노드에서 생성되는 파일](#6-linux-노드에서-생성되는-파일)
  - [7. kubelet 등록 과정](#7-kubelet-등록-과정)
  - [10. 실제 설치 예제 (NFS CSI Driver)](#10-실제-설치-예제-nfs-csi-driver)
  - [11. StorageClass 생성](#11-storageclass-생성)
  - [12. PVC 생성](#12-pvc-생성)
  - [13. 실제 동작 흐름](#13-실제-동작-흐름)
  - [14. 실제 Linux mount 위치](#14-실제-linux-mount-위치)
  - [15. 전체 흐름](#15-전체-흐름)

---

# 1장 CSI Driver 란?

## 1️⃣ CSI Driver란

```
CSI는 쿠버네티스가 외부 스토리지 시스템과 통신하는 표준 인터페이스다.

즉, Kubernetes ↔ Storage 시스템을 연결하는 드라이버이다.
```

예

```
AWS EBS
Ceph
NFS
NetApp
Longhorn
```

## 2️⃣ 한줄 정리

```
CSI Driver = Kubernetes가 외부 스토리지를 사용할 수 있게 하는 드라이버
```

## 3️⃣ 왜 CSI가 만들어졌나

```
옛날 Kubernetes는 스토리지 플러그인이 kubelet 내부 코드에 있었다.
```

예

```
kubernetes.io/aws-ebs
kubernetes.io/gce-pd
kubernetes.io/nfs
```

문제

```
1. Kubernetes 코드 수정 필요
2. Kubernetes release 의존
3. Vendor 개발 어려움
```

그래서 등장

```
CSI (Container Storage Interface)
```

## 4️⃣ CSI 전체 구조

```
Kubernetes
   │
   ▼
CSI Sidecar
   │
   ▼
CSI Driver
   │
   ▼
Storage System
```

더 정확한 구조

```
          Kubernetes
                │
                ▼
  +---------------------------+
  |      CSI Sidecars         |
  |---------------------------|
  | external-provisioner      |
  | external-attacher         |
  | external-resizer          |
  | external-snapshotter      |
  +---------------------------+
                │
                ▼
        CSI Driver
  ┌───────────────┬───────────────┐
  │ Controller     │ Node Plugin   │
  └───────────────┴───────────────┘
                │
                ▼
          Storage System
```

## 5️⃣ CSI Driver 구성 요소

CSI driver는 3가지로 구성된다.

```
1️⃣ Controller Plugin
2️⃣ Node Plugin
3️⃣ Sidecar Containers
```

## 6️⃣ Controller Plugin

역할

```
스토리지 생성 / 삭제 / attach
```

```
보통 Deployment로 실행된다.
```

예

```
CreateVolume
DeleteVolume
ControllerPublishVolume
```

```
➜  kubectl get pods -n kube-system

ebs-csi-controller
ceph-csi-controller
```

## 7️⃣ Node Plugin

역할

```
Node에서 실제 mount 수행
```

```
Node plugin은 DaemonSet으로 실행된다.
```

예

```
NodeStageVolume
NodePublishVolume
```

```
➜  kubectl get pods -n kube-system

ebs-csi-node
ceph-csi-node
```

## 8️⃣ CSI Sidecar

Sidecar는 Kubernetes API와 CSI Driver를 연결한다.

| Sidecar              | 역할        |
| -------------------- | ----------- |
| external-provisioner | volume 생성 |
| external-attacher    | attach      |
| external-resizer     | resize      |
| external-snapshotter | snapshot    |

```
➜  kubectl get pods -n kube-system

보면 같이 붙어있다.

csi-provisioner
csi-attacher
csi-resizer
```

## 9️⃣ CSI RPC API

CSI는 gRPC API로 동작한다.

대표 RPC

```
CreateVolume
DeleteVolume
ControllerPublishVolume
ControllerUnpublishVolume
NodeStageVolume
NodePublishVolume
NodeUnpublishVolume
```

### 주요 RPC 설명

#### CreateVolume

```
새 volume 생성

[예]
AWS EBS 생성
```

#### ControllerPublishVolume

```
volume attach

[예]
EBS attach → EC2 instance
```

#### NodeStageVolume

```
Node에 device 준비
```

#### NodePublishVolume

```
container에 mount
```

## 🔟 실제 Kubernetes 동작 흐름

```
PVC 생성
↓
StorageClass 확인
↓
external-provisioner
↓
CSI CreateVolume
↓
PV 생성
↓
Pod 생성
↓
Scheduler node 선택
↓
CSI attach
↓
kubelet mount
↓
container mount
```

실제 흐름 그림

```
PVC
 │
 ▼
external-provisioner
 │
 ▼
CSI CreateVolume
 │
 ▼
PV 생성
 │
 ▼
Pod 생성
 │
 ▼
Scheduler
 │
 ▼
external-attacher
 │
 ▼
CSI ControllerPublishVolume
 │
 ▼
Node attach
 │
 ▼
kubelet
 │
 ▼
CSI NodePublishVolume
 │
 ▼
mount
 │
 ▼
container runtime mount
```

## 11. 실제 Linux에서 일어나는 일

CSI Node plugin이 mount 수행

예

```
mount /dev/xvdf /var/lib/kubelet/plugins/kubernetes.io/csi/...
```

Pod mount

```
/var/lib/kubelet/pods/<uid>/volumes/
```

container mount, container runtime

```
containerd
```

bind mount

## 12. 실제 Linux mount 구조

노드에서

```
mount | grep kubelet
```

예

```
/dev/sdb on /var/lib/kubelet/plugins/kubernetes.io/csi/pv/... type ext4
```

Pod mount

```
/var/lib/kubelet/pods/<uid>/volumes/
```

container mount

```
overlayfs
 + volume bind mount
```

## 13. CSI Driver 확인

클러스터에서

```
kubectl get csidriver
```

예

```
NAME
ebs.csi.aws.com
cephfs.csi.ceph.com
```

노드 plugin

```
kubectl get pods -n kube-system
```

## 14. CSI 관련 리소스

Kubernetes API

```
CSIDriver
CSINode
VolumeAttachment
```

예

```
kubectl get volumeattachment
```

## 15. 실제 디렉토리

노드

```
/var/lib/kubelet/plugins/
```

CSI plugin socket

```
(예시)
/var/lib/kubelet/plugins/ebs.csi.aws.com/
```

gRPC socket

```
csi.sock
```

---

# 2장 CSI Driver 설치

## 1. 설치 시 생성되는 Kubernetes 리소스

CSI Driver 설치하면 보통 다음 리소스들이 생성된다.

```
Deployment
DaemonSet
ServiceAccount
ClusterRole
ClusterRoleBinding
CSIDriver
StorageClass
```

구조

```
kube-system namespace

 ├ Deployment
 │   └ csi-controller
 │
 ├ DaemonSet
 │   └ csi-node
 │
 ├ CSIDriver
 │
 └ StorageClass
```

## 2. Controller Pod 구조

Controller는 보통 Deployment이다.

예

```
➜  kubectl get pods -n kube-system
ebs-csi-controller-xxx
```

Pod 내부 containers

```
1. csi-driver
2. csi-provisioner
3. csi-attacher
4. csi-resizer
5. csi-snapshotter
```

| Container       | 역할             |
| --------------- | ---------------- |
| csi-driver      | 실제 storage API |
| csi-provisioner | volume 생성      |
| csi-attacher    | volume attach    |
| csi-resizer     | volume resize    |
| snapshotter     | snapshot         |

## 3. Node Plugin 구조

```
Node plugin은 DaemonSet으로 설치된다.

왜냐하면 모든 node에서 mount 해야하기 때문
```

예

```
➜  kubectl get pods -n kube-system
ebs-csi-node-xxxxx
```

Pod 내부 containers

```
1. csi-driver
2. node-driver-registrar
3. liveness-probe
```

| Container  | 역할         |
| ---------- | ------------ |
| csi-driver | mount 수행   |
| registrar  | kubelet 등록 |
| liveness   | health check |

## 4. CSI Driver 리소스

설치하면 Kubernetes API에 CSIDriver 객체가 생성된다.

예

```
kubectl get csidriver
```

예

```
NAME
ebs.csi.aws.com
nfs.csi.k8s.io
```

## 5. CSIDriver YAML

```yaml
apiVersion: storage.k8s.io/v1
kind: CSIDriver
metadata:
  name: nfs.csi.k8s.io

spec:
  # true  → attach 필요
  # false → mount만
  attachRequired: false

  # Pod 정보를 CSI driver에 전달
  podInfoOnMount: true
```

## 6. Linux 노드에서 생성되는 파일

CSI Driver 설치 후 노드에 디렉토리 생성

```
/var/lib/kubelet/plugins/
```

예

```
/var/lib/kubelet/plugins/nfs.csi.k8s.io
```

CSI socket

```
csi.sock

이 socket으로 kubelet ↔ CSI driver 통신한다.
```

## 7. kubelet 등록 과정

```
Node plugin 시작
↓
node-driver-registrar
↓
kubelet plugin registry
↓
CSI driver 등록
```

registry 위치

```
/var/lib/kubelet/plugins_registry
```

socket 예

```
nfs.csi.k8s.io-reg.sock
```

## 10. 실제 설치 예제 (NFS CSI Driver)

공식 NFS CSI driver 설치

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/deploy/kubernete
```

## 11. StorageClass 생성

CSI Driver는 보통 StorageClass와 같이 사용한다.

예

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-csi

provisioner: nfs.csi.k8s.io

parameters:
  server: 10.0.0.10
  share: /exports
```

## 12. PVC 생성

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc

spec:
  accessModes:
    - ReadWriteMany

  storageClassName: nfs-csi

  resources:
    requests:
      storage: 5Gi
```

## 13. 실제 동작 흐름

```
PVC 생성
↓
external-provisioner 감지
↓
CSI CreateVolume RPC
↓
PV 생성
↓
Pod 생성
↓
kubelet
↓
NodePublishVolume
↓
mount
```

## 14. 실제 Linux mount 위치

노드에서

```
/var/lib/kubelet/pods/
```

예

```
/var/lib/kubelet/pods/<uid>/volumes/kubernetes.io~csi/
```

mount 확인

```
mount | grep kubelet
```

## 15. 전체 흐름

```
CSI Driver 설치
 │
 ▼
Controller Deployment
 │
 ▼
Node DaemonSet
 │
 ▼
CSI socket 등록
 │
 ▼
StorageClass 생성
 │
 ▼
PVC 생성
 │
 ▼
external-provisioner
 │
 ▼
CSI CreateVolume
 │
 ▼
PV 생성
 │
 ▼
Pod 생성
 │
 ▼
kubelet mount
```
