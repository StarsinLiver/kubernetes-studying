## 목차

- [1장 Secret 이란?](#1장-secret-이란)
  - [1. Secret 한줄 정리](#1-secret-한줄-정리)
  - [2. 역할 (왜 존재하냐)](#2-역할-왜-존재하냐)
  - [3. 어디서 실행되냐 (구조)](#3-어디서-실행되냐-구조)
  - [4. 실제 Linux 파일 위치](#4-실제-linux-파일-위치)
  - [5. Secret 핵심 개념](#5-secret-핵심-개념)
  - [6. Secret 타입 (중요)](#6-secret-타입-중요)
  - [7. Secret 생성 방법](#7-secret-생성-방법)
  - [8. YAML 옵션 완전 분석](#8-yaml-옵션-완전-분석)
  - [9. Pod에서 사용하는 방식](#9-pod에서-사용하는-방식)
  - [10. etcd에 어떻게 저장되냐](#10-etcd에-어떻게-저장되냐)
  - [11. 실제 내부 저장 구조](#11-실제-내부-저장-구조)
  - [12. 암호화 적용하기 (Encryption at Rest)](#12-암호화-적용하기-encryption-at-rest)
  - [13. etcd 에서 pod 흐름](#13-etcd-에서-pod-흐름)
  - [14. 전체 흐름](#14-전체-흐름)
  - [15. ConfigMap vs Secret 비교](#15-configmap-vs-secret-비교)
  - [16. 실무 핵심 포인트 (중요!)](#16-실무-핵심-포인트-중요)
  - [17. 실제 확인 방법](#17-실제-확인-방법)
  - [18. 요약](#18-요약)
- [2장 Secret 을 안전하게 사용하자](#2장-secret-을-안전하게-사용하자)
  - [0. 전체 그림](#0-전체-그림)
  - [1. etcd encryption (저장 시 암호화)](#1-etcd-encryption-저장-시-암호화)
    - [✅ 한줄 정의](#-한줄-정의)
    - [🔹 어디서 동작하냐](#-어디서-동작하냐)
    - [🔹 내부 흐름](#-내부-흐름)
    - [🔹 설정 방법](#-설정-방법)
    - [🔹 실제 etcd 저장 형태](#-실제-etcd-저장-형태)
  - [2. RBAC (접근 제어)](#2-rbac-접근-제어)
    - [✅ 한줄 정의](#-한줄-정의-1)
    - [🔹 어디서 동작하냐](#-어디서-동작하냐-1)
    - [🔹 내부 흐름](#-내부-흐름-1)
    - [🔹 예제](#-예제)
    - [🔹 핵심 포인트](#-핵심-포인트)
  - [3. HashiCorp Vault (외부 Secret 관리)](#3-hashicorp-vault-외부-secret-관리)
    - [✅ 한줄 정의](#-한줄-정의-2)
    - [🔹 어디서 동작하냐](#-어디서-동작하냐-2)
    - [🔹 구조](#-구조)
    - [🔹 특징](#-특징)
    - [🔹 예시 흐름](#-예시-흐름)
    - [🔹 장단점](#-장단점)
  - [4. External Secrets Operator (ESO)](#4-external-secrets-operator-eso)
    - [✅ 한줄 정의](#-한줄-정의-3)
    - [🔹 어디서 동작하냐](#-어디서-동작하냐-3)
    - [🔹 구조](#-구조-1)
    - [🔹 예제](#-예제-1)
    - [🔹 흐름](#-흐름)
    - [🔹 장단점](#-장단점-1)
  - [5. 4개 비교 (핵심)](#5-4개-비교-핵심)
  - [6. 실무 권장 조합 (중요)](#6-실무-권장-조합-중요)

---

# 1장 Secret 이란?

## 1. Secret 한줄 정리

```
👉 민감한 데이터를 저장해서 Pod에 안전하게 전달하는 Kubernetes 객체
```

## 2. 역할 (왜 존재하냐)

```
ConfigMap은 평문 → 위험
그래서 Secret 등장
```

👉 사용 목적

```
DB 비밀번호
API Key
TLS 인증서
토큰
```

👉 핵심

```
컨테이너 이미지와 민감정보를 분리
```

## 3. 어디서 실행되냐 (구조)

```
Secret은 실행되는 게 아니라 저장되는 리소스
```

```
kubectl apply
   ↓
API Server
   ↓
(etcd 저장)
   ↓
kubelet이 가져감
   ↓
Pod에 주입
```

## 4. 실제 Linux 파일 위치

Pod에 mount하면

```
/var/lib/kubelet/pods/<pod-id>/volumes/kubernetes.io~secret/

👉 여기 파일로 생성됨
```

## 5. Secret 핵심 개념

👉 ConfigMap과 거의 동일한 구조지만

차이는 다음과 같다.

```
민감정보
base64 인코딩
(옵션) 암호화 가능
```

## 6. Secret 타입 (중요)

```
type: Opaque                        # 기본 (일반 key-value)
type: kubernetes.io/tls            # TLS 인증서
type: kubernetes.io/dockerconfigjson
type: kubernetes.io/service-account-token
```

## 7. Secret 생성 방법

YAML

```YAML
apiVersion: v1
kind: Secret
metadata:
  name: my-secret

type: Opaque

data:
  username: YWRtaW4=   # base64(admin)
  password: MTIzNA==   # base64(1234)
```

CLI

```zsh
kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password=1234

👉 내부적으로 자동 base64 처리됨
```

## 8. YAML 옵션 완전 분석

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: string
  namespace: string
  labels:
    key: value
  annotations:
    key: value

type: string

# 여기서는 base64로 미리 변환한 value 를 넣는다. (반드시 base64)
data: # base64 필수
  key: base64value

# 여기서는 자동으로 base64 변환됨
stringData: # 평문 입력 (자동 base64 변환)
  key: value

immutable: false # secret 변경 불가 설정
```

## 9. Pod에서 사용하는 방식

env 방식

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: my-secret
        key: password
```

volume 방식

```yaml
volumeMounts:
  - name: secret-volume
    mountPath: /etc/secret
volumes:
  - name: secret-volume
    secret:
      secretName: my-secret

# 👉 결과:
# Pod 내부 /etc/secret/password 에 파일로 저장
```

## 10. etcd에 어떻게 저장되냐

저장 위치

```
/registry/secrets/<namespace>/<name>
```

예

```
/registry/secrets/default/my-secret
```

cli

```
ETCDCTL_API=3 etcdctl get /registry/secrets/{namespace}/#{secret-name}
```

저장 내용 (개념)

```
{
  "data": {
    "password": "MTIzNA=="
  }
}

👉 base64 상태 그대로 저장됨
```

🔥 중요한 사실

```
👉 base64 = 암호화 ❌
👉 그냥 인코딩 ✔
```

## 11. 실제 내부 저장 구조

```
Secret 객체
   ↓
Go struct
   ↓
Protobuf 직렬화
   ↓
etcd 저장
```

```
👉 etcd 내부:
k8s\x00\x01... (binary)
```

## 12. 암호화 적용하기 (Encryption at Rest)

기본

```
Secret → base64 → etcd
```

암호화 활성화:

```
Secret → base64 → AES 암호화 → etcd
```

설정 yaml 파일

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration

resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: base64key
      - identity: {}
```

## 13. etcd 에서 pod 흐름

```
etcd
  ↓
복호화 (optional)
  ↓
base64 decode
  ↓
Pod 전달
```

```
👉 Pod에서는 평문 사용됨
```

## 14. 전체 흐름

```
1. Secret 생성
2. API Server 전달
3. (옵션) 암호화
4. etcd 저장
5. kubelet이 가져옴
6. Pod에 주입 (env or file)
7. 애플리케이션 사용
```

## 15. ConfigMap vs Secret 비교

| 항목      | ConfigMap | Secret   |
| --------- | --------- | -------- |
| 목적      | 설정      | 민감정보 |
| etcd 저장 | 평문      | base64   |
| 암호화    | 없음      | 옵션     |
| 보안 수준 | 낮음      | 중간     |

## 16. 실무 핵심 포인트 (중요!)

⚠️ 1. Secret도 기본은 안전하지 않음

```
→ base64라서 쉽게 복호화 가능
```

⚠️ 2. 반드시 해야 할 것

```
etcd encryption 활성화
RBAC 제한
```

⚠️ 3. 더 강력한 방식

```
HashiCorp Vault
External Secrets Operator
```

👉 진짜 보안은 다음과 같다.

```
etcd 암호화
접근 제어
```

## 17. 실제 확인 방법

```
kubectl get secret my-secret -o yaml
```

디코딩

```
kubectl get secret my-secret -o jsonpath="{.data.password}" | base64 -d
```

## 18. 요약

```
Secret 생성
   ↓
API Server
   ↓
(base64)
   ↓
(옵션 AES 암호화)
   ↓
etcd 저장
   ↓
kubelet
   ↓
Pod (평문 사용)
```

---

# 2장 Secret 을 안전하게 사용하자

## 0. 전체 그림

```
[사용자 / Pod]
      ↓
[API Server]
      ↓
(암호화 여부)
      ↓
[etcd 저장]
      ↑
RBAC로 접근 제어

+ 외부:
Vault / External Secrets Operator
```

👉 핵심

```
저장 보안 → etcd encryption
접근 보안 → RBAC
외부 보안 → Vault / ESO
```

## 1. etcd encryption (저장 시 암호화)

### ✅ 한줄 정의

```
👉 Secret을 etcd에 암호화해서 저장하는 기능
```

### 🔹 어디서 동작하냐

```
kube-apiserver에서 동작
etcd에 쓰기 직전에 암호화
```

### 🔹 내부 흐름

```
Secret 생성
   ↓
API Server
   ↓
AES 암호화
   ↓
etcd 저장
```

읽을 때

```
etcd
   ↓
복호화
   ↓
API Server
   ↓
Pod 전달
```

### 🔹 설정 방법

1️⃣ encryption-config.yaml

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration

resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: c2VjcmV0a2V5MTIzNDU2
      - identity: {}
```

2️⃣ kube-apiserver 옵션 추가

```
--encryption-provider-config=/etc/kubernetes/encryption-config.yaml
```

### 🔹 실제 etcd 저장 형태

❌ 기본:

```
password: MTIzNA==
```

⭕ 암호화:

```
k8s:enc:aescbc:v1:key1:...
```

🔹 핵심 포인트

```
기본은 암호화 안됨
반드시 운영 환경에서는 켜야 함
기존 Secret은 재암호화 필요
```

## 2. RBAC (접근 제어)

### ✅ 한줄 정의

```
👉 누가 Secret을 읽을 수 있는지 제한하는 권한 시스템
```

### 🔹 어디서 동작하냐

```
API Server에서 요청 받을 때
```

### 🔹 내부 흐름

```
kubectl get secret
   ↓
API Server
   ↓
인증(Authentication)
   ↓
인가(Authorization, RBAC)
   ↓
허용/거부
```

### 🔹 예제

Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: secret-reader

rules:
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get"]
```

RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-secrets
  namespace: default

subjects:
  - kind: User
    name: developer

roleRef:
  kind: Role
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

### 🔹 핵심 포인트

```
RBAC 없으면 누구나 Secret 조회 가능
최소 권한 원칙 적용해야 함
```

## 3. HashiCorp Vault (외부 Secret 관리)

### ✅ 한줄 정의

```
👉 Secret을 Kubernetes 밖에서 안전하게 관리하는 시스템
```

### 🔹 어디서 동작하냐

```
외부 시스템 (Vault 서버)
```

### 🔹 구조

```
Pod
  ↓
Vault Agent / API
  ↓
Vault Server
  ↓
Secret 저장
```

### 🔹 특징

```
Secret을 etcd에 저장하지 않음
동적 Secret 생성 가능 (DB 계정 등)
TTL (자동 만료)
```

### 🔹 예시 흐름

```
Pod 시작
   ↓
Vault에서 Secret 요청
   ↓
임시 비밀번호 발급
   ↓
Pod 사용
   ↓
시간 지나면 자동 폐기
```

### 🔹 장단점

장점

```
보안 최고 수준
Secret rotation 자동화
audit 가능
```

단점

```
구축 복잡
운영 비용 증가
```

## 4. External Secrets Operator (ESO)

### ✅ 한줄 정의

```
👉 외부 Secret(Vault, AWS 등)을 Kubernetes Secret으로 자동 동기화하는 도구
```

### 🔹 어디서 동작하냐

Kubernetes 내부 Controller

### 🔹 구조

```
External Secret (CRD)
      ↓
ESO Controller
      ↓
Vault / AWS Secret Manager
      ↓
Kubernetes Secret 생성
```

### 🔹 예제

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: my-secret

spec:
  refreshInterval: 1h

  secretStoreRef:
    name: vault-backend
    kind: SecretStore

  target:
    name: k8s-secret

  data:
    - secretKey: password
      remoteRef:
        key: db/password
```

### 🔹 흐름

```
ESO
  ↓
외부 Secret 조회
  ↓
Kubernetes Secret 생성
  ↓
Pod에서 사용
```

### 🔹 장단점

장점

```
Kubernetes 방식 그대로 사용 가능
외부 Secret 자동 동기화
Vault와 잘 맞음
```

단점

```
결국 Secret이 etcd에 존재함
완전 외부 관리보다 보안 낮음
```

## 5. 4개 비교 (핵심)

| 항목      | etcd encryption | RBAC       | Vault     | ESO         |
| --------- | --------------- | ---------- | --------- | ----------- |
| 목적      | 저장 보호       | 접근 제한  | 외부 보안 | 자동 동기화 |
| 위치      | API Server      | API Server | 외부      | Kubernetes  |
| etcd 저장 | 암호화          | 그대로     | 없음      | 있음        |
| 난이도    | 중간            | 낮음       | 높음      | 중간        |

## 6. 실무 권장 조합 (중요)

🔹 기본 (필수)

```
etcd encryption + RBAC
```

🔹 보안 강화

```
Vault + ESO
```

🔹 최고 수준

```
Vault (direct) + no Secret in etcd (Do not use ESO)
```
