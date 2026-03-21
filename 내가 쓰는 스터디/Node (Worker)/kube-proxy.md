## 목차

- [1장 kube-proxy란?](#1장-kube-proxy란)
  - [2️⃣ 왜 kube-proxy가 필요한가](#2️⃣-왜-kube-proxy가-필요한가)
  - [3️⃣ kube-proxy가 하는 일](#3️⃣-kube-proxy가-하는-일)
  - [4️⃣ kube-proxy 동작 방식](#4️⃣-kube-proxy-동작-방식)
  - [5️⃣ kube-proxy 동작 모드](#5️⃣-kube-proxy-동작-모드)
    - [① userspace (옛날 방식)](#-userspace-옛날-방식)
    - [② iptables 모드 (가장 많이 사용)](#-iptables-모드-가장-많이-사용)
    - [③ IPVS 모드 (고성능)](#-ipvs-모드-고성능)
  - [6️⃣ 실제 iptables 규칙](#6️⃣-실제-iptables-규칙)
  - [7️⃣ kube-proxy는 패킷을 직접 보내지 않는다](#7️⃣-kube-proxy는-패킷을-직접-보내지-않는다)
  - [8️⃣ kube-proxy 위치](#8️⃣-kube-proxy-위치)
  - [9️⃣ Service 트래픽 실제 흐름](#9️⃣-service-트래픽-실제-흐름)
  - [🔟 핵심 정리](#-핵심-정리)

---

# 1장 kube-proxy란?

kube-proxy는 각 노드(Node)에서 실행되는 Kubernetes 컴포넌트로,
<br>👉 Service IP로 들어온 트래픽을 실제 Pod로 전달하도록 네트워크 규칙을 만드는 역할을 한다.

즉 한 줄로 말하면:

Service → Pod 로 트래픽을 보내는 라우팅 규칙을 만드는 프로그램

### 2️⃣ 왜 kube-proxy가 필요한가

```
Kubernetes에서 Pod는 계속 바뀐다.

예:
Pod A 10.244.1.3
Pod B 10.244.2.5
Pod C 10.244.3.7

이 Pod들이 하나의 Service에 묶이면:
Service IP = 10.96.0.10

사용자는 이렇게 접속한다.
curl 10.96.0.10

하지만 실제로는
10.96.0.10 → Pod A
→ Pod B
→ Pod C

여러 Pod로 로드밸런싱되어야 한다.

문제:
Service IP는 실제로 존재하는 네트워크 인터페이스가 아니다.
그래서 누군가가 트래픽을 Pod로 보내줘야 한다.
그게 kube-proxy다.
```

### 3️⃣ kube-proxy가 하는 일

```
kube-proxy는 kube-apiserver를 계속 감시한다.

watch Service
watch Endpoint
watch EndpointSlice

예를 들어 Service가 생성되면:
kubectl create service

API 서버에 정보가 저장된다.
kube-proxy는 이걸 보고

iptables rules
또는
IPVS rules

를 만든다.

대표적으로 사용하는 것이 iptables이다.
```

### 4️⃣ kube-proxy 동작 방식

```
트래픽 흐름:
Client
   ↓
Service IP (ClusterIP)
   ↓
iptables rule
   ↓
Pod IP

예시
10.96.0.10:80
   ↓
10.244.1.3:80
10.244.2.5:80
10.244.3.7:80

kube-proxy는 이 매핑을 만든다.
```

### 5️⃣ kube-proxy 동작 모드

kube-proxy는 3가지 모드가 있다.

#### ① userspace (옛날 방식)

```
Client
 ↓
kube-proxy
 ↓
Pod

문제 :
1. 느림
2. hop 추가

그래서 거의 안 쓴다.
```

#### ② iptables 모드 (가장 많이 사용)

```
Client
 ↓
iptables rule
 ↓
Pod

특징 :
1. 커널 레벨
2. 빠름
3. kub-proxy는 규칙만 설정

실제 forwarding은 Linux kernel이 한다.
```

#### ③ IPVS 모드 (고성능)

```
특징 :
L4 Load Balancer

지원 알고리즘 :
round robin
least connection
hash

대규모 클러스터에서 사용한다.
```

### 6️⃣ 실제 iptables 규칙

Service 생성하면 이런 rule이 생긴다.

```
예 :
KUBE-SERVICES
KUBE-SVC-XXXXX
KUBE-SEP-XXXXX

구조 :
Service IP
↓
KUBE-SVC
↓
KUBE-SEP
↓
Pod IP

예시 :
10.96.0.10:80
↓
KUBE-SVC-ABC
↓
KUBE-SEP-1 → 10.244.1.3
KUBE-SEP-2 → 10.244.2.5
```

### 7️⃣ kube-proxy는 패킷을 직접 보내지 않는다

```
많은 사람들이 이렇게 생각한다.
Client → kube-proxy → Pod (❌ 틀림)

실제로는
Client → iptables → Pod

kube-proxy는
👉 rule만 만든다
```

### 8️⃣ kube-proxy 위치

각 Node에 하나씩 있다.

```
Node1
├ kubelet
└ kube-proxy

Node2
├ kubelet
└ kube-proxy

이렇게 DaemonSet 형태로 실행된다.
```

### 9️⃣ Service 트래픽 실제 흐름

```
전체 흐름
Pod A
↓
Service IP
↓
iptables
↓
Pod B

즉 Service는 virtual IP 이다.
```

### 🔟 핵심 정리

```
kube-proxy 역할

Service 생성 감지
↓
Endpoint 확인
↓
iptables/IPVS rule 생성
↓
Service → Pod 트래픽 전달
```
