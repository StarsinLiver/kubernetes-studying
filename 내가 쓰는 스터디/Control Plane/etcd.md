## 목차

- [1장 etcd 란?](#1장-etcd-란)
  - [🧠 역할](#-역할)
  - [⚙️ 구조 (중요🔥)](#️-구조-중요)
  - [📦 데이터 구조](#-데이터-구조)
  - [🔄 동작 방식 (진짜 핵심🔥)](#-동작-방식-진짜-핵심)
  - [🔐 특징](#-특징)
  - [🔍 Watch 기능 (중요🔥)](#-watch-기능-중요)
  - [🧭 revision 이란?](#-revision-이란)

---

# 1장 etcd 란?

🔹 한 줄 정의

쿠버네티스 클러스터의 모든 상태를 저장하는 분산 Key-Value DB

### 🧠 역할

```
✔️ 1. 상태 저장소
Pod
Service
ConfigMap
Node
👉 전부 etcd에 저장됨

✔️ 2. 단일 진실의 원천 (Single Source of Truth)
👉 클러스터 상태는 오직 etcd 기준

✔️ 3. 변경 이력 관리
revision 기반
watch 가능
```

### ⚙️ 구조 (중요🔥)

```
kubectl
   ↓
kube-apiserver
   ↓
etcd (저장)

👉 ❗ 직접 접근 ❌
👉 반드시 API Server 통해야 함
```

### 📦 데이터 구조

```
etcd는 Key-Value 저장소다

🔹 실제 구조 느낌
/registry/pods/default/nginx
/registry/services/default/my-service
/registry/nodes/node1

👉 값(value)은 JSON 형태
🔹 예시
{
  "metadata": {
    "name": "nginx"
  },
  "spec": {...},
  "status": {...}
}
```

### 🔄 동작 방식 (진짜 핵심🔥)

```
✔️ Pod 생성 흐름
1. kubectl apply
2. API Server 요청
3. etcd에 저장
4. scheduler가 감지 (watch)
5. kubelet이 감지 (watch)
6. 실행

👉 핵심:
❗ etcd는 단순 DB가 아니라 “이벤트 발생 지점”
```

### 🔐 특징

```
✔️ Strong Consistency
항상 최신 데이터

✔️ 분산 시스템
여러 노드 구성 가능

✔️ 트랜잭션 지원
atomic update
```

### 🔍 Watch 기능 (중요🔥)

```
✔️ 개념
데이터 변경을 실시간 감시

🔹 예
Pod 생성 → etcd 변경 → 이벤트 발생 → watcher가 감지

🔹 누가 watch 하냐?
scheduler
controller-manager
kubelet

👉 전부 API Server 통해 watch
```

### 🧭 revision 이란?

```
✔️ 개념
etcd에서 데이터가 변경될 때마다 증가하는 전역 버전 번호

🔥 핵심 개념
클러스터 전체 변경 이력을 순서대로 기록하는 번호

👉 revision = “DB 전체의 commit 번호”
---
rev 100 → Pod 생성
rev 101 → Service 생성
rev 102 → Pod 상태 변경
---
👉 모든 변경이 하나의 타임라인으로 이어짐

📦 revision 종류 (중요🔥)
etcd에는 3가지 개념이 있다:

1️⃣ global revision
👉 etcd 전체 기준
---
rev 1 → key A 생성
rev 2 → key B 생성
rev 3 → key A 수정
---
👉 모든 key 공유

2️⃣ createRevision
👉 해당 key가 처음 생성된 시점

3️⃣ modRevision
👉 해당 key가 마지막으로 수정된 시점

🔄 실제 흐름
Pod 생성
rev 200 → Pod 생성 (createRevision=200, modRevision=200)
Pod 상태 변경
rev 201 → Running 상태 업데이트 (modRevision=201)

🔍 Watch와 revision (핵심🔥🔥🔥)
✔️ watch 동작
“rev 200 이후 변경된 것만 알려줘”

🔹 예
---
현재 rev = 200

watch 시작 (rev 200)
↓
rev 201 → 이벤트 발생
rev 202 → 이벤트 발생
---
👉 즉:
❗ revision 기반으로 “이후 변경만 추적”

🔹 예시
Key: /registry/pods/nginx
createRevision = 100
modRevision    = 105

👉 의미:
100: 처음 생성
105: 마지막 변경

참조 : watch는 이 revision 기반
```
