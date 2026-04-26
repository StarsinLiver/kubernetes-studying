## 목차

- [1장 ResourceQuota 란?](#1장-resourcequota-란)
  - [✅ 1. ResourceQuota](#-1-resourcequota)
  - [✅ 2. YAML 예시](#-2-yaml-예시)
    - [옵션 상세](#옵션-상세)
  - [✅ 3. 동작 위치](#-3-동작-위치)

---

# 1장 ResourceQuota 란?

## ✅ 1. ResourceQuota

✔ 한줄 정리

```
👉 Namespace 전체 리소스 사용량 제한
```

✔ 역할

Namespace 단위로 제한

```
Pod 개수
CPU / Memory
Storage
```

## ✅ 2. YAML 예시

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "4"
    limits.memory: 8Gi
```

### 옵션 상세

spec.hard 란?

```
👉 “절대 넘으면 안되는 값”
```

| 옵션            | 의미               |
| --------------- | ------------------ |
| pods            | 최대 Pod 개수      |
| requests.cpu    | 최소 보장 CPU 총합 |
| limits.cpu      | 최대 CPU 총합      |
| requests.memory | 최소 메모리        |
| limits.memory   | 최대 메모리        |

## ✅ 3. 동작 위치

```
API Server에서 검증
Scheduler와 연결
```

✔ 확인

```
kubectl describe quota -n dev
```

✔ 핵심 개념

```
Namespace 전체 제한
초과하면 생성 실패
운영 환경에서 필수
```
