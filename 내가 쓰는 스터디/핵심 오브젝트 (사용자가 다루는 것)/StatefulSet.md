## 목차

- [1장 StatefulSet](#1장-statefulset)
  - [1️⃣ StatefulSet이 무엇인가](#1️⃣-statefulset이-무엇인가)
  - [2️⃣ StatefulSet 핵심 특징](#2️⃣-statefulset-핵심-특징)
  - [3️⃣ Deployment vs StatefulSet](#3️⃣-deployment-vs-statefulset)
  - [4️⃣ StatefulSet YAML 구조](#4️⃣-statefulset-yaml-구조)
  - [5️⃣ StatefulSet에서 가장 중요한 것](#5️⃣-statefulset에서-가장-중요한-것)
    - [serviceName](#servicename)
  - [6️⃣ StatefulSet Pod 이름 생성](#6️⃣-statefulset-pod-이름-생성)
  - [7️⃣ StatefulSet Controller 내부 동작](#7️⃣-statefulset-controller-내부-동작)
  - [8️⃣ syncStatefulSet() 알고리즘](#8️⃣-syncstatefulset-알고리즘)
  - [9️⃣ Pod 생성 순서](#9️⃣-pod-생성-순서)
  - [🔟 Pod 삭제 순서](#-pod-삭제-순서)
  - [11. 스토리지 생성 (volumeClaimTemplates)](#11-스토리지-생성-volumeclaimtemplates)
  - [12. Rolling Update](#12-rolling-update)
  - [13. StatefulSet object 저장 위치](#13-statefulset-object-저장-위치)
  - [14. 실제 Kubernetes 구조](#14-실제-kubernetes-구조)
  - [15. 실제 확인 명령어](#15-실제-확인-명령어)
  - [🔥 StatefulSet에서 진짜 중요한 개념](#-statefulset에서-진짜-중요한-개념)
- [2장 더 나아가서](#2장-더-나아가서)
  - [Stable Network Identity](#stable-network-identity)
    - [1️⃣ Stable Network Identity](#1️⃣-stable-network-identity)
    - [2️⃣ StatefulSet Pod 이름 규칙](#2️⃣-statefulset-pod-이름-규칙)
    - [3️⃣ 그런데 이름만 있으면 부족하다](#3️⃣-그런데-이름만-있으면-부족하다)
    - [4️⃣ Headless Service 등장](#4️⃣-headless-service-등장)
    - [5️⃣ Headless Service의 역할](#5️⃣-headless-service의-역할)
    - [6️⃣ StatefulSet DNS 구조](#6️⃣-statefulset-dns-구조)
    - [7️⃣ DNS는 누가 만드는가](#7️⃣-dns는-누가-만드는가)
    - [8️⃣ 실제 DNS 조회 예시](#8️⃣-실제-dns-조회-예시)
  - [Stable Storage](#stable-storage)
    - [1️⃣ Stable Storage](#1️⃣-stable-storage)
    - [2️⃣ volumeClaimTemplates](#2️⃣-volumeclaimtemplates)
    - [3️⃣ PVC란 무엇인가](#3️⃣-pvc란-무엇인가)
      - [PVC 역할](#pvc-역할)
    - [4️⃣ StatefulSet PVC 특징](#4️⃣-statefulset-pvc-특징)
    - [5️⃣ 실제 생성 흐름](#5️⃣-실제-생성-흐름)
    - [6️⃣ 실제 볼륨 mount](#6️⃣-실제-볼륨-mount)
    - [7️⃣ StatefulSet 핵심 특징](#7️⃣-statefulset-핵심-특징)

---

# 1장 StatefulSet

## 1️⃣ StatefulSet이 무엇인가

StatefulSet은 상태(state)를 가지는 애플리케이션을 위해 만들어진 컨트롤러다.

예

```
Database
Kafka
Redis cluster
ZooKeeper
Elasticsearch
```

이런 시스템은 Pod가 서로 구분되어야 한다.
<br> 하지만 Deployment는 밑과 같이 이런 랜덤 이름을 만든다.

[Deployment]

```
pod-abc123
pod-x4f9s
pod-zz91a
```

[StatefulSet] 은

```
pod-0
pod-1
pod-2
```

처럼 고정된 identity를 가진다.

## 2️⃣ StatefulSet 핵심 특징

StatefulSet은 3가지가 핵심이다.

```
1️⃣ Stable Pod Name
2️⃣ Stable Network Identity
3️⃣ Stable Storage
```

예

```
mysql-0
mysql-1
mysql-2

삭제 후 재생성해도 mysql-0 그대로 유지된다.
```

## 3️⃣ Deployment vs StatefulSet

| 특징          | Deployment | StatefulSet |
| ------------- | ---------- | ----------- |
| Pod 이름      | 랜덤       | 고정        |
| Pod 생성 순서 | 무작위     | 순차        |
| Pod 삭제 순서 | 무작위     | 역순        |
| 스토리지      | 공유 가능  | Pod별 고정  |
| 사용 사례     | stateless  | stateful    |

## 4️⃣ StatefulSet YAML 구조

```yaml
apiVersion: apps/v1
kind: StatefulSet

metadata:
  name: web

spec:
  serviceName: web

  replicas: 3

  selector:
    matchLabels:
      app: web

  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx

  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi
```

## 5️⃣ StatefulSet에서 가장 중요한 것

### serviceName

```
serviceName: web

이것은 Headless Service와 연결된다.
```

예

```
kind: Service
clusterIP: None
```

DNS

```
web-0.web.default.svc.cluster.local
web-1.web.default.svc.cluster.local
web-2.web.default.svc.cluster.local
```

## 6️⃣ StatefulSet Pod 이름 생성

[Deployment]

```
nginx-7d9fbcdf4-abc12
```

[StatefulSet]

```
nginx-0
nginx-1
nginx-2

이는 StatefulSet Controller가 ordinal index 를 사용한다.
```

## 7️⃣ StatefulSet Controller 내부 동작

Controller 위치

```
kube-controller-manager
```

내부 구조

````
StatefulSet Controller
   │
   ├── Informer
   │
   ├── WorkQueue
   │
   └── Worker
        ↓
     syncStatefulSet()
     ```
````

## 8️⃣ syncStatefulSet() 알고리즘

핵심 로직은 다음과 같다.

```go
syncStatefulSet(set):

   pods = getPods(set)

   for i in range(replicas):

       if pod[i] not exist:
           createPod(i)
```

즉

```
ordinal 기반 생성
```

## 9️⃣ Pod 생성 순서

StatefulSet은 순서가 중요하다.

예

```
replicas = 3
```

생성 순서

```
web-0
web-1
web-2
```

Controller는 web-0 Running 확인 후에 web-1 생성

## 🔟 Pod 삭제 순서

삭제는 역순이다.

```
web-2
web-1
web-0
```

왜냐하면 cluster consistency 때문이다.

## 11. 스토리지 생성 (volumeClaimTemplates)

StatefulSet은 volumeClaimTemplates를 사용한다.

예

```yaml
volumeClaimTemplates:
  - metadata:
      name: data
```

Pod 생성 시 PVC 생성

```
data-web-0
data-web-1
data-web-2
```

각 Pod는 자기 디스크를 가진다.

## 12. Rolling Update

Deployment

```
병렬 업데이트
```

StatefulSet

```
순차 업데이트
```

예

```
web-2 업데이트
web-1 업데이트
web-0 업데이트
```

또는 podManagementPolicy 옵션으로 변경 가능.

## 13. StatefulSet object 저장 위치

StatefulSet도 etcd 에 저장된다.

key

```
/registry/statefulsets/default/web
```

확인

```
➜  etcdctl get /registry/statefulsets/default/web
```

## 14. 실제 Kubernetes 구조

전체 구조

```
StatefulSet
   ↓
Pod
   ↓
PVC
   ↓
Storage
```

## 15. 실제 확인 명령어

StatefulSet 확인

```
kubectl get statefulset
```

Pod 확인

```
kubectl get pods
```

PVC 확인

```
kubectl get pvc
```

DNS 확인

```
kubectl exec -it web-0 -- hostname
```

## 🔥 StatefulSet에서 진짜 중요한 개념

3개만 기억하면 된다.

```
Stable Identity
Ordered Deployment
Stable Storage
```

# 2장 더 나아가서

## Stable Network Identity

### 1️⃣ Stable Network Identity

Deployment Pod 이름

```
nginx-7c8d9f6c7d-x2abc
nginx-7c8d9f6c7d-k9lmn
```

```
문제

Pod 죽음
→ 새 Pod 생성
→ 이름 변경
→ IP 변경

DB나 클러스터 시스템에서는 큰 문제다.

[예]
MySQL cluster
Kafka
Zookeeper
Redis cluster

이런 시스템은 각 노드가 고정 이름을 가져야 한다.
```

그래서 StatefulSet은 이렇게 만든다.

```
mysql-0
mysql-1
mysql-2
```

따라서 Pod이 죽어도 mysql-0이 다시 생성된다.

### 2️⃣ StatefulSet Pod 이름 규칙

Pod 이름

```
<statefulset-name>-<ordinal>
```

예

```
mysql-0
mysql-1
mysql-2
```

이 숫자를 ordinal index 라고 한다.

### 3️⃣ 그런데 이름만 있으면 부족하다

Pod은 IP로 통신한다.

문제

```
mysql-0 IP = 10.244.1.10
```

Pod 재생성

```
mysql-0 IP = 10.244.3.22
```

IP가 바뀐다. 그래서 필요한 것이 DNS이다.

### 4️⃣ Headless Service 등장

일반 Service

```yaml
spec:
  type: ClusterIP
```

이 경우 DNS

```
mysql.default.svc.cluster.local

→ Service IP 반환
```

하지만 StatefulSet은 Pod IP가 필요하다.

그래서 다음과 같이 구성한다.

```
clusterIP: None
```

Headless Service 정의

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
    - port: 3306
```

이걸 Headless Service 라고 한다.

### 5️⃣ Headless Service의 역할

일반 Service

```
DNS → Service IP
```

Headless Service

```
DNS → Pod IP
```

### 6️⃣ StatefulSet DNS 구조

Pod DNS 이름

```
<pod-name>.<service-name>.<namespace>.svc.cluster.local
```

예

```
mysql-0.mysql.default.svc.cluster.local
mysql-1.mysql.default.svc.cluster.local
mysql-2.mysql.default.svc.cluster.local
```

DNS 조회

```
➜  nslookup mysql-0.mysql

[결과]
10.244.1.5
```

### 7️⃣ DNS는 누가 만드는가

핵심 구성요소

```
CoreDNS
```

흐름

```
Pod 생성
   ↓
API Server에 등록
   ↓
CoreDNS가 watch
   ↓
DNS 레코드 생성
```

DNS record 형태

```
A record
```

예

```
mysql-0.mysql.default.svc.cluster.local
→ 10.244.1.5
```

### 8️⃣ 실제 DNS 조회 예시

Pod 내부

```
nslookup mysql
```

결과

```
mysql-0.mysql.default.svc.cluster.local
mysql-1.mysql.default.svc.cluster.local
mysql-2.mysql.default.svc.cluster.local
```

즉, Headless Service는 Service → Pod list 반환이다.

## Stable Storage

### 1️⃣ Stable Storage

이제 두 번째 핵심이다.

Deployment 문제는 Pod 재생성이다.

```
Pod 삭제
→ 새 Pod 생성
→ 데이터 사라짐
```

DB에서는 치명적이다.

```
그래서 StatefulSet은 Pod마다 고정 Storage를 만든다.
```

### 2️⃣ volumeClaimTemplates

StatefulSet YAML

```yaml
volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

이게 의미하는 것

```
Pod마다 PVC 생성
```

예

```
[StatefulSet]
replicas: 3

[생성 결과]
data-mysql-0
data-mysql-1
data-mysql-2

PVC 3개 생성된다.
```

### 3️⃣ PVC란 무엇인가

구조

```
Pod
 ↓
PVC (PersistentVolumeClaim)
 ↓
PV (PersistentVolume)
 ↓
Storage
```

#### PVC 역할

```
PVC는 "나는 10GB storage 필요해"라는 요청 객체다.
이를 Kubernetes는 PV를 찾아 연결한다.
```

예

```
resources:
  requests:
    storage: 10Gi
```

### 4️⃣ StatefulSet PVC 특징

```
[StatefulSet Pod 삭제]
mysql-0 삭제

[다시 생성]
mysql-0 생성

[PVC]
data-mysql-0

즉, 같은 PVC 재사용(데이터 유지)
```

### 5️⃣ 실제 생성 흐름

StatefulSet 생성

```
StatefulSet Controller
      ↓
Pod 생성
      ↓
PVC 생성
      ↓
Storage Provisioner
      ↓
PV 생성
      ↓
Pod mount
```

### 6️⃣ 실제 볼륨 mount

Pod 내부

```
volumeMounts:
- name: data
  mountPath: /var/lib/mysql
```

```
실제 /var/lib/mysql 에 디스크가 연결된다.
```

### 7️⃣ StatefulSet 핵심 특징

| 특징            | 이유              |
| --------------- | ----------------- |
| stable pod name | cluster node 식별 |
| stable DNS      | node 연결         |
| stable storage  | 데이터 유지       |
