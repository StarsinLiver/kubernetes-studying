## 목차

- [1장 Annotation](#1장-annotation)
  - [✅ 1. Annotation 이란?](#-1-annotation-이란)
  - [✅ 2. Label vs Annotation (핵심 차이🔥)](#-2-label-vs-annotation-핵심-차이)
  - [✅ 3. 어디서 실행되냐](#-3-어디서-실행되냐)
  - [✅ 4. Linux 어디에 저장되냐](#-4-linux-어디에-저장되냐)
  - [✅ 5. YAML 구조](#-5-yaml-구조)
  - [✅ 6. 핵심 개념 (중요🔥)](#-6-핵심-개념-중요)
  - [✅ 7. 실제 핵심 사용처 (실무🔥)](#-7-실제-핵심-사용처-실무)
    - [✔ 1) Ingress Controller (가장 중요🔥)](#-1-ingress-controller-가장-중요)
    - [✔ 2) Prometheus (모니터링)](#-2-prometheus-모니터링)
    - [✔ 3) Helm](#-3-helm)
    - [✔ 4) kubectl last-applied](#-4-kubectl-last-applied)
  - [✅ 8. 전체 구조](#-8-전체-구조)
  - [✅ 9. 동작 흐름 (Ingress 예시🔥)](#-9-동작-흐름-ingress-예시)
  - [✅ 10. 핵심 요약](#-10-핵심-요약)

---

# 1장 Annotation

## ✅ 1. Annotation 이란?

한줄 정리

```
👉 리소스에 부가적인 설정이나 정보를 저장하는 key-value 메타데이터 (선택에는 사용 안됨)
```

어떤 역할을 하는가

```
Ingress 설정 (rewrite, SSL 등)
Helm / 배포 도구 메타데이터
모니터링/로그 설정
컨트롤러 동작 제어

👉 핵심:
“시스템이나 도구에게 전달하는 설정값”
```

## ✅ 2. Label vs Annotation (핵심 차이🔥)

| 구분          | Label               | Annotation          |
| ------------- | ------------------- | ------------------- |
| 목적          | 선택/필터링         | 설정/설명           |
| selector 사용 | 가능                | 불가능 ❌           |
| 크기          | 작음                | 큼 가능             |
| 사용처        | Service, Deployment | Ingress, Controller |

```
👉 한줄 요약:

Label → “누구냐”
Annotation → “어떻게 동작해야 하냐”
```

## ✅ 3. 어디서 실행되냐

```
실행 개념 없음 ❌
그냥 metadata
```

👉 실제 사용:

```
Controller가 읽어서 동작 변경
```

예:

```
Ingress Controller
Cert Manager
Prometheus
```

## ✅ 4. Linux 어디에 저장되냐

👉 etcd에 저장됨

```
/registry/<resource>/<namespace>/<name>
```

확인방법

```
kubectl get pod <name> -o yaml
kubectl describe pod <name>
```

## ✅ 5. YAML 구조

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod

  # annotation
  annotations:
    description: "이건 테스트 Pod"
    owner: "team-a"
```

## ✅ 6. 핵심 개념 (중요🔥)

🔥 1) Selector에 사용 불가

```yaml
selector:
  annotations: ❌ 불가능
```

🔥 2) 크기 제한 거의 없음

```
긴 JSON도 가능
```

🔥 3) 시스템/툴 의존

```
👉 Kubernetes가 해석 안함
👉 “읽는 쪽이 해석”
```

## ✅ 7. 실제 핵심 사용처 (실무🔥)

### ✔ 1) Ingress Controller (가장 중요🔥)

예: NGINX Ingress Controller

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
```

👉 의미:

```
URL rewrite
HTTPS 강제
```

### ✔ 2) Prometheus (모니터링)

```yaml
annotations:
  prometheus.io/scrape: "true"
  prometheus.io/port: "8080"
```

### ✔ 3) Helm

```yaml
annotations:
  meta.helm.sh/release-name: my-app
```

### ✔ 4) kubectl last-applied

```
kubectl.kubernetes.io/last-applied-configuration
```

👉 apply 관리용

## ✅ 8. 전체 구조

```
Resource
 ├── metadata
 │     ├── labels (선택용)
 │     └── annotations (설정용🔥)
```

## ✅ 9. 동작 흐름 (Ingress 예시🔥)

```
Ingress 생성
   ↓
Annotation 포함
   ↓
Ingress Controller가 읽음
   ↓
nginx 설정 파일 생성
   ↓
동작 변경
```

👉 핵심:

```
Annotation → Controller → 실제 시스템 설정
```

## ✅ 10. 핵심 요약

1. Annotation = 설정/부가정보
2. selector 불가
3. Controller가 읽어서 동작 변경
4. etcd에 저장됨
5. Ingress, Prometheus, Helm에서 핵심

```
👉 “Annotation은 Kubernetes가 아니라 ‘컨트롤러가 해석해서 실제 동작을 바꾸는 설정값’이다”
```
