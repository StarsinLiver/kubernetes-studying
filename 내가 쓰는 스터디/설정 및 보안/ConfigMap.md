## 목차

- [1장 ConfigMap 이란](#1장-configmap-이란)
  - [1. ConfigMap 한줄 정의](#1-configmap-한줄-정의)
  - [2. ConfigMap의 역할 (왜 존재하냐)](#2-configmap의-역할-왜-존재하냐)
  - [3. 어디서 실행되냐 (구조 관점)](#3-어디서-실행되냐-구조-관점)
  - [4. 실제 Linux 파일 어디에 있냐](#4-실제-linux-파일-어디에-있냐)
  - [5. ConfigMap 확인 방법](#5-configmap-확인-방법)
  - [6. 전체 구조](#6-전체-구조)
  - [7. 중요한 흐름](#7-중요한-흐름)
  - [8. ConfigMap 만드는 방법](#8-configmap-만드는-방법)
    - [방법 1: YAML](#방법-1-yaml)
    - [방법 2: CLI](#방법-2-cli)
    - [방법 3: 파일 기반](#방법-3-파일-기반)
  - [9. YAML 옵션 완전 분석](#9-yaml-옵션-완전-분석)
  - [10. Pod에서 사용하는 YAML](#10-pod에서-사용하는-yaml)
    - [1️⃣ 환경변수로 주입](#1️⃣-환경변수로-주입)
    - [2️⃣ 파일로 mount (Volume mount) (가장 많이 씀)](#2️⃣-파일로-mount-volume-mount-가장-많이-씀)
    - [3️⃣ 전체 env 가져오기](#3️⃣-전체-env-가져오기)
  - [12. 실제 내부 동작](#12-실제-내부-동작)
  - [13. 실무에서 중요한 포인트](#13-실무에서-중요한-포인트)
  - [14. Secret 과의 차이](#14-secret-과의-차이)
- [2장 어디에서 실행되고 어디에 저장되느냐](#2장-어디에서-실행되고-어디에-저장되느냐)
  - [1. 한줄 핵심](#1-한줄-핵심)
  - [2. etcd에 저장되는 위치 (Key 구조)](#2-etcd에-저장되는-위치-key-구조)
  - [3. 실제 저장 데이터 구조](#3-실제-저장-데이터-구조)
  - [4. 실제 내부는 JSON이냐?](#4-실제-내부는-json이냐)
  - [5. 실제 etcd에서 조회해보기](#5-실제-etcd에서-조회해보기)
  - [6. 왜 protobuf로 저장하냐](#6-왜-protobuf로-저장하냐)
  - [7. 저장 흐름](#7-저장-흐름)
  - [8. 읽을 때 흐름](#8-읽을-때-흐름)
  - [9. 실제 파일로 변환되는 모습](#9-실제-파일로-변환되는-모습)
- [3장 Protobuf 란?](#3장-protobuf-란)
  - [1. Protobuf 한줄 정의](#1-protobuf-한줄-정의)
  - [2. 직렬화가 뭐냐 (핵심 개념)](#2-직렬화가-뭐냐-핵심-개념)
  - [3. JSON vs Protobuf 차이](#3-json-vs-protobuf-차이)
  - [4. Protobuf 구조](#4-protobuf-구조)
  - [5. 실제 저장 방식 (핵심 이해)](#5-실제-저장-방식-핵심-이해)
  - [6. Kubernetes에서 Protobuf 역할](#6-kubernetes에서-protobuf-역할)
  - [7. etcd 저장 구조와 연결](#7-etcd-저장-구조와-연결)
  - [8. 읽을 때 동작](#8-읽을-때-동작)
  - [9. 왜 Kubernetes가 Protobuf를 쓰냐](#9-왜-kubernetes가-protobuf를-쓰냐)
    - [1️⃣ 성능](#1️⃣-성능)
    - [2️⃣ 저장 효율](#2️⃣-저장-효율)
    - [3️⃣ 네트워크 비용 감소](#3️⃣-네트워크-비용-감소)

---

# 1장 ConfigMap 이란

## 1. ConfigMap 한줄 정의

```
👉 컨테이너와 분리된 설정값(key-value)을 저장해서 Pod에 주입하는 객체
```

## 2. ConfigMap의 역할 (왜 존재하냐)

```
컨테이너 이미지는 불변(immutable)이어야 함
→ 그런데 설정(DB 주소, 포트, 환경별 설정 등)은 계속 바뀜

그래서:

코드/이미지 ≠ 설정 분리
같은 이미지로 dev / staging / prod 환경 구성 가능
```

👉 핵심 목적:

```
환경별 설정 분리
재배포 없이 설정 변경
```

언제 쓰냐

```
DB 주소
Redis 주소
feature flag
config 파일 (application.properties)
nginx 설정
```

## 3. 어디서 실행되냐 (구조 관점)

ConfigMap 은

```
실행되는 게 아니라 저장되는 리소스
```

위치

```
Kubernetes API Server에 등록됨
내부적으로는 etcd에 저장됨
```

흐름

```
kubectl apply
  ↓
API Server
  ↓
etcd 저장
  ↓
Pod가 요청해서 가져감
```

## 4. 실제 Linux 파일 어디에 있냐

중요 포인트

```
👉 ConfigMap 자체는 노드의 파일이 아님 (etcd에 있음)
```

하지만 Pod에 mount 한다면

```
/var/lib/kubelet/pods/<pod-id>/volumes/kubernetes.io~configmap/

여기 실제 파일로 생성됨
```

확인 방법

```
# Pod ID 확인
kubectl get pod -o wide

# 노드 접속 후
cd /var/lib/kubelet/pods/

# 해당 Pod 디렉토리 들어가서 확인
```

👉 하지만 보통은 직접 안 들어감 (kubectl로 확인함)

## 5. ConfigMap 확인 방법

```
kubectl get configmap
```

## 6. 전체 구조

```
ConfigMap
  ├── data (문자열)
  ├── binaryData (바이너리)
  └── metadata

Pod
  ├── env
  ├── envFrom
  └── volume mount
```

## 7. 중요한 흐름

```
1. ConfigMap 생성
2. Pod에서 참조
3. kubelet이 ConfigMap 가져옴
4. 컨테이너에 주입
   - env OR 파일
5. 애플리케이션이 사용
```

🔥 핵심

```
Pod 생성 시점에 값이 들어감
env 방식은 변경 반영 안됨 (재시작 필요)
volume 방식은 자동 반영됨 (딜레이 있음)
```

## 8. ConfigMap 만드는 방법

### 방법 1: YAML

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
  namespace: default

data:
  db_host: "mysql-service"
  db_port: "3306"
  app_mode: "production"
```

### 방법 2: CLI

```zsh
kubectl create configmap my-config \
--from-literal=db_host=mysql \
--from-literal=db_port=3306
```

### 방법 3: 파일 기반

```zsh
kubectl create configmap my-config --from-file=app.properties
```

## 9. YAML 옵션 완전 분석

기본 구조

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: string # 필수 (리소스 이름)
  namespace: string # 선택 (기본 default)
  labels: # 선택 (분류용)
    key: value
  annotations: # 선택 (설명/메타)
    key: value

# data는 문자열만 가능
# key 는 파일명, value 는 파일 내용
data: # 핵심 (string key-value)
  key1: value1
  key2: value2
  key3: |
    this is 
    value3

# 이미지, 인증서 같은 바이너리
# base64 필요
binaryData: # 선택 (base64 인코딩)
  file.bin: base64string

immutable: false # true면 수정 불가 (성능/안정성)
```

## 10. Pod에서 사용하는 YAML

ConfigMap 사용 방식 3가지가 있다.

### 1️⃣ 환경변수로 주입

```yaml
env:
  - name: DB_HOST
    valueFrom:
      configMapKeyRef:
        name: my-config
        key: db_host
```

```yaml
# Pod 내부에서
apiVersion: v1
kind: Pod
metadata:
  name: configmap-env-pod

spec:
  containers:
    - name: app
      image: nginx

      env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: my-config
              key: db_host
```

### 2️⃣ 파일로 mount (Volume mount) (가장 많이 씀)

```yaml
volumes:
  - name: config-volume
    configMap:
      name: my-config
```

```yaml
# Pod 내부에서
# 결과
# /etc/config/db_host
# /etc/config/db_port

apiVersion: v1
kind: Pod
metadata:
  name: configmap-volume-pod

spec:
  containers:
    - name: app
      image: nginx

      volumeMounts:
        - name: config-volume
          mountPath: /etc/config

  volumes:
    - name: config-volume
      configMap:
        name: my-config
```

### 3️⃣ 전체 env 가져오기

```yaml
envFrom:
  - configMapRef:
      name: my-config
```

## 12. 실제 내부 동작

kubelet이 하는 일

```
1. API Server에서 ConfigMap 조회
2. 로컬 캐시에 저장
3. 파일로 변환
4. 컨테이너에 mount

👉 그래서:
빠르다는 장점이 있다.
노드마다 캐싱됨
```

## 13. 실무에서 중요한 포인트

⚠️ 1. ConfigMap은 암호용 아님

```
→ 비밀번호는 Secret 사용
```

⚠️ 2. env는 변경 반영 안됨

```
→ 재배포 필요
```

⚠️ 3. volume은 자동 반영됨

```
→ 약간의 delay 존재
```

⚠️ 4. immutable 추천

```
→ 대규모 환경에서 성능 개선
```

## 14. Secret 과의 차이

ConfigMap vs Secret 차이

| 구분      | ConfigMap | Secret                      |
| --------- | --------- | --------------------------- |
| 목적      | 일반 설정 | 민감 정보                   |
| 데이터    | 평문      | base64 인코딩               |
| etcd 저장 | 평문      | 기본은 base64 (암호화 아님) |
| 암호화    | ❌        | ⭕ (옵션으로 가능)          |
| 예시      | DB host   | DB password                 |

---

# 2장 어디에서 실행되고 어디에 저장되느냐

## 1. 한줄 핵심

```
👉 ConfigMap은 etcd에 "그대로 JSON 형태(평문)"로 저장된다
```

## 2. etcd에 저장되는 위치 (Key 구조)

etcd는 key-value DB라서 이런 형태로 저장됨:

```
/registry/configmaps/<namespace>/<name>
```

예시:

```
/registry/configmaps/default/my-config

👉 여기 value에 실제 ConfigMap 내용이 들어있다
```

## 3. 실제 저장 데이터 구조

예제 ConfigMap YAML

```yaml
# 예시문
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
  namespace: default

data:
  db_host: mysql
  db_port: "3306"
```

etcd에 저장될 때 (개념적으로)

```json
{
  "kind": "ConfigMap",
  "apiVersion": "v1",
  "metadata": {
    "name": "my-config",
    "namespace": "default",
    "uid": "xxxx-xxxx",
    "resourceVersion": "12345",
    "creationTimestamp": "2026-04-19T00:00:00Z"
  },
  "data": {
    "db_host": "mysql",
    "db_port": "3306"
  }
}
```

핵심 포인트

```
👉 data 값이 그대로 평문으로 저장됨
👉 아무런 인코딩 / 암호화 없음
```

## 4. 실제 내부는 JSON이냐?

엄밀히 말하면

```
👉 내부는 JSON이 아니라 protobuf(binary)로 저장됨
```

하지만

```
etcdctl로 보면 JSON처럼 보이거나
decode하면 JSON 형태
```

즉

```
Kubernetes Object
   ↓
(내부 변환)
   ↓
protobuf
   ↓
etcd 저장
```

## 5. 실제 etcd에서 조회해보기

관리자 권한 필요

```zsh
ETCDCTL_API=3 etcdctl get /registry/configmaps/default/my-config
```

출력 예시 (압축된 binary)

```
k8s... (binary data)
```

사람이 읽으려면

```
ETCDCTL_API=3 etcdctl get /registry/configmaps/default/my-config --print-value-only

--print-value-only
또는:
--write-out=json
```

## 6. 왜 protobuf로 저장하냐

이유

```
성능 (빠름)
저장 효율
네트워크 효율

👉 JSON보다 훨씬 가볍고 빠름
```

## 7. 저장 흐름

```
kubectl apply
   ↓
API Server (REST 요청)
   ↓
Object 생성 (Go struct)
   ↓
protobuf 직렬화
   ↓
etcd 저장 (/registry/configmaps/...)
```

## 8. 읽을 때 흐름

```
etcd
  ↓
protobuf → Go object 변환
  ↓
API Server
  ↓
kubectl / kubelet / controller
```

## 9. 실제 파일로 변환되는 모습

ConfigMap

```
data:
  db_host: mysql
  db_port: "3306"
```

Pod에 mount되면

```
/etc/config/db_host   → mysql
/etc/config/db_port   → 3306

즉,
👉 key = 파일명
👉 value = 파일 내용
```

---

# 3장 Protobuf 란?

## 1. Protobuf 한줄 정의

```
👉 데이터를 빠르고 작게 직렬화(serialize)하기 위한 바이너리 포맷
```

## 2. 직렬화가 뭐냐 (핵심 개념)

```
👉 “객체 → 저장/전송 가능한 형태로 변환”
```

예

```
{ "name": "kim", "age": 30 }

이걸 파일로 저장 또는 네트워크로 전송하려면 변환해야 함
```

## 3. JSON vs Protobuf 차이

| 구분        | JSON   | Protobuf |
| ----------- | ------ | -------- |
| 형태        | 텍스트 | 바이너리 |
| 사람이 읽기 | 가능   | 불가능   |
| 크기        | 큼     | 작음     |
| 속도        | 느림   | 빠름     |
| 스키마      | 없음   | 있음     |

```
🔥핵심 차이

👉 JSON = 사람이 보기 좋음
👉 Protobuf = 컴퓨터가 처리하기 좋음
```

## 4. Protobuf 구조

```
Protobuf는 먼저 스키마(.proto)를 정의함
```

예:

```
syntax = "proto3";

message User {
  string name = 1;
  int32 age = 2;
}
```

```
👉 여기서 중요한 것
name = 1
age = 2

👉 번호(tag) 기반으로 저장됨
```

## 5. 실제 저장 방식 (핵심 이해)

JSON

```
{
  "name": "kim",
  "age": 30
}
```

Protobuf (개념적)

```
1: "kim"
2: 30
```

```
👉 key 이름이 없음
👉 숫자로 매핑됨

👉 그래서
용량 ↓
속도 ↑
```

왜 빠르냐

```
문자열 키 없음
바이너리 포맷
파싱 비용 낮음
```

## 6. Kubernetes에서 Protobuf 역할

쿠버네티스 내부 흐름

```
kubectl apply
   ↓
API Server
   ↓
Go struct (Object)
   ↓
Protobuf 직렬화
   ↓
etcd 저장
```

즉

```
우리가 보는 YAML/JSON ❌
내부는 전부 Protobuf ✔
```

## 7. etcd 저장 구조와 연결

ConfigMap 저장

```
/registry/configmaps/default/my-config
   ↓
value = protobuf binary
```

👉 그래서 etcd에서 보면

```
etcdctl get ...

👉 결과:
k8s\x00\x01... (이상한 바이너리)
```

## 8. 읽을 때 동작

```
etcd (protobuf)
   ↓
API Server
   ↓
protobuf → Go object
   ↓
JSON/YAML 변환
   ↓
kubectl 출력

👉 우리가 보는 YAML은 변환된 결과
```

## 9. 왜 Kubernetes가 Protobuf를 쓰냐

이유 3개:

### 1️⃣ 성능

```
API 요청 많음
빠른 직렬화 필요
```

### 2️⃣ 저장 효율

```
etcd 용량 절약
```

### 3️⃣ 네트워크 비용 감소

```
컨트롤 플레인 통신 최적화
```
