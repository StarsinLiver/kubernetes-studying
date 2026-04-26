## 목차

- [1장 Taint / Toleration](#1장-taint--toleration)
  - [✅ 1. Taint 과 Toleration 란?](#-1-taint-과-toleration-란)
  - [✅ 2. 어디서 실행되냐 (아키텍처🔥)](#-2-어디서-실행되냐-아키텍처)
    - [실행 주체](#실행-주체)
    - [흐름](#흐름)
  - [✅ 3. Linux 어디에 저장되냐](#-3-linux-어디에-저장되냐)
    - [✔ 확인 방법](#-확인-방법)
  - [✅ 4. 핵심 개념 (매우 중요🔥)](#-4-핵심-개념-매우-중요)
    - [🔥 1) Taint 구조](#-1-taint-구조)
    - [🔥 2) Effect 종류](#-2-effect-종류)
    - [🔥 3) Toleration은 “허용”이지 “강제 배치”가 아님](#-3-toleration은-허용이지-강제-배치가-아님)
  - [✅ 5. 전체 구조](#-5-전체-구조)
  - [✅ 6. 중요한 흐름 (실제 동작🔥)](#-6-중요한-흐름-실제-동작)
    - [예시 Effect : NoSchedule](#예시-effect--noschedule)
    - [예시 Effect : NoExecute (중요🔥)](#예시-effect--noexecute-중요)
  - [✅ 7. CLI 및 YAML 완전 상세 설명](#-7-cli-및-yaml-완전-상세-설명)
    - [✔ Node에 Taint 추가](#-node에-taint-추가)
    - [✔ Node에 Taint 제거](#-node에-taint-제거)
    - [✔ Pod toleration](#-pod-toleration)
  - [✅ 8. 실전 예제](#-8-실전-예제)
    - [✔ 전용 DB 노드 만들기](#-전용-db-노드-만들기)
    - [✔ 장애 노드 자동 제거](#-장애-노드-자동-제거)

---

# 1장 Taint / Toleration

## ✅ 1. Taint 과 Toleration 란?

한줄 정리

```
Taint는 “노드에 붙이는 거부 규칙”
Toleration은 “파드가 그 규칙을 무시할 수 있는 허용권”
```

어떤 역할을 하는가

```
특정 노드에 아무 파드나 올라가는 것 방지
전용 노드 (GPU, DB, 시스템 노드) 보호
장애 노드에서 파드 축출
유지보수/드레인 시 제어

👉 핵심:
“노드가 파드를 밀어내고, 파드가 그걸 견딜 수 있냐 없냐”
```

## ✅ 2. 어디서 실행되냐 (아키텍처🔥)

### 실행 주체

Scheduler

```
파드 생성 시 배치 판단
```

Kubelet

```
이미 올라간 파드 유지/퇴출 결정
```

### 흐름

```
Pod 생성
   ↓
Scheduler
   ↓ (Taint 확인)
배치 가능 여부 결정
   ↓
Node 배치
   ↓
Kubelet (Taint 감지)
   ↓
Toleration 없으면 제거
```

## ✅ 3. Linux 어디에 저장되냐

👉 etcd에 저장됨

```
/registry/nodes/<node-name>
/registry/pods/<pod-name>
```

### ✔ 확인 방법

Node taint 확인

```
kubectl describe node <node-name>
```

Pod toleration 확인

```
kubectl get pod <pod> -o yaml
```

## ✅ 4. 핵심 개념 (매우 중요🔥)

### 🔥 1) Taint 구조

```
key=value:effect

예:
dedicated=database:NoSchedule
```

### 🔥 2) Effect 종류

| Effect           | 의미                  |
| ---------------- | --------------------- |
| NoSchedule       | 새로운 파드 배치 금지 |
| PreferNoSchedule | 가능하면 피함         |
| NoExecute        | 기존 파드까지 제거    |

### 🔥 3) Toleration은 “허용”이지 “강제 배치”가 아님

👉 헷갈리는 포인트

```
toleration 있음 → 배치 “가능”
반드시 배치됨 ❌
```

## ✅ 5. 전체 구조

```
Node
 ├── Taint (거부 규칙)
 │
Pod
 └── Toleration (허용 규칙)
```

## ✅ 6. 중요한 흐름 (실제 동작🔥)

### 예시 Effect : NoSchedule

1. Node에 taint 있음

```
dedicated=db:NoSchedule
```

2. Pod 생성

```
toleration 없음 → 배치 실패
toleration 있음 → 배치 가능
```

### 예시 Effect : NoExecute (중요🔥)

1. Node에 taint 추가됨
2. 기존 Pod 확인

```
toleration 없음 → 즉시 제거
toleration 있음 → 유지
```

## ✅ 7. CLI 및 YAML 완전 상세 설명

### ✔ Node에 Taint 추가

```
kubectl taint nodes <node이름> dedicated=db:NoSchedule
```

### ✔ Node에 Taint 제거

```
kubectl taint nodes <node이름> dedicated=db:NoSchedule-

맨 뒤에 - 붙임
```

### ✔ Pod toleration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod

spec:
  tolerations:
    # taint key 선택
    - key: "dedicated"
      # Equal : key + value 일치
      # Exists : key만 존재하면 OK
      operator: "Equal"
      # operator Equal일 때만 사용
      value: "db"
      # NoSchedule / PreferNoSchedule / NoExecute
      effect: "NoSchedule"
      # NoExecute일 때만 사용
      # 60초 후 Pod 제거
      tolerationSeconds: 60
```

## ✅ 8. 실전 예제

### ✔ 전용 DB 노드 만들기

Node

```
kubectl taint nodes node1 dedicated=db:NoSchedule
```

Pod

```yaml
tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "db"
    effect: "NoSchedule"
```

👉 결과:

```
DB Pod만 node1에 올라감
```

### ✔ 장애 노드 자동 제거

```
node.kubernetes.io/not-ready:NoExecute

👉 kubelet 자동 추가
```
