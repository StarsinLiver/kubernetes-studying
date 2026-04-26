## 목차

- [1장 Label](#1장-label)
  - [✅ 1. Label 이란?](#-1-label-이란)
  - [✅ 2. 어디서 실행되냐](#-2-어디서-실행되냐)
  - [✅ 3. Linux 어디에 저장되냐](#-3-linux-어디에-저장되냐)
  - [✅ 4. YAML 예시](#-4-yaml-예시)
    - [Label 의 규칙](#label-의-규칙)
    - [핵심 개념](#핵심-개념)
- [2장 Selector](#2장-selector)
  - [✅ 1. Selector 이란?](#-1-selector-이란)
  - [✅ 2. 어디서 실행되냐](#-2-어디서-실행되냐-1)
  - [✅ 3. 어디에 저장되나?](#-3-어디에-저장되나)
  - [✅ 4. Selector 종류 (중요🔥)](#-4-selector-종류-중요)
    - [✔ 1) matchLabels (가장 기본)](#-1-matchlabels-가장-기본)
    - [✔ 2) matchExpressions (고급)](#-2-matchexpressions-고급)
      - [✔ operator 종류](#-operator-종류)
- [3장 Label 과 Selector](#3장-label-과-selector)
  - [✅ 1. Label + Selector 관계 (핵심🔥)](#-1-label--selector-관계-핵심)
  - [✅ 2. 전체 구조](#-2-전체-구조)
  - [✅ 3. 중요한 흐름 (실제 동작🔥)](#-3-중요한-흐름-실제-동작)
  - [✅ 4. Linux 레벨에서 실제 확인](#-4-linux-레벨에서-실제-확인)
    - [✔ etcd (개념)](#-etcd-개념)
    - [✔ kubectl](#-kubectl)

---

# 1장 Label

## ✅ 1. Label 이란?

✔ 한줄 정리

```
리소스를 식별하고 그룹화하기 위한 key-value 메타데이터
👉 “Kubernetes는 Label로 구분하고 Selector로 연결한다”
```

✔ 어떤 역할을 하는가

```
Pod 구분 (frontend / backend)
Service가 대상 Pod 찾기
Deployment가 Pod 관리
NetworkPolicy 대상 지정
스케줄링 (nodeSelector)

👉 핵심
“이 리소스는 누구냐?”를 정의
```

## ✅ 2. 어디서 실행되냐

```
실행 개념 없음 ❌
그냥 metadata
```

👉 처리 주체

```
API Server (저장)
Controller (조회해서 사용)
```

## ✅ 3. Linux 어디에 저장되냐

👉 etcd에 저장됨

```
/registry/pods/<namespace>/<pod-name>
```

✔ 확인 방법

```
kubectl get pod --show-labels
kubectl describe pod <name>
```

## ✅ 4. YAML 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  # 이 부분!
  labels:
    app: nginx
    tier: frontend
    env: dev
```

### Label 의 규칙

| 항목        | 설명                     |
| ----------- | ------------------------ |
| key         | 문자열                   |
| value       | 문자열                   |
| 형식        | `key=value`              |
| prefix 가능 | `app.kubernetes.io/name` |

### 핵심 개념

🔥 1) 자유롭게 붙일 수 있음

```
제한 거의 없음
```

🔥 2) 동일 리소스 여러 label 가능

```
labels:
  app: nginx
  tier: frontend
  env: prod
```

🔥 3) identity가 아니라 “분류”

```
👉 Pod 이름이 identity
👉 Label은 grouping
```

---

# 2장 Selector

## ✅ 1. Selector 이란?

✔ 한줄 정리

```
👉 Label을 기준으로 리소스를 선택하는 조건
```

✔ 어떤 역할

```
Service → Pod 선택
Deployment → 관리 대상 선택
NetworkPolicy → 대상 Pod 지정

👉 핵심
“이 조건에 맞는 애들만 가져와”
```

## ✅ 2. 어디서 실행되냐

```
Controller 내부 로직에서 실행됨
```

예:

```
Service controller
Deployment controller
Calico (NetworkPolicy)
```

## ✅ 3. 어디에 저장되나?

```
👉 etcd에 같이 저장됨 (resource spec 안에)
```

## ✅ 4. Selector 종류 (중요🔥)

### ✔ 1) matchLabels (가장 기본)

```yaml
selector:
  matchLabels:
    app: nginx
```

👉 의미

```
app=nginx인 Pod만 선택
```

### ✔ 2) matchExpressions (고급)

```yaml
selector:
  matchExpressions:
    - key: env
      operator: In
      values:
        - prod
        - staging
```

#### ✔ operator 종류

| operator     | 의미     |
| ------------ | -------- |
| In           | 값 포함  |
| NotIn        | 값 제외  |
| Exists       | key 존재 |
| DoesNotExist | key 없음 |

---

# 3장 Label 과 Selector

## ✅ 1. Label + Selector 관계 (핵심🔥)

```
Pod (labels)
   ↑
Selector (조건)
   ↑
Service / Deployment / NetworkPolicy
```

👉 연결 구조:

```
Service
  └── selector → Pod 선택

Deployment
  └── selector → Pod 관리

NetworkPolicy
  └── selector → 대상 Pod 지정
```

## ✅ 2. 전체 구조

```
Namespace
 ├── Pod (labels)
 │     app=frontend
 │
 ├── Service (selector)
 │     app=frontend
 │
 └── NetworkPolicy (selector)
       app=frontend
```

## ✅ 3. 중요한 흐름 (실제 동작🔥)

예시 : Service → Pod 연결

1. Pod 생성

```yaml
labels:
  app: nginx
```

2. Service 생성

```yaml
selector:
  app: nginx
```

3. Service controller

```
👉 etcd 조회 → matching Pod 찾음
```

4. Endpoint 생성

```
👉 Service → Pod 연결됨
```

## ✅ 4. Linux 레벨에서 실제 확인

### ✔ etcd (개념)

```
etcdctl get /registry/pods --prefix
```

### ✔ kubectl

```
kubectl get pod -l app=nginx
```
