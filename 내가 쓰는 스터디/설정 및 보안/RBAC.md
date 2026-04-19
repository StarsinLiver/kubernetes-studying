## 목차

- [1장 RBAC 란?](#1장-rbac-란)
  - [1. RBAC 한줄 정의](#1-rbac-한줄-정의)
  - [2. 왜 필요하냐 (존재 이유)](#2-왜-필요하냐-존재-이유)
  - [3. 핵심 구성요소 (4개만 기억하면 끝)](#3-핵심-구성요소-4개만-기억하면-끝)
  - [4. 구성요소 하나씩](#4-구성요소-하나씩)
    - [1️⃣ Subject (주체)](#1️⃣-subject-주체)
    - [2️⃣ Role](#2️⃣-role)
    - [3️⃣ ClusterRole](#3️⃣-clusterrole)
    - [4️⃣ RoleBinding](#4️⃣-rolebinding)
    - [5️⃣ ClusterRoleBinding](#5️⃣-clusterrolebinding)
  - [5. 전체 구조](#5-전체-구조)
  - [6. 실제 동작 흐름](#6-실제-동작-흐름)
  - [7. YAML 구조 완전 분석](#7-yaml-구조-완전-분석)
  - [8. 실전 예제](#8-실전-예제)
  - [9. 내부 동작 (깊은 이해)](#9-내부-동작-깊은-이해)
  - [10. 확인 방법](#10-확인-방법)
- [2장 User | Group 란?](#2장-user--group-란)
  - [1. User | Group | ServiceAccount 란?](#1-user--group--serviceaccount-란)
  - [2. User 와 Group 이란 무엇일까?](#2-user-와-group-이란-무엇일까)
  - [3. User 와 Group 생성 방법 (실제 방식)](#3-user-와-group-생성-방법-실제-방식)
    - [방법 1️⃣ 클라이언트 인증서 (가장 기본, 소규모)](#방법-1️⃣-클라이언트-인증서-가장-기본-소규모)
      - [🔷 Group은 어떻게 생기냐](#-group은-어떻게-생기냐)
      - [🔷 RBAC에서 사용하는 방식](#-rbac에서-사용하는-방식)
    - [방법 2️⃣ OIDC (대규모)](#방법-2️⃣-oidc-대규모)
    - [방법 3️⃣ Static Token (거의 안씀)](#방법-3️⃣-static-token-거의-안씀)
- [3장 ServiceAccount](#3장-serviceaccount)
  - [1. ServiceAccount (중요)](#1-serviceaccount-중요)
  - [2. ServiceAccount + RBAC + Pod 연결 구조](#2-serviceaccount--rbac--pod-연결-구조)
  - [3. ServiceAccount 실제 내부 동작](#3-serviceaccount-실제-내부-동작)

---

# 1장 RBAC 란?

## 1. RBAC 한줄 정의

```
👉 누가(Who) 어떤 리소스에 무엇을(What) 할 수 있는지 정의하는 권한 시스템
```

## 2. 왜 필요하냐 (존재 이유)

쿠버네티스는 API 기반 시스템이다

```
kubectl → API Server → etcd
```

👉 문제:

```
아무나 Secret 읽으면?
아무나 Pod 삭제하면?
```

👉 해결

```
RBAC으로 접근 제한
```

## 3. 핵심 구성요소 (4개만 기억하면 끝)

```
User / ServiceAccount (누가)
        ↓
Role / ClusterRole (무엇을 할 수 있는지)
        ↓
RoleBinding / ClusterRoleBinding (연결)
```

## 4. 구성요소 하나씩

### 1️⃣ Subject (주체)

```
👉 “누가”
```

종류

```
User
Group
ServiceAccount (Pod가 사용하는 계정)
```

### 2️⃣ Role

```
👉 “무엇을 할 수 있는지 정의”
👉 namespace 범위
```

### 3️⃣ ClusterRole

```
👉 cluster 전체 범위
```

### 4️⃣ RoleBinding

```
👉 Role + Subject 연결
```

### 5️⃣ ClusterRoleBinding

```
👉 ClusterRole + Subject 연결
```

## 5. 전체 구조

```
[User / Pod]
      ↓
[Role or ClusterRole]
      ↓
[Binding]
      ↓
권한 부여
```

## 6. 실제 동작 흐름

```
kubectl get pods
   ↓
API Server
   ↓
1. 인증(Authentication)
   ↓
2. 인가(Authorization, RBAC)
   ↓
3. 허용 / 거부
```

```
👉 RBAC은 인가 단계에서 동작
```

## 7. YAML 구조 완전 분석

🔹 Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default

rules:
  # 값	   의미
  # ""     core (pods, services, secrets)
  # apps   deployments
  # batch  jobs
  - apiGroups: [""] # core API

    # 가능한 값:
    # pods
    # services
    # configmaps
    # secrets
    # deployments
    # * (전체)
    resources: ["pods"] # 대상 리소스

    # verb	  의미
    # get     하나 조회
    # list	  목록
    # watch	  변경 감지
    # create  생성
    # update  수정
    # patch	  일부 수정
    # delete  삭제
    # *       전체
    verbs: ["get", "list"] # 허용 동작
```

🔹 RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default

subjects:
  - kind: User # User | Group | ServiceAccount
    name: developer

roleRef: # 어떠한 Role 에 연결할지
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

🔹 ClusterRole

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-reader

rules:
  # 값	   의미
  # ""     core (pods, services, secrets)
  # apps   deployments
  # batch  jobs
  - apiGroups: [""] # API 그룹

    # 가능한 값:
    # pods
    # nodes (ClusterRole에서만 가능)
    # namespaces
    # configmaps
    # secrets
    # deployments
    # * (전체)
    resources: ["nodes"] # 대상 리소스
    verbs: ["get", "list", "watch", "create", "update", "delete"] # 허용 동작

    # 추가 옵션들
    # 특정 리소스만 허용
    resourceNames: ["my-pod"]
    # API URL 직접 접근 권한
    nonResourceURLs: ["/healthz"]
```

```
👉 특징
namespace 없음
cluster 전체
```

🔹 ClusterRoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-pods-global

subjects:
  - kind: User # User | Group | ServiceAccount
    name: developer
    apiGroup: rbac.authorization.k8s.io

  - kind: ServiceAccount # # User | Group | ServiceAccount
    name: app-sa
    namespace: default

roleRef:
  kind: ClusterRole
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

```
👉 cluster 전체 적용
```

## 8. 실전 예제

Secret 읽기 권한 부여

```yaml
kind: Role
metadata:
  name: secret-reader
  namespace: default

rules:
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get"]
---
kind: RoleBinding
subjects:
  - kind: ServiceAccount
    name: app-sa

roleRef:
  kind: Role
  name: secret-reader
```

```
👉 결과:

해당 Pod만 Secret 읽기 가능
```

## 9. 내부 동작 (깊은 이해)

API Server 내부

```
Request 들어옴
   ↓
Authentication (누군지 확인)
   ↓
Authorization (RBAC 확인)
   ↓
Rule 매칭
   ↓
허용 or 거부
```

👉 RBAC은

```
etcd에서 Role/Binding 조회
메모리에 캐싱
빠르게 검사
```

## 10. 확인 방법

```
kubectl auth can-i get pods
```

특정 사용자 기준:

```
kubectl auth can-i get secrets --as=developer
```

---

# 2장 User | Group 란?

## 1. User | Group | ServiceAccount 란?

✅ 한줄 정리

```
👉 User = 사람
👉 Group = 사용자 묶음
👉 ServiceAccount = Pod(애플리케이션)
```

🔹 1️⃣ User

```
👉 사람이 kubectl 사용할 때
```

예시 CLI

```
kubectl get pods
```

👉 인증

```
인증서
OIDC (로그인)
```

```
👉 Kubernetes 내부 객체 ❌
👉 외부 시스템에서 관리됨
```

🔹 2️⃣ Group

```
👉 여러 User 묶음
```

예:

```
dev-team
admin-team
```

```
👉 RBAC에서 한 번에 권한 부여
```

## 2. User 와 Group 이란 무엇일까?

```
👉 Kubernetes에는 “User / Group을 생성하는 API가 없다.”
```

🔷 한줄 핵심

```
👉 User / Group은 Kubernetes 밖에서 생성하고, Kubernetes는 “인증된 결과만 사용”한다
```

🔷 왜 User/Group 리소스가 없냐

쿠버네티스 구조:

```
[사용자]
   ↓
(외부 인증 시스템)
   ↓
API Server
   ↓
RBAC
```

👉 Kubernetes 역할

```
인증(Authentication) ❌ 직접 안함
인가(Authorization, RBAC) ⭕
```

👉 즉

```
“누군지 확인”은 외부
“무엇을 할 수 있는지”는 RBAC
```

## 3. User 와 Group 생성 방법 (실제 방식)

### 방법 1️⃣ 클라이언트 인증서 (가장 기본, 소규모)

1. 개인 키 생성

```
openssl genrsa -out developer.key 2048
```

2. CSR 생성

```
openssl req -new -key developer.key -out developer.csr -subj "/CN=developer/O=dev-team"

👉 여기 핵심:

CN = User 이름
O = Group 이름
```

3. 인증서 서명 (CA)

```
openssl x509 -req -in developer.csr \
  -CA ca.crt -CAkey ca.key \
  -CAcreateserial -out developer.crt -days 365
```

4. kubeconfig 등록

```
kubectl config set-credentials developer \
  --client-certificate=developer.crt \
  --client-key=developer.key
```

```
👉 이제 Kubernetes는:

User = developer
Group = dev-team

으로 인식함
```

#### 🔷 Group은 어떻게 생기냐

```
👉 따로 생성 ❌
👉 인증서에서 자동으로 결정됨
-subj "/CN=developer/O=dev-team/O=backend-team"
```

👉 결과

```
User: developer
Groups:
  - dev-team
  - backend-team
```

#### 🔷 RBAC에서 사용하는 방식

```
subjects:
  - kind: User
    name: developer

  - kind: Group
    name: dev-team

👉 Kubernetes는:

“이름만 보고 매칭”한다
```

### 방법 2️⃣ OIDC (대규모)

```
👉 회사 환경에서는 이거 많이 씀

예:
Google
Keycloak
Okta
```

흐름

```
사용자 로그인
   ↓
OIDC Provider (JWT 발급)
   ↓
API Server
   ↓
User / Group 추출
```

API Server 옵션

```
--oidc-issuer-url=https://accounts.google.com
--oidc-client-id=xxxxx
--oidc-username-claim=email
--oidc-groups-claim=groups
```

결과:

```
email → User
groups → Group
```

### 방법 3️⃣ Static Token (거의 안씀)

```csv
token,user,uid,group
abcd1234,developer,uid1,dev-team
```

👉 API Server 옵션으로 등록

---

# 3장 ServiceAccount

## 1. ServiceAccount (중요)

👉 Pod가 사용하는 계정

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
```

👉 특징

```
namespace 단위
Pod에 자동으로 붙음
토큰을 가짐
```

👉 토큰 위치

```
/var/run/secrets/kubernetes.io/serviceaccount/token
```

## 2. ServiceAccount + RBAC + Pod 연결 구조

```
👉 Pod는 ServiceAccount를 통해 RBAC 권한을 가진다
```

🔷 🔥 전체 흐름 (진짜 핵심)

```
Pod 생성
   ↓
ServiceAccount 연결
   ↓
토큰 발급
   ↓
API Server 요청 시 사용
   ↓
RBAC 검사
   ↓
허용 / 거부
```

## 3. ServiceAccount 실제 내부 동작

🔥 Pod 내부

```
/var/run/secrets/kubernetes.io/serviceaccount/
```

👉 포함:

```
token
ca.crt
namespace
```

🔥 API 호출 시

```
Pod → API Server
Authorization: Bearer <token>
```

🔥 API Server 처리

```
1. 토큰 검증
2. 어떤 ServiceAccount인지 확인
3. RBAC 룰 조회
4. 허용 / 거부
```
