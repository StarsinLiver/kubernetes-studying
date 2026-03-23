## 목차

- [1장 Deployment](#1장-deployment)
  - [1️⃣ Deployment의 역할](#1️⃣-deployment의-역할)
  - [2️⃣ Deployment YAML 구조](#2️⃣-deployment-yaml-구조)
  - [3️⃣ Deployment 내부 구조](#3️⃣-deployment-내부-구조)
  - [4️⃣ Deployment Controller](#4️⃣-deployment-controller)
  - [5️⃣ 실제 생성 흐름](#5️⃣-실제-생성-흐름)
  - [6️⃣ Deployment 업데이트 (Rolling Update)](#6️⃣-deployment-업데이트-rolling-update)
  - [7️⃣ RollingUpdate 설정](#7️⃣-rollingupdate-설정)
  - [8️⃣ Deployment Rollback](#8️⃣-deployment-rollback)
  - [9️⃣ Linux에서 실제 존재 위치](#9️⃣-linux에서-실제-존재-위치)
  - [🔟 실제 확인 명령어](#-실제-확인-명령어)
  - [11. 실제 Kubernetes 구조](#11-실제-kubernetes-구조)
  - [12. Deployment Controller 내부 로직](#12-deployment-controller-내부-로직)
  - [🔥 Kubernetes에서 가장 중요한 구조](#-kubernetes에서-가장-중요한-구조)
- [2장 Deployment Controller 의 내부 로직](#2장-deployment-controller-의-내부-로직)
  - [1️⃣ Deployment Controller의 syncDeployment() 알고리즘](#1️⃣-deployment-controller의-syncdeployment-알고리즘)
    - [내부 처리 흐름](#내부-처리-흐름)
  - [2️⃣ 새로운 ReplicaSet을 찾는 과정](#2️⃣-새로운-replicaset을-찾는-과정)
  - [3️⃣ RollingUpdate 핵심 알고리즘](#3️⃣-rollingupdate-핵심-알고리즘)
    - [maxSurge 계산](#maxsurge-계산)
    - [maxUnavailable 계산](#maxunavailable-계산)
  - [4️⃣ Rolling Update 실제 단계](#4️⃣-rolling-update-실제-단계)
    - [Step 1. 새 ReplicaSet scale up](#step-1-새-replicaset-scale-up)
    - [Step 2. old ReplicaSet scale down](#step-2-old-replicaset-scale-down)
    - [Step 3. new ReplicaSet scale up](#step-3-new-replicaset-scale-up)
    - [Step 4. old ReplicaSet scale down](#step-4-old-replicaset-scale-down)
    - [Step 5. scale up](#step-5-scale-up)
    - [Step 6. scale down (완료)](#step-6-scale-down-완료)
  - [5️⃣ 실제 Kubernetes 코드 흐름](#5️⃣-실제-kubernetes-코드-흐름)
  - [6️⃣ 실제 Controller 구조](#6️⃣-실제-controller-구조)
  - [7️⃣ 실제 API Server 호출](#7️⃣-실제-api-server-호출)
  - [8️⃣ 전체 Kubernetes 흐름](#8️⃣-전체-kubernetes-흐름)

---

# 1장 Deployment

Deployment는 사실 새로운 실행 단위가 아니라 “상위 컨트롤러”다.
<br>즉 실제 구조는 이렇게 된다.

ReplicaSet 을 보고 오면 이해하기 편하다.

```
Deployment
   ↓
ReplicaSet
   ↓
Pod
   ↓
Container
```

## 1️⃣ Deployment의 역할

Deployment의 핵심 역할은 애플리케이션의 버전 관리와 업데이트 관리다.

ReplicaSet은 단순히 Pod 개수 유지만 한다.
<br>하지만 Deployment는 Rolling Update, Rollback, Version 관리, ReplicaSet 관리 를 담당한다.

[ReplicaSet]

```
Pod 개수 유지
```

[Deployment]

```
Rolling Update
Rollback
Version 관리
ReplicaSet 관리
```

예

```
nginx:1.24 → nginx:1.25
```

Deployment는

```
새 ReplicaSet 생성
기존 ReplicaSet 축소
새 ReplicaSet 확장
```

을 자동으로 한다.

## 2️⃣ Deployment YAML 구조

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deploy

spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx

  # Optional (Rolling Update)
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```

```
여기서 중요한 구조는 spec.template 이다.
왜냐하면 이게 ReplicaSet의 Pod Template로 그대로 복사된다.
```

## 3️⃣ Deployment 내부 구조

Deployment는 실제로 ReplicaSet을 생성한다.

구조

```
Deployment
   │
   ├── ReplicaSet (old)
   │       ├── Pod
   │       ├── Pod
   │
   └── ReplicaSet (new)
           ├── Pod
           ├── Pod
```

```
즉, Deployment Controller가 ReplicaSet lifecycle을 관리한다.
```

## 4️⃣ Deployment Controller

```
Deployment는 kube-controller-manager안에 있는 Deployment Controller가 처리한다.
```

Controller 흐름

```
Deployment 생성
      ↓
Deployment Controller 감지
      ↓
ReplicaSet 생성
      ↓
ReplicaSet Controller
      ↓
Pod 생성
```

즉

```
Deployment Controller
   ↓
ReplicaSet Controller
   ↓
Pod
```

## 5️⃣ 실제 생성 흐름

Deployment 생성

```
kubectl apply -f deploy.yaml
```

흐름

```
kubectl
   ↓
API Server
   ↓
etcd (Deployment 저장)
   ↓
Deployment Controller WATCH
   ↓
ReplicaSet 생성
   ↓
ReplicaSet Controller
   ↓
Pod 생성
   ↓
Scheduler
   ↓
Node 선택
   ↓
kubelet
   ↓
Container 실행
```

## 6️⃣ Deployment 업데이트 (Rolling Update)

```
Deployment의 핵심 기능이다.
```

예

```
image: nginx:1.24 → image: nginx:1.25

업데이트 하면 Deployment Controller가 새 ReplicaSet 생성한다.
```

구조

```
ReplicaSet v1 (3 pods)
ReplicaSet v2 (0 pods)
```

```
[1]
v1 → 2 (2 pods)
v2 → 1 (1 pods)

[2]
v1 → 1 (1 pods)
v2 → 2 (2 pods)

[최종]
v1 → 0 (0 pods)
v2 → 3 (3 pods)
```

이걸 Rolling Update라고 한다.

## 7️⃣ RollingUpdate 설정

Deployment YAML

```yaml
strategy:
  type: RollingUpdate

  rollingUpdate:
    maxUnavailable: 1 # 동시에 내려갈 수 있는 Pod 수
    maxSurge: 1 # 추가로 생성 가능한 Pod 수
```

예

```
replicas = 3
maxUnavailable = 1
maxSurge = 1
```

가능한 상태

```
2 ~ 4 pods
```

## 8️⃣ Deployment Rollback

Deployment는 이전 ReplicaSet을 유지한다.

그래서

```
➜  kubectl rollout undo deployment nginx

하면 old ReplicaSet을 다시 확장한다.
```

## 9️⃣ Linux에서 실제 존재 위치

```
Deployment도 Node에는 존재하지 않는다.
Deployment object는 etcd에 저장된다.
```

etcd key 위치 예시

```
/registry/deployments/default/nginx-deploy
```

확인

```
➜  etcdctl get /registry/deployments/default/nginx-deploy
```

## 🔟 실제 확인 명령어

Deployment 확인

```
kubectl get deployment
```

ReplicaSet 확인

```
kubectl get rs
```

Pod 확인

```
kubectl get pods
```

관계 확인

```
kubectl describe deployment nginx-deploy
```

## 11. 실제 Kubernetes 구조

전체 구조

```
kube-controller-manager
   │
   ├── Deployment Controller
   │
   └── ReplicaSet Controller
```

흐름

```
Deployment
   ↓
ReplicaSet
   ↓
Pod
   ↓
Container
```

## 12. Deployment Controller 내부 로직

Deployment Controller도 ReplicaSet Controller와 동일한 패턴이다.

```
Informer
   ↓
WorkQueue
   ↓
Worker
   ↓
syncDeployment()
```

핵심 로직

```
새 ReplicaSet 필요 여부 확인
기존 ReplicaSet scale
RollingUpdate 수행
```

## 🔥 Kubernetes에서 가장 중요한 구조

대부분 애플리케이션은

```
Deployment
   ↓
ReplicaSet
   ↓
Pod
```

구조로 실행된다.

즉

```
Deployment = 애플리케이션 관리
ReplicaSet = Pod 개수 유지
Pod = 실제 실행 단위
```

---

# 2장 Deployment Controller 의 내부 로직

## 1️⃣ Deployment Controller의 syncDeployment() 알고리즘

Deployment Controller도 다른 컨트롤러와 동일한 패턴이다.

```
Informer
   ↓
WorkQueue
   ↓
worker goroutine
   ↓
syncDeployment()
```

```
worker가 큐에서 Deployment key를 꺼내면 syncDeployment("default/nginx-deploy") 가 실행된다.
```

### 내부 처리 흐름

syncDeployment()의 핵심 흐름은 대략 다음과 같다.

```
syncDeployment(deployment):

    rsList = getReplicaSets(deployment)

    podList = getPods(deployment)

    newRS = findNewReplicaSet(deployment, rsList)

    oldRS = findOldReplicaSets(rsList, newRS)

    if strategy == RollingUpdate:
        rollingUpdate(deployment, newRS, oldRS)

    if strategy == Recreate:
        recreate(deployment)
```

핵심은 ReplicaSet을 관리하는 것이다.
<br>Deployment Controller는 Pod을 직접 관리하지 않는다.

```
Deployment
   ↓
ReplicaSet
   ↓
Pod
```

## 2️⃣ 새로운 ReplicaSet을 찾는 과정

```
Deployment spec.template이 변경되면 새 ReplicaSet이 필요하다.
Kubernetes는 이를 PodTemplate hash로 판단한다.
```

template 예시

```
template:
  spec:
    containers:
      image: nginx:1.24 -> image: nginx:1.25 (변경)
```

그러면 Controller는 pod-template-hash를 계산한다.

pod-template-hash 예시

```
nginx-deploy-6c7f8d
nginx-deploy-9a4c2f
```

이 hash가 ReplicaSet 이름에 들어간다.

즉

```
ReplicaSet v1
ReplicaSet v2
```

## 3️⃣ RollingUpdate 핵심 알고리즘

Deployment YAML

```yaml
replicas: 3

strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1
```

### maxSurge 계산

maxSurge = 추가로 생성 가능한 Pod 수

```
[최대]
replicas + maxSurge

3 + 1 = 4
```

### maxUnavailable 계산

동시에 내려갈 수 있는 Pod 수

```
[최소]
replicas - maxUnavailable

3 - 1 = 2
```

## 4️⃣ Rolling Update 실제 단계

초기 상태

```
ReplicaSet v1 = 3 pods
ReplicaSet v2 = 0 pods
```

### Step 1. 새 ReplicaSet scale up

```
[변경]
v1 = 3 (3 pods)
v2 = 1 (1 pods)

총 4 pods

(maxSurge 사용)
```

### Step 2. old ReplicaSet scale down

```
v1 = 2 (2 pods)
v2 = 1 (1 pods)
```

### Step 3. new ReplicaSet scale up

```
v1 = 2 (2 pods)
v2 = 2 (2 pods)
```

### Step 4. old ReplicaSet scale down

```
v1 = 1 (1 pods)
v2 = 2 (1 pods)
```

### Step 5. scale up

```
v1 = 1 (1 pods)
v2 = 3 (1 pods)
```

### Step 6. scale down (완료)

```
v1 = 0 (1 pods)
v2 = 3 (1 pods)
```

## 5️⃣ 실제 Kubernetes 코드 흐름

Deployment Controller 코드 위치

```
pkg/controller/deployment/deployment_controller.go
```

핵심 함수

```
syncDeployment()
```

그 안에서 호출

```
rolloutRolling()
```

핵심 코드 (단순화)

```
func rolloutRolling(d *Deployment) {

    newRS := getNewReplicaSet()

    oldRS := getOldReplicaSets()

    scaleUp(newRS)

    scaleDown(oldRS)
}
```

scaleUp 코드

```
func scaleUp(rs, deployment) {

    maxSurge := resolveMaxSurge()

    desired := deployment.spec.replicas

    allowed := desired + maxSurge

    if totalPods < allowed {
        rs.spec.replicas++
    }
}
```

scaleDown 코드

```
func scaleDown(oldRS) {

    if availablePods > minAvailable {
        oldRS.spec.replicas--
    }
}
```

## 6️⃣ 실제 Controller 구조

Deployment Controller 내부 구조

```
Deployment Controller
   │
   ├── Informer
   │
   ├── WorkQueue
   │
   └── Worker Goroutines
         ↓
      syncDeployment()
         ↓
      rollingUpdate()
         ↓
      scaleReplicaSet()
```

## 7️⃣ 실제 API Server 호출

ReplicaSet scale 시 Controller는

```
PATCH /apis/apps/v1/namespaces/default/replicasets/nginx-rs

즉, spec.replicas 수정한다.
```

그 후

```
ReplicaSet Controller가 다시 동작해서 Pod 생성 / 삭제한다.
```

## 8️⃣ 전체 Kubernetes 흐름

Deployment 업데이트 시 전체 흐름

```
kubectl apply
      ↓
API Server
      ↓
etcd (Deployment 저장)
      ↓
Deployment Controller
      ↓
새 ReplicaSet 생성
      ↓
ReplicaSet scale 조정
      ↓
ReplicaSet Controller
      ↓
Pod 생성 / 삭제
      ↓
Scheduler
      ↓
Node 선택
      ↓
kubelet
      ↓
container 실행
```
