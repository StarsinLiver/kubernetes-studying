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
