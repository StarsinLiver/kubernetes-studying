## 목차

- [1장 kube-controller-manager 란?](#1장-kube-controller-manager-란)
    - [🧠 역할](#-역할)
    - [⚙️ 전체 흐름 (핵심🔥)](#️-전체-흐름-핵심)
    - [핵심 개념: Reconciliation Loop](#핵심-개념-reconciliation-loop)
    - [🧩 주요 컨트롤러들](#-주요-컨트롤러들)
    - [🔄 동작 방식](#-동작-방식)
    - [💥 핵심 특징](#-핵심-특징)
- [2장 예시로 ReplicaSet Controller 를 살펴보자 (나머지 컨트롤러도 동작은 같다.)](#2장-예시로-replicaset-controller-를-살펴보자-나머지-컨트롤러도-동작은-같다)
  - [1️⃣ Controller의 핵심 개념](#1️⃣-controller의-핵심-개념)
  - [2️⃣ kube-controller-manager 는 무엇인가](#2️⃣-kube-controller-manager-는-무엇인가)
  - [3️⃣ ReplicaSet Controller의 실제 역할](#3️⃣-replicaset-controller의-실제-역할)
  - [4️⃣ 내부 동작 흐름](#4️⃣-내부-동작-흐름)
    - [etcd 저장](#etcd-저장)
    - [여기서 Controller Manager 등장](#여기서-controller-manager-등장)
  - [5️⃣ ReplicaSet Controller 알고리즘](#5️⃣-replicaset-controller-알고리즘)
  - [6️⃣ Pod 생성은 누가 하는가](#6️⃣-pod-생성은-누가-하는가)
  - [8️⃣ 전체 흐름 (ReplicaSet)](#8️⃣-전체-흐름-replicaset)
  - [9️⃣ Controller는 API Server를 직접 읽지 않는다](#9️⃣-controller는-api-server를-직접-읽지-않는다)
  - [🔟 Controller Manager 내부 구조](#-controller-manager-내부-구조)
- [3장 ReplicaSet Controller 오해와 이해](#3장-replicaset-controller-오해와-이해)
  - [1️⃣ ReplicaSet Controller는 이 프로세스 내부의 Go 코드 일부다.](#1️⃣-replicaset-controller는-이-프로세스-내부의-go-코드-일부다)
  - [2️⃣ ReplicaSet Controller 내부 구조](#2️⃣-replicaset-controller-내부-구조)
  - [3️⃣ ReplicaSet 생성 시 실제 동작](#3️⃣-replicaset-생성-시-실제-동작)
    - [etcd 저장](#etcd-저장-1)
    - [Controller가 watch 하고 있음](#controller가-watch-하고-있음)
    - [Informer가 이벤트 수신](#informer가-이벤트-수신)
    - [Worker가 하는 일](#worker가-하는-일)
    - [즉, 쿠버네티스 대부분의 컨트롤러는 같은 패턴이다.](#즉-쿠버네티스-대부분의-컨트롤러는-같은-패턴이다)
- [4장 각 controller의 내부 원리는 무엇인가](#4장-각-controller의-내부-원리는-무엇인가)
  - [1️⃣ Informer는 어떻게 WATCH를 구현하는가](#1️⃣-informer는-어떻게-watch를-구현하는가)
    - [1. 핵심 역할](#1-핵심-역할)
    - [2. 구조](#2-구조)
    - [3. 실제 동작 흐름](#3-실제-동작-흐름)
      - [① LIST](#-list)
      - [② WATCH](#-watch)
    - [4. 실제 Linux 네트워크 레벨](#4-실제-linux-네트워크-레벨)
    - [5. Informer의 진짜 목적](#5-informer의-진짜-목적)
  - [2️⃣ WorkQueue는 어떤 자료구조인가](#2️⃣-workqueue는-어떤-자료구조인가)
    - [1. 역할](#1-역할)
    - [2. 기본 구조](#2-기본-구조)
    - [3. 실제 자료구조](#3-실제-자료구조)
    - [4. 왜 dirty set이 필요할까](#4-왜-dirty-set이-필요할까)
    - [5. Rate Limiting Queue](#5-rate-limiting-queue)
  - [3️⃣ syncReplicaSet() 내부 코드](#3️⃣-syncreplicaset-내부-코드)
    - [1. syncReplicaSet 흐름](#1-syncreplicaset-흐름)
    - [2. 실제 Pod 생성 코드](#2-실제-pod-생성-코드)
    - [3. Pod Template 변환](#3-pod-template-변환)
    - [4. API Server 호출](#4-api-server-호출)
    - [5. 그 다음 실제 흐름](#5-그-다음-실제-흐름)
  - [5️⃣ 전체 흐름 (Controller 내부)](#5️⃣-전체-흐름-controller-내부)

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

---

# 2장 예시로 ReplicaSet Controller 를 살펴보자 (나머지 컨트롤러도 동작은 같다.)

## 1️⃣ Controller의 핵심 개념

쿠버네티스의 거의 모든 기능은 Controller 패턴으로 동작한다.

핵심 아이디어는 이것이다.

```
desired state
vs
current state
```

즉

```
원하는 상태
vs
현재 상태

차이를 계속 비교해서 같게 맞춘다. 이걸 Control Loop 라고 한다.
```

구조

```
while(true) {
   현재 상태 확인
   원하는 상태 확인
   차이 계산
   차이 수정
}
```

ReplicaSet이라면

```
desired = 3 pods
actual = 1 pod

그러면

2개 pod 생성
```

## 2️⃣ kube-controller-manager 는 무엇인가

Controller들을 모아놓은 프로세스 하나다.

프로세스 이름

```
kube-controller-manager
```

컨트롤 플레인에서 실행된다.

확인

```
ps -ef | grep controller
```

예

```
kube-controller-manager
```

대표적인 것들

```
ReplicaSet Controller
Deployment Controller
Node Controller
Job Controller
Endpoint Controller
ServiceAccount Controller
Namespace Controller
Garbage Collector
```

즉

```
Controller Manager
   ├── ReplicaSet Controller
   ├── Deployment Controller
   ├── Job Controller
   ├── Node Controller
   └── ...
```

## 3️⃣ ReplicaSet Controller의 실제 역할

ReplicaSet Controller는 이 두 개를 계속 감시한다.

```
ReplicaSet object
Pod object
```

목표

```
ReplicaSet.spec.replicas
=
실제 Pod 개수
```

## 4️⃣ 내부 동작 흐름

### etcd 저장

ReplicaSet 생성

```
kubectl apply -f rs.yaml
```

흐름

```
kubectl
   ↓
API Server
   ↓
etcd 저장
```

ReplicaSet object

```
/registry/replicasets/default/my-rs
```

```
[확인방법]
➜  etcdctl get /registry/replicasets/default/my-replicaset
```

여기까지는 단순 저장이다.

### 여기서 Controller Manager 등장

ReplicaSet Controller는 API Server를 계속 watch 한다.

```
watch /replicasets
watch /pods
```

이벤트 발생 -> ReplicaSet 생성
<br> 이 이벤트를 Controller 가 받는다.

```
ReplicaSet Controller
   ↓
syncReplicaSet()
```

## 5️⃣ ReplicaSet Controller 알고리즘

내부 로직은 거의 이렇게 생겼다.

```
function syncReplicaSet(rs):

   desired = rs.spec.replicas

   pods = getPodsBySelector(rs.selector)

   actual = len(pods)

   if actual < desired:
       createPod()

   if actual > desired:
       deletePod()
```

## 6️⃣ Pod 생성은 누가 하는가

중요한 포인트다.

ReplicaSet Controller가 직접 container를 만들지 않는다.

Controller가 하는 일

```
Pod Object 생성
```

이를 API Server로 보낸다.

```
POST /api/v1/pods
```

그러면 Pod object 저장된다.

```
API Server
   ↓
etcd
```

Pod이 생성되면 Scheduler가 등장한다.

```
Pod
   ↓
nodeName 없음
   ↓
Scheduler
   ↓
Node 선택
```

Pod 업데이트

```
spec.nodeName = node1
```

그 다음 kubelet 이 Pod를 본다.

```
watch pods (Pod 발견)
container runtime 호출 (container 생성)
```

## 8️⃣ 전체 흐름 (ReplicaSet)

```
kubectl apply
   ↓
API Server
   ↓
etcd (ReplicaSet 저장)
   ↓
ReplicaSet Controller (watch)
   ↓
Pod object 생성
   ↓
API Server
   ↓
etcd (Pod 저장)
   ↓
Scheduler
   ↓
Node 선택
   ↓
kubelet
   ↓
container runtime
   ↓
container 생성
```

## 9️⃣ Controller는 API Server를 직접 읽지 않는다

Controller는 보통 해당 패턴을 사용한다.

```
LIST (전체 읽기)
   ↓
WATCH (이벤트 스트림)
```

예

```
Pod Added
Pod Deleted
ReplicaSet Updated
```

이걸 Shared Informer 라고 한다.

## 🔟 Controller Manager 내부 구조

ReplicaSet Controller 내부 구조

```
API Server Watch
     ↓
Informer
     ↓
WorkQueue
     ↓
Worker
     ↓
syncReplicaSet()
```

즉

```
event-driven + loop
```

---

# 3장 ReplicaSet Controller 오해와 이해

질문

```
replicaset 을 생성한다는것은 replicaset controller 의 syncReplicaset() 함수를
사용하는 worker 를 하나 더 만드는 것 뿐인 것 같은데
이는 자식 프로세스로 thread 로 되는건가?
```

결론부터 말하면

```
ReplicaSet을 생성한다고 해서 새로운 worker / thread / process가 생성되는 것은 아니다.
```

즉

```
ReplicaSet Controller는 이미 실행 중인 worker들이 여러 ReplicaSet을 처리하는 구조다.
```

## 1️⃣ ReplicaSet Controller는 이 프로세스 내부의 Go 코드 일부다.

```
kube-controller-manager
   └── ReplicaSet Controller
```

즉 별도 프로세스가 아니다.

## 2️⃣ ReplicaSet Controller 내부 구조

실제 구조는 이렇게 생겼다.

```
ReplicaSet Controller
   │
   ├── Informer (watch API Server)
   │
   ├── WorkQueue
   │
   └── Worker Threads
         ├── worker1
         ├── worker2
         ├── worker3
         └── worker4
```

worker 개수는 보통

```
--concurrent-replicaset-syncs 옵션으로 설정된다.

기본값은 보통 5이며
즉 5개의 worker goroutine이 존재한다.
```

```
> goroutine 은 OS Thread를 사용한다.
goroutine
   ↓
Go runtime scheduler
   ↓
OS thread

> goroutine은 Linux에서 직접 보이지 않는다.
```

## 3️⃣ ReplicaSet 생성 시 실제 동작

### etcd 저장

ReplicaSet 생성

```
kubectl apply -f rs.yaml
```

흐름

```
kubectl
   ↓
API Server
   ↓
etcd
```

### Controller가 watch 하고 있음

ReplicaSet Controller는

```
WATCH /replicasets
```

를 하고 있다.

ReplicaSet 생성 이벤트 발생

```
ADDED event
```

### Informer가 이벤트 수신

```
Informer
   ↓
enqueue ReplicaSet key
   ↓
WorkQueue
```

큐에 들어간다.

```
"default/my-replicaset"
```

### Worker가 하는 일

이미 존재하는 worker가 큐에서 꺼낸다.

```
worker goroutine
   ↓
syncReplicaSet("default/my-replicaset")
```

즉, 기존 worker가 처리하는 것이다.

### 즉, 쿠버네티스 대부분의 컨트롤러는 같은 패턴이다.

```
Informer
   ↓
WorkQueue
   ↓
Worker goroutines
   ↓
sync()
```

---

# 4장 각 controller의 내부 원리는 무엇인가

## 1️⃣ Informer는 어떻게 WATCH를 구현하는가

### 1. 핵심 역할

Informer는 API Server의 object 변화를 실시간으로 감시한다.

예

```
ReplicaSet 생성
Pod 삭제
Pod 업데이트
Node 상태 변경
```

Controller는 이걸 받아서 작업한다.

### 2. 구조

Informer 내부 구조

```
API Server
   │
   │ WATCH
   ▼
Reflector
   │
   ▼
Delta FIFO Queue
   │
   ▼
Local Store (cache)
   │
   ▼
Event Handler
```

### 3. 실제 동작 흐름

처음 시작할 때

#### ① LIST

전체 object 가져온다

```
GET /api/v1/pods
```

응답은 다음과 같을 것이다.

```
{
 "items":[
   pod1,
   pod2,
   pod3
 ]
}
```

이걸 Local Cache에 저장한다.

#### ② WATCH

그 다음부터는 streaming 연결을 유지한다.

```
GET /api/v1/pods?watch=true
```

API Server는 HTTP streaming으로 이벤트를 계속 보낸다.

```
{"type":"ADDED","object":{pod}}
{"type":"MODIFIED","object":{pod}}
{"type":"DELETED","object":{pod}}
```

즉

```
HTTP connection
   │
   └─ 계속 열린 상태
```

이게 WATCH streaming이다.

### 4. 실제 Linux 네트워크 레벨

API Server와 Controller Manager 사이에는

```
HTTP/2 or HTTP chunked stream
```

으로 연결된다.

확인 예

```
ss -ntp
```

또는

```
tcpdump port 6443
```

API Server 포트

```
6443
```

### 5. Informer의 진짜 목적

Controller가 매번 API Server를 조회하면

```
10000 pod
100 controller

즉, 조회 개수가 너무 많다.
```

이면

```
API Server overload
```

그래서 local cache 를 사용한다.

```
따라서, Controller는 cache에서 조회한다.
```

## 2️⃣ WorkQueue는 어떤 자료구조인가

### 1. 역할

```
WorkQueue는 controller 작업 대기열이다.
```

예

```
ReplicaSet 변경
Pod 삭제
```

이런 이벤트가 들어온다.

### 2. 기본 구조

WorkQueue 내부 구조

```
queue
 ├── set
 ├── slice
 └── rate limiter
```

핵심 특징

```
중복 제거
rate limiting
retry
```

### 3. 실제 자료구조

쿠버네티스 코드 기준

```
type Type struct {
    queue []t
    dirty set
    processing set
}
```

설명

```
queue       실제 작업 목록
dirty       이미 queue에 있는 key
processing  현재 처리 중
```

### 4. 왜 dirty set이 필요할까

예

```
Pod update
Pod update
Pod update

3번 발생
```

하지만 queue에는 pod1 한번만 들어간다.

```
즉, deduplication
```

### 5. Rate Limiting Queue

Controller 실패 시 retry(재시도) 한다.

예시

```
Pod 생성 실패
```

그러면

```
1초 후 retry
2초 후 retry
4초 후 retry

이걸 Exponential Backoff 라고 한다.
```

## 3️⃣ syncReplicaSet() 내부 코드

이건 ReplicaSet Controller의 핵심이다.

실제 코드 위치

```
pkg/controller/replicaset/replica_set.go
```

### 1. syncReplicaSet 흐름

간단화하면

```go
func (rsc *ReplicaSetController) syncReplicaSet(key string) {

  rs := getReplicaSet(key)

  pods := listPods(rs.selector)

  diff := rs.spec.replicas - len(pods)

  if diff > 0 {
     createPods()
  }

  if diff < 0 {
     deletePods()
  }
}
```

### 2. 실제 Pod 생성 코드

```
func (rsc *ReplicaSetController) createPods(rs *apps.ReplicaSet, n int) {
    for i := 0; i < n; i++ {
        pod := newPodFromTemplate(rs.Spec.Template)
        rsc.kubeClient.CoreV1().Pods(rs.Namespace).Create(context, pod)
    }
}
```

핵심

```
Pod object 생성
API Server에 POST
```

### 3. Pod Template 변환

ReplicaSet YAML

```
template:
  spec:
    containers:
      - image: nginx
```

코드에서는 PodTemplateSpec으로, 이걸 Pod로 변환한다.

### 4. API Server 호출

실제 REST API

```
POST /api/v1/namespaces/default/pods
```

```
즉, ReplicaSet Controller는 container 생성 X

단지 Pod object 생성만 한다.
```

### 5. 그 다음 실제 흐름

Pod 생성 후 Scheduler와 kubelet 이 등장한다.

```
Pod
nodeName 없음

[Scheduler]
node 선택

[업데이트]
spec.nodeName=node1

[kubelet]
kubelet이 container runtime으로 Pod를 실행한다.
```

## 5️⃣ 전체 흐름 (Controller 내부)

전체 구조

```
API Server
   │
   │ LIST/WATCH
   ▼
Informer
   │
   ▼
Local Cache
   │
   ▼
Event Handler
   │
   ▼
WorkQueue
   │
   ▼
Worker Goroutine
   │
   ▼
syncReplicaSet()
   │
   ▼
Pod 생성
   │
   ▼
API Server
```
