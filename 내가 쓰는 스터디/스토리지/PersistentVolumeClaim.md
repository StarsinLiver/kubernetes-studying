## 목차

- [1장 PersistentVolumeClaim 이란](#1장-persistentvolumeclaim-이란)
  - [1️⃣ PVC (PersistentVolumeClaim) 이란](#1️⃣-pvc-persistentvolumeclaim-이란)
  - [2️⃣ 한줄 정리](#2️⃣-한줄-정리)
  - [3️⃣ 전체 구조](#3️⃣-전체-구조)
  - [4️⃣ PVC가 존재하는 이유](#4️⃣-pvc가-존재하는-이유)
  - [5️⃣ PVC YAML (전체 구조)](#5️⃣-pvc-yaml-전체-구조)
  - [7️⃣ PVC 상태](#7️⃣-pvc-상태)
  - [8️⃣ PV-PVC Binding 내부 알고리즘](#8️⃣-pv-pvc-binding-내부-알고리즘)
  - [9️⃣ Kubernetes 내부 구조](#9️⃣-kubernetes-내부-구조)
  - [🔟 Pod가 PVC를 사용할 때](#-pod가-pvc를-사용할-때)
  - [11. 실제 Linux mount 위치](#11-실제-linux-mount-위치)
  - [12. 실제 Linux mount 확인](#12-실제-linux-mount-확인)

---

# 1장 PersistentVolumeClaim 이란

## 1️⃣ PVC (PersistentVolumeClaim) 이란

```
PVC는 Pod가 사용할 스토리지를 요청하는 객체다.

즉 Pod → PVC → PV → 실제 Storage구조이다.
```

## 2️⃣ 한줄 정리

```
Pod가 사용할 스토리지를 요청하는 Kubernetes 객체
```

## 3️⃣ 전체 구조

PVC는 스토리지 요청 API이다.

```
Application
   │
   ▼
Pod
   │
   ▼
PVC (요청)
   │
   ▼
PV (실제 volume)
   │
   ▼
Storage (disk / nfs / cloud disk)
```

## 4️⃣ PVC가 존재하는 이유

만약 PVC가 없다면 Pod가 직접

```
nfs:
hostPath:
awsElasticBlockStore:
```

같은 것을 써야 한다.

문제

```
1. Pod가 storage 종류를 알아야 함
2. 환경이 바뀌면 Pod 수정해야 함
3. Cloud마다 다름
```

그래서 Kubernetes는 Pod → PVC만 쓰게 한다.

## 5️⃣ PVC YAML (전체 구조)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: myclaim
  namespace: default
  labels: {}
  annotations: {}
  finalizers: {}

spec:
  # Pod가 volume을 어떤 방식으로 사용할지
  # PV와 matching 조건 (PVC accessModes ⊆ PV accessModes)
  # ReadWriteOnce : 한 노드에서만 읽기/쓰기 가능 다만, 여러 Pod 가능 (같은 node)
  # ReadOnlyMany : 여러 노드에서 읽기만 가능
  # ReadWriteMany : 여러 노드 읽기쓰기
  # ReadWriteOncePod : 오직 하나의 Pod만 사용
  accessModes:
    - ReadWriteOnce

  # Filesystem : 즉, ext4 xfs 같은 파일 시스템으로 마운트
  # Block (Raw block device) : 컨테이너에 /dev/sdX 형태로 제공 (예: DB에서 사용)
  volumeMode: Filesystem

  # PVC가 요구하는 스토리지 크기
  # PV binding 조건 (PV.capacity >= PVC.request)
  resources:
    requests:
      storage: 10Gi

  # 어떤 StorageClass를 사용할지
  storageClassName: standard

  # PVC가 특정 PV만 사용 (일반적으로 안쓴다.)
  volumeName: my-pv

  # 특정 label을 가진 PV만 선택
  selector:
    matchLabels:
      type: ssd

  # PVC를 Volume snapshot / clone 에서 생성
  # 예: snapshot → new pvc
  dataSource: {}

  # 새 API
  # CRD source 지원
  dataSourceRef: {}
```

## 7️⃣ PVC 상태

PVC lifecycle

```
Pending
Bound
Lost
```

Pending

```
PV 없음
```

Bound

```
PV 연결
```

Lost

```
PV 사라짐
```

## 8️⃣ PV-PVC Binding 내부 알고리즘

```
이건 kube-controller-manager 안에 있다.

컴포넌트
persistentvolume-controller
```

Binding 알고리즘

```
1 PVC 생성
2 PV 목록 조회
3 조건 검사
```

조건

```
capacity >= request
accessModes match
storageClass match
selector match
volumeMode match
```

조건 맞으면

```
PV.claimRef = PVC
PVC.volumeName = PV

상태
Bound
```

## 9️⃣ Kubernetes 내부 구조

Controller 위치

```
kube-controller-manager
```

내부

```
pkg/controller/volume/persistentvolume
```

주요 코드

```
pv_controller.go
binder.go
recycler.go
```

## 🔟 Pod가 PVC를 사용할 때

```yaml
volumes:
  - name: data
    persistentVolumeClaim:
      claimName: myclaim
```

전체 흐름

```
Pod 생성
   │
scheduler node 선택
   │
kubelet 실행
   │
kubelet
   │
VolumeManager
   │
CSI attach
   │
mount
   │
container runtime mount
```

## 11. 실제 Linux mount 위치

노드에서

```
/var/lib/kubelet/pods

[예]
/var/lib/kubelet/pods/<uid>/volumes/kubernetes.io~csi/
```

컨테이너 mount

```
/var/lib/containerd/io.containerd.runtime.v2.task/k8s.io
```

## 12. 실제 Linux mount 확인

노드에서

```
mount | grep kubelet
또는
findmnt
또는
ls /var/lib/kubelet/pods
```
