## 목차

- [1장 kube-controller-manager 란?](#1장-kube-controller-manager-란)
  - [🧠 역할](#-역할)
  - [⚙️ 전체 흐름 (핵심🔥)](#️-전체-흐름-핵심)
  - [핵심 개념: Reconciliation Loop](#핵심-개념-reconciliation-loop)
  - [🧩 주요 컨트롤러들](#-주요-컨트롤러들)
  - [🔄 동작 방식](#-동작-방식)
  - [💥 핵심 특징](#-핵심-특징)

---

# 1장 kube-controller-manager 란?

🔹 한 줄 정의

클러스터의 현재 상태(status)를 원하는 상태(spec)로 계속 맞추는 컨트롤러들의 집합
<br> 참조 : 쿠버네티스는 “원하는 상태(spec)”를 유지하려고 계속 자동으로 수정한다
<br> 그걸 실제로 수행하는 게 controller-manager

### 🧠 역할

```
✔️ 하는 일
상태 비교
부족하면 생성
많으면 삭제

❌ 안 하는 일
노드 선택 ❌ (scheduler)
컨테이너 실행 ❌ (kubelet)
```

### ⚙️ 전체 흐름 (핵심🔥)

```
사용자 spec 설정
   ↓
etcd 저장
   ↓
controller가 watch
   ↓
현재 상태 확인
   ↓
차이 발생
   ↓
보정 (reconcile)
```

### 핵심 개념: Reconciliation Loop

```
❗ 계속 반복하면서 상태를 맞춘다

🔹 예시 (ReplicaSet)
replicas: 3
실제 상황
현재 Pod = 2
원하는 Pod = 3

👉 controller 동작:
Pod 1개 추가 생성
```

### 🧩 주요 컨트롤러들

```
kube-controller-manager 안에는 여러 controller가 있다

1️⃣ ReplicaSet Controller
👉 Pod 개수 유지

2️⃣ Deployment Controller
👉 업데이트 / 롤링 업데이트

3️⃣ Node Controller
👉 노드 상태 체크

4️⃣ Job Controller
👉 작업 완료 여부 관리

5️⃣ Endpoints Controller
👉 Service → Pod 연결 관리
```

### 🔄 동작 방식

```
1️⃣ watch
etcd 변화 감지

2️⃣ 비교
spec vs status

3️⃣ 수정
필요한 작업 실행

📦 예시 1: Pod 삭제
replicas = 3
현재 = 3
---
Pod 하나 삭제됨
↓
현재 = 2
↓
controller 감지
↓
Pod 하나 생성
---

📦 예시 2: Deployment 업데이트
image: v1 → v2

👉 controller:
v1 Pod 삭제
v2 Pod 생성

📡 controller는 어떻게 감지하냐?
👉 kube-apiserver watch
etcd 변경 → apiserver → controller watch
```

### 💥 핵심 특징

```
✔️ 계속 반복
이벤트 기반 + loop

✔️ 자동 복구
self-healing

✔️ 선언형 시스템
“어떻게”가 아니라 “원하는 상태”

🧪 실제 확인
kubectl -n kube-system logs {controller pod}
```
