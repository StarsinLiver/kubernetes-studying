## 목차

- [1장 Namespace 란?](#1장-namespace-란)
  - [✅ 1. Namespace 한줄 정리](#-1-namespace-한줄-정리)
  - [✅ 2. Namespace의 역할 (왜 존재하냐)](#-2-namespace의-역할-왜-존재하냐)
  - [✅ 3. 어디서 실행되냐 (Control Plane 관점)](#-3-어디서-실행되냐-control-plane-관점)
    - [구성 요소별 역할](#구성-요소별-역할)
      - [API Server](#api-server)
      - [etcd](#etcd)
      - [Controller Manager](#controller-manager)
  - [✅ 4. 실제 Linux 어디에 저장되냐](#-4-실제-linux-어디에-저장되냐)
    - [🔍 실제 확인 방법](#-실제-확인-방법)
      - [1) kubectl로 확인](#1-kubectl로-확인)
      - [2) etcd 직접 조회 (마스터 노드에서)](#2-etcd-직접-조회-마스터-노드에서)
  - [✅ 5. 핵심 개념 (시험/면접 포인트)](#-5-핵심-개념-시험면접-포인트)
  - [✅ 6. 전체 구조 (아키텍처 관점)](#-6-전체-구조-아키텍처-관점)
  - [✅ 7. 중요한 흐름 (Lifecycle)](#-7-중요한-흐름-lifecycle)
  - [✅ 8. Namespace 생성 방법](#-8-namespace-생성-방법)
  - [✅ 9. Namespace 지정해서 리소스 생성](#-9-namespace-지정해서-리소스-생성)
  - [✅ 10. default Namespace 변경](#-10-default-namespace-변경)
  - [✅ 11. Namespace와 연결되는 핵심 기능들](#-11-namespace와-연결되는-핵심-기능들)
    - [🔥 1) RBAC](#-1-rbac)
    - [🔥 2) ResourceQuota](#-2-resourcequota)
    - [🔥 3) LimitRange](#-3-limitrange)
    - [🔥 4) NetworkPolicy](#-4-networkpolicy)
  - [✅ 12. 실무에서 쓰는 패턴](#-12-실무에서-쓰는-패턴)
- [2장 finalizer 란?](#2장-finalizer-란)
  - [✅ 1. Finalizer](#-1-finalizer)
  - [✅ 2. 실제 YAML 예시](#-2-실제-yaml-예시)
  - [✅ 3. 어디서 관리될까](#-3-어디서-관리될까)
  - [✅ 4. CLI 에서 확인](#-4-cli-에서-확인)
  - [✅ 5. 핵심 개념](#-5-핵심-개념)
  - [✅ 6. 강제 삭제 (위험)](#-6-강제-삭제-위험)

---

# 1장 Namespace 란?

## ✅ 1. Namespace 한줄 정리

```
👉 클러스터 안에서 리소스를 논리적으로 분리하는 “가상 공간”
```

## ✅ 2. Namespace의 역할 (왜 존재하냐)

Namespace는 단순한 폴더 개념이 아니라 실제 운영에서 매우 중요함

```
리소스 격리 (dev / staging / prod)
같은 이름 리소스 공존 가능 (pod 이름 중복 허용)
권한 관리 단위 (RBAC)
리소스 제한 (CPU, Memory quota)
네트워크 정책 적용 단위
```

👉 즉, 멀티테넌시(여러 팀/서비스 공존)를 위한 핵심 구조

## ✅ 3. 어디서 실행되냐 (Control Plane 관점)

Namespace는 특정 노드에서 실행되는 개념이 아님

```
→ Control Plane에서 관리되는 "메타데이터 객체"
```

### 구성 요소별 역할

#### API Server

```
Namespace 생성/조회/삭제 요청 처리
```

#### etcd

```
실제 Namespace 데이터 저장
```

#### Controller Manager

```
Namespace lifecycle 관리 (삭제 시 garbage collection)
```

## ✅ 4. 실제 Linux 어디에 저장되냐

👉 핵심: 파일로 저장 안됨

```
Namespace는 etcd에 key-value 형태로 저장됨
```

예시 (개념적으로)

```
/registry/namespaces/default
/registry/namespaces/dev
```

### 🔍 실제 확인 방법

#### 1) kubectl로 확인

```
kubectl get ns
kubectl describe ns default
```

#### 2) etcd 직접 조회 (마스터 노드에서)

```
ETCDCTL_API=3 etcdctl get /registry/namespaces --prefix --keys-only

또는 상세:
etcdctl get /registry/namespaces/default -w json
```

## ✅ 5. 핵심 개념 (시험/면접 포인트)

🔥 1) Namespace는 “논리적 격리”

```
물리적 격리 ❌
클러스터는 하나
```

🔥 2) 일부 리소스는 Namespace 없음

```
👉 Cluster-wide 리소스
```

예시

```
Node
PersistentVolume
Namespace
ClusterRole
```

🔥 3) 기본 Namespace들

```
default → 기본
kube-system → 시스템 컴포넌트
kube-public → 공개
kube-node-lease → 노드 상태
```

## ✅ 6. 전체 구조 (아키텍처 관점)

```
Kubernetes Cluster
 ├── Namespace: dev
 │    ├── Pod
 │    ├── Service
 │    └── Deployment
 │
 ├── Namespace: prod
 │    ├── Pod
 │    ├── Service
 │    └── Deployment
 │
 └── Cluster-level
      ├── Node
      ├── PV
      └── ClusterRole
```

## ✅ 7. 중요한 흐름 (Lifecycle)

생성 흐름

```
1. kubectl apply
2. API Server 요청 수신
3. etcd에 저장
4. Namespace controller 관리 시작
```

삭제 흐름 (중요🔥)

```
1. Namespace 삭제 요청
2. 상태: Terminating
3. 내부 리소스 전부 삭제
4. finalizer 제거
5. 완전히 삭제

👉 그래서 namespace 삭제가 오래 걸리는 이유 = 내부 리소스 때문
```

## ✅ 8. Namespace 생성 방법

```
kubectl create namespace dev
```

또는 YAML

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev

  # labels (옵션): 필터링, 선택, 정책 적용에 사용
  labels:
    env: development
    team: backend

  # annotations (옵션)
  # 추가 설명 (비식별 메타데이터)
  # 시스템이 아닌 사람이 읽는 용도
  annotations:
    description: "개발 환경 네임스페이스"
```

## ✅ 9. Namespace 지정해서 리소스 생성

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  namespace: dev # namespace 이름
```

또는 CLI

```
kubectl run nginx --image=nginx -n dev
```

## ✅ 10. default Namespace 변경

```
kubectl config set-context --current --namespace=dev
```

## ✅ 11. Namespace와 연결되는 핵심 기능들

Namespace는 단독으로 끝나는게 아니라 아래랑 같이 써야 진짜 의미 있음

### 🔥 1) RBAC

```
Namespace 단위 권한
```

### 🔥 2) ResourceQuota

```
kind: ResourceQuota
CPU, Memory 제한
```

### 🔥 3) LimitRange

```
Pod 단위 제한
```

### 🔥 4) NetworkPolicy

```
Namespace 간 통신 제어
```

## ✅ 12. 실무에서 쓰는 패턴

📌 환경 분리

```
dev
staging
prod
```

📌 팀 분리

```
team-a
team-b
```

📌 서비스 단위 분리 (대규모)

```
auth
payment
api
```

---

# 2장 finalizer 란?

## ✅ 1. Finalizer

✔ 한줄 정리

```
👉 리소스 삭제 전에 “반드시 해야 할 작업”을 보장하는 잠금 장치
```

✔ 어떤 역할?

```
리소스를 삭제할 때 바로 삭제되지 않고 중간에 멈추게 하는 장치
```

예시

```
1. PVC 삭제 → 실제 스토리지 정리 필요
2. Namespace 삭제 → 내부 리소스 전부 삭제 필요
```

✔ 동작 흐름 (중요🔥)

```
1. kubectl delete
2. 리소스 상태 → Terminating
3. finalizer 존재 확인
4. finalizer 작업 수행
5. finalizer 제거
6. 완전 삭제
```

## ✅ 2. 실제 YAML 예시

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
  finalizers:
    - kubernetes
```

## ✅ 3. 어디서 관리될까

```
API Server + Controller
etcd에 저장됨
```

## ✅ 4. CLI 에서 확인

```
kubectl get ns dev -o yaml
```

👉 여기서 확인이 가능하다.

```
finalizers:
  - kubernetes
```

## ✅ 5. 핵심 개념

```
finalizer가 있으면 삭제 안됨
컨트롤러가 직접 제거해야 삭제됨
문제 생기면 stuck됨 (🔥 실무 자주 발생)
```

## ✅ 6. 강제 삭제 (위험)

```
kubectl patch ns dev -p '{"metadata":{"finalizers":[]}}' --type=merge
```
