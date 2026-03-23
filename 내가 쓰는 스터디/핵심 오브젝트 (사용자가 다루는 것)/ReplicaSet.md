## 목차

- [1장 ReplicaSet 이란](#1장-replicaset-이란)
  - [1️⃣ ReplicaSet 이란 무엇인가](#1️⃣-replicaset-이란-무엇인가)
  - [2️⃣ 핵심 개념 (ReplicaSet의 본질)](#2️⃣-핵심-개념-replicaset의-본질)
  - [3️⃣ ReplicaSet YAML 기본 구조](#3️⃣-replicaset-yaml-기본-구조)
  - [5️⃣ ReplicaSet 내부 동작 흐름](#5️⃣-replicaset-내부-동작-흐름)
  - [6️⃣ ReplicaSet Controller](#6️⃣-replicaset-controller)
  - [7️⃣ Linux에서 ReplicaSet은 어디에 존재할까](#7️⃣-linux에서-replicaset은-어디에-존재할까)
  - [8️⃣ etcd에서 실제 확인](#8️⃣-etcd에서-실제-확인)
  - [9️⃣ Linux에서 실제로 생기는 것](#9️⃣-linux에서-실제로-생기는-것)
  - [🔟 ReplicaSet 동작 핵심 (Label Selector)](#-replicaset-동작-핵심-label-selector)
    - [⚠️ 중요한 특징](#️-중요한-특징)
  - [Kubernetes 전체 구조에서 위치](#kubernetes-전체-구조에서-위치)
- [2장 그렇다면 Controller manager 는 어떤 동작을 할까?](#2장-그렇다면-controller-manager-는-어떤-동작을-할까)
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

---

# 1장 ReplicaSet 이란

## 1️⃣ ReplicaSet 이란 무엇인가

ReplicaSet의 역할

```
👉 Pod 개수를 항상 원하는 개수로 유지하는 것
```

```
예를 들어 replicas = 3 이라면 3개가 항상 유지된다.

Pod 1
Pod 2
Pod 3

만약 Pod 2 죽었다면(CrashBack) ReplicaSet → 새 Pod 생성

그래서 다시

Pod 1
Pod 3
Pod 4 (새로 생성)
```

## 2️⃣ 핵심 개념 (ReplicaSet의 본질)

ReplicaSet은 사실 딱 3개만 중요하다

```
replicas
selector
template
```

구조

```
ReplicaSet
   │
   ├── replicas (원하는 Pod 개수)
   │
   ├── selector (어떤 Pod를 관리할지)
   │
   └── template (Pod 생성 방법)
```

예

```
replicas = 3
selector = app=nginx
template = nginx Pod
```

## 3️⃣ ReplicaSet YAML 기본 구조

```yaml
apiVersion: apps/v1
kind: ReplicaSet

metadata:
  name: my-replicaset
  namespace: default
  labels:
    app: nginx
  annotations:
    description: example

spec:
  replicas: 3
  minReadySeconds: 10

  selector:
    matchLabels:
      app: nginx
    matchExpressions:
      - key: tier
        operator: In
        values:
          - frontend

  template:
    metadata:
      labels:
        app: nginx
        tier: frontend
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
          resources:
            limits:
              cpu: "1"
              memory: "512Mi"
      restartPolicy: Always
```

## 5️⃣ ReplicaSet 내부 동작 흐름

ReplicaSet이 Pod를 만드는 과정

```
kubectl apply -f rs.yaml
```

흐름

```
kubectl
   ↓
API Server
   ↓
etcd (ReplicaSet 저장)
   ↓
ReplicaSet Controller
   ↓
현재 Pod 수 확인
   ↓
부족하면 Pod 생성
   ↓
Scheduler
   ↓
Node 선택
   ↓
kubelet
   ↓
container runtime
   ↓
container 실행
```

## 6️⃣ ReplicaSet Controller

ReplicaSet은 Controller Manager 내부에서 동작한다

```
kube-controller-manager
```

그 안에 존재하는 컨트롤러는 다음과 같다.

```
ReplicaSet Controller
Deployment Controller
Node Controller
Job Controller
ServiceAccount Controller
```

ReplicaSet Controller의 역할은 다음과 같다.

```
desired replicas
vs
actual pods

차이를 계산한다.

[예시]
desired = 3
actual = 1
그러면 2개 Pod 생성
```

## 7️⃣ Linux에서 ReplicaSet은 어디에 존재할까

ReplicaSet도 Node에는 존재하지 않는다

ReplicaSet은

```
Control Plane Object
```

저장 위치

```
etcd
```

key 예시

```
/registry/replicasets/default/my-replicaset
```

## 8️⃣ etcd에서 실제 확인

control-plane에서

```
➜  etcdctl get /registry/replicasets/default/my-replicaset

그러면 JSON 형태로 나온다.

{
  "kind":"ReplicaSet",
  "spec":{
     "replicas":3
  }
}
```

etcd db 파일은 다음 위치와 같다.

```
/var/lib/etcd
```

## 9️⃣ Linux에서 실제로 생기는 것

ReplicaSet 자체는 Linux에 아무것도 만들지 않는다.
<br>ReplicaSet이 만드는 것은 Pod 이다.

따라서 Node에서는

```
container
network namespace
veth
cgroup
pause container

같은 것들이 생긴다. ReplicaSet 자체는 논리적 객체다.
```

## 🔟 ReplicaSet 동작 핵심 (Label Selector)

ReplicaSet의 핵심은 selector이다.

```
selector:
  matchLabels:
    app: nginx
```

Pod의 Label

```
labels:
  app: nginx
```

그러면 ReplicaSet이 그 Pod를 관리한다

### ⚠️ 중요한 특징

ReplicaSet은 Pod를 직접 만들지 않아도 관리할 수 있다

예

```
[Pod]
labels:
  app: nginx

[ReplicaSet]
selector: app=nginx
```

그러면 기존 Pod도 관리 대상이 된다.

## Kubernetes 전체 구조에서 위치

```
API Server
   │
   │ (ReplicaSet 저장)
   ▼
etcd

Controller Manager
   │
   └── ReplicaSet Controller

ReplicaSet
   │
   ▼
Pod
   │
   ▼
kubelet
   │
   ▼
container runtime
```

---

# 2장 그렇다면 Controller manager 는 어떤 동작을 할까?

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

## 즉, 쿠버네티스 대부분의 컨트롤러는 같은 패턴이다.

```
Informer
   ↓
WorkQueue
   ↓
Worker goroutines
   ↓
sync()
```
