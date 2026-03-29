## 목차

- [1장 ingress 란](#1장-ingress-란)
  - [1️⃣ Ingress란 무엇인가](#1️⃣-ingress란-무엇인가)
  - [2️⃣ 왜 Ingress가 필요한가](#2️⃣-왜-ingress가-필요한가)
  - [3️⃣ Ingress 전체 구조](#3️⃣-ingress-전체-구조)
  - [4️⃣ Ingress Controller란](#4️⃣-ingress-controller란)
  - [5️⃣ 실제 Kubernetes에서 구조](#5️⃣-실제-kubernetes에서-구조)
  - [6️⃣ 실제 Linux에서 무엇이 실행되는가](#6️⃣-실제-linux에서-무엇이-실행되는가)
  - [7️⃣ 실제 nginx 설정 파일](#7️⃣-실제-nginx-설정-파일)
  - [8️⃣ Ingress 생성 방법](#8️⃣-ingress-생성-방법)
  - [9️⃣ 실제 패킷 흐름](#9️⃣-실제-패킷-흐름)
  - [🔟 kube-proxy 관계](#-kube-proxy-관계)
  - [11장 Linux에서 실제 확인](#11장-linux에서-실제-확인)
  - [12장 실제 DNS + Ingress 구조](#12장-실제-dns--ingress-구조)
  - [13장 TLS (HTTPS)](#13장-tls-https)
  - [14장 Kubernetes 네트워크 전체 그림](#14장-kubernetes-네트워크-전체-그림)
  - [15장 핵심 개념 정리](#15장-핵심-개념-정리)
  - [🔥 Service vs Ingress](#-service-vs-ingress)
  - [🔥 실제 Kubernetes 네트워크 핵심 구조](#-실제-kubernetes-네트워크-핵심-구조)
- [2장 NGINX Ingress 내부 구조](#2장-nginx-ingress-내부-구조)
  - [1️⃣ NGINX Ingress 내부 구조](#1️⃣-nginx-ingress-내부-구조)
    - [NGINX Ingress Pod 내부 구조](#nginx-ingress-pod-내부-구조)
    - [1️⃣ Controller Process](#1️⃣-controller-process)
    - [2️⃣ NGINX process](#2️⃣-nginx-process)
      - [NGINX worker 구조](#nginx-worker-구조)
    - [실제 nginx 설정 파일](#실제-nginx-설정-파일)
    - [config reload 과정](#config-reload-과정)
    - [실제 로그](#실제-로그)
  - [2️⃣ iptables + Ingress 실제 rule 분석](#2️⃣-iptables--ingress-실제-rule-분석)
    - [Ingress Controller는 보통 Service로 노출된다.](#ingress-controller는-보통-service로-노출된다)
    - [실제 패킷 흐름](#실제-패킷-흐름)
    - [실제 iptables rule](#실제-iptables-rule)
    - [실제 흐름 (Ingress)](#실제-흐름-ingress)

---

| ingress는 2026년 여전히 사용되나 Gateway API를 차세대 API로 밀고있다.

# 1장 ingress 란

## 1️⃣ Ingress란 무엇인가

한 줄 정의

```
HTTP/HTTPS 트래픽을 외부에서 받아 Kubernetes Service로 라우팅하는 Layer7 Gateway
```

즉 역할

```
외부 사용자
   ↓
Ingress
   ↓
Service
   ↓
Pod
```

## 2️⃣ 왜 Ingress가 필요한가

Service만 사용하면 보통 이렇게 된다.

```
Service A → NodePort 30001
Service B → NodePort 30002
Service C → NodePort 30003
```

외부 사용자는

```
http://nodeIP:30001
http://nodeIP:30002
```

이렇게 접속해야 한다.

문제

```
포트 관리 어려움
TLS 관리 어려움
도메인 기반 라우팅 불가능
```

그래서 등장

```
Ingress
```

## 3️⃣ Ingress 전체 구조

전체 아키텍처

```
Internet
   ↓
LoadBalancer (optional)
   ↓
Ingress Controller
   ↓
Service
   ↓
Pod
```

핵심

```
Ingress = 설정
Ingress Controller = 실제 서버
```

즉 Ingress는

```
단순한 Kubernetes resource
```

실제 트래픽 처리

```
NGINX
HAProxy
Traefik
```

같은 Ingress Controller

## 4️⃣ Ingress Controller란

한 줄 요약

```
Ingress 규칙을 읽어서 실제 프록시 서버(NGINX 등)를 구성하는 프로그램
```

대표

```
NGINX Ingress
Traefik
HAProxy
Istio Gateway
```

가장 많이 사용하는 것

```
NGINX Ingress Controller

이나 이제 안하겠다고 했던거 같은데
```

## 5️⃣ 실제 Kubernetes에서 구조

설치하면

```
kubectl get pods -n ingress-nginx
```

예

```
ingress-nginx-controller
```

그리고 Service

```
kubectl get svc -n ingress-nginx
```

예

```
ingress-nginx-controller
ClusterIP / NodePort / LoadBalancer
```

## 6️⃣ 실제 Linux에서 무엇이 실행되는가

Ingress Controller Pod 내부

프로세스

```
nginx
```

확인

```
kubectl exec -it ingress-nginx-controller -- ps -ef
```

보면

```
nginx: master process
nginx: worker process
```

즉

Ingress =

```
Kubernetes
   ↓
NGINX 설정 자동 생성
   ↓
NGINX reverse proxy
```

## 7️⃣ 실제 nginx 설정 파일

Pod 안

```
kubectl exec -it ingress-nginx-controller -- bash
```

파일

```
/etc/nginx/nginx.conf
또는
/etc/nginx/conf.d/
```

Ingress 생성하면 NGINX config 자동 생성됨

예

```
server {
  server_name example.com;

  location /api {
    proxy_pass http://svc-api;
  }
}
```

## 8️⃣ Ingress 생성 방법

예

Service 두 개

```
api-service
web-service
```

Ingress yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example
spec:
  rules:
    - host: example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80

          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-service
                port:
                  number: 80
```

라우팅 결과

```
example.com/api → api-service
example.com/ → web-service
```

## 9️⃣ 실제 패킷 흐름

사용자

```
curl http://example.com/api
```

패킷 흐름

```
Internet
   ↓
Node IP
   ↓
Ingress Controller Service
   ↓
NGINX
   ↓
Service ClusterIP
   ↓
kube-proxy
   ↓
Pod
```

실제 네트워크 단계

```
Client
 ↓
NodePort / LoadBalancer
 ↓
iptables
 ↓
Ingress Pod
 ↓
NGINX
 ↓
Service IP
 ↓
iptables
 ↓
Pod
```

## 🔟 kube-proxy 관계

Ingress Controller는

```
Service
```

로 실행된다.

즉

```
Client
 ↓
NodePort
 ↓
iptables (kube-proxy)
 ↓
Ingress Pod
```

그리고

Ingress → Service도

```
ClusterIP
 ↓
iptables
 ↓
Pod
```

즉 kube-proxy 두 번 사용됨.

## 11장 Linux에서 실제 확인

Ingress 확인

```
kubectl get ingress
```

Ingress Controller 확인

```
kubectl get pods -n ingress-nginx
```

Service 확인

```
kubectl get svc -n ingress-nginx
```

nginx 설정 확인

```
kubectl exec -it ingress-nginx-controller -- cat /etc/nginx/nginx.conf
```

iptables 확인

노드에서

```
iptables -t nat -L

찾기
KUBE-NODEPORT
```

## 12장 실제 DNS + Ingress 구조

일반적인 구조

```
example.com
   ↓
DNS
   ↓
LoadBalancer IP
   ↓
Ingress Controller
   ↓
Service
   ↓
Pod
```

## 13장 TLS (HTTPS)

Ingress에서 TLS 가능

```yaml
spec:
  tls:
    - hosts:
        - example.com
      secretName: tls-secret
```

secret 생성

```
kubectl create secret tls tls-secret \
--cert=cert.pem \
--key=key.pem
```

NGINX가 TLS termination 수행

## 14장 Kubernetes 네트워크 전체 그림

```
Internet
   ↓
DNS
   ↓
LoadBalancer
   ↓
Ingress Controller
   ↓
Service
   ↓
Pod
```

클러스터 내부

```
Pod
 ↓
CoreDNS
 ↓
Service
 ↓
kube-proxy
 ↓
Pod
```

## 15장 핵심 개념 정리

Ingress

```
HTTP/HTTPS 라우팅 규칙
```

Ingress Controller

```
실제 트래픽 처리하는 프록시 서버
```

Ingress 특징

```
Layer7 routing
host 기반
path 기반
TLS termination
```

## 🔥 Service vs Ingress

| 특징       | Service    | Ingress     |
| ---------- | ---------- | ----------- |
| Layer      | L4         | L7          |
| 라우팅     | 포트       | Host / Path |
| TLS        | 없음       | 있음        |
| 로드밸런싱 | kube-proxy | NGINX       |

## 🔥 실제 Kubernetes 네트워크 핵심 구조

```
Internet
   ↓
Ingress
   ↓
Service
   ↓
Pod
```

내부

```
Pod
 ↓
CoreDNS
 ↓
Service
 ↓
kube-proxy
 ↓
Pod
```

---

# 2장 NGINX Ingress 내부 구조

## 1️⃣ NGINX Ingress 내부 구조

한줄 정의

```
Kubernetes Ingress 규칙을 감시(watch)하여 NGINX 설정을 자동 생성하고 reload 하는 controller
```

즉 내부 구조

```
Ingress Resource
      │
      ▼
Ingress Controller
      │
      ▼
NGINX Config 생성
      │
      ▼
NGINX reload
```

### NGINX Ingress Pod 내부 구조

Ingress Controller Pod 안에는 두 개 주요 구성요소가 있다.

```
+----------------------------------+
| ingress-nginx-controller         |
|                                  |
| 1. controller process            |
| 2. nginx process                 |
|                                  |
+----------------------------------+
```

### 1️⃣ Controller Process

역할

```
Kubernetes API watch
Ingress 변경 감지
Service 변경 감지
Endpoint 변경 감지
```

즉

```
controller
   │
   ├─ watch Ingress
   ├─ watch Service
   ├─ watch EndpointSlice
   │
   ▼
nginx config 생성
```

실행 파일

```
/nginx-ingress-controller
```

확인

```
kubectl exec -it ingress-nginx-controller -- ps -ef
```

예

```
/nginx-ingress-controller
```

### 2️⃣ NGINX process

controller가 생성한 config를 기반으로

```
nginx reverse proxy
```

실행됨.

확인

```
ps -ef
```

결과

```
nginx: master process
nginx: worker process
```

#### NGINX worker 구조

```
            master
              │
    ┌─────────┼─────────┐
    │         │         │
 worker1   worker2   worker3
```

| 프로세스 | 역할           |
| -------- | -------------- |
| master   | 설정 관리      |
| worker   | 실제 요청 처리 |

### 실제 nginx 설정 파일

Ingress Controller Pod 안

```
/etc/nginx/nginx.conf
또는
/etc/nginx/conf.d/
```

확인

```
kubectl exec -it ingress-nginx-controller -- cat /etc/nginx/nginx.conf
```

Ingress 생성하면

```
server {
  server_name example.com;

  location /api {
      proxy_pass http://svc-api;
  }
}
```

자동 생성됨.

### config reload 과정

Ingress 변경

```
kubectl apply -f ingress.yaml
```

흐름

```
API Server
   │
   ▼
Ingress Controller watch
   │
   ▼
nginx config regenerate
   │
   ▼
nginx reload
```

reload 명령

```
nginx -s reload
```

이때 worker는 끊기지 않음

구조

```
old worker
   │
new worker start
   │
old worker 종료
```

그래서

```
zero downtime
```

### 실제 로그

```
kubectl logs ingress-nginx-controller
```

보면

```
Configuration changes detected
Backend successfully reloaded
```

## 2️⃣ iptables + Ingress 실제 rule 분석

### Ingress Controller는 보통 Service로 노출된다.

예

```
kubectl get svc -n ingress-nginx
```

결과

```
ingress-nginx-controller
Type: NodePort
Port: 80:30080
```

### 실제 패킷 흐름

외부 요청

```
curl http://nodeIP:30080
```

패킷 흐름

```
Client
  │
  ▼
Node IP:30080
  │
  ▼
iptables
  │
  ▼
Ingress Controller Pod
  │
  ▼
NGINX
  │
  ▼
Service
  │
  ▼
Pod
```

### 실제 iptables rule

노드에서

```
iptables -t nat -L
```

NodePort rule

```
KUBE-NODEPORT

[예]
Chain KUBE-NODEPORT
target     prot opt source destination
KUBE-SVC-XXXX tcp -- 0.0.0.0/0 0.0.0.0/0 tcp dpt:30080
```

Service rule

```
Chain KUBE-SVC-XXXX

[예]
KUBE-SEP-AAAA
KUBE-SEP-BBBB
```

Endpoint rule

```
Chain KUBE-SEP-AAAA

[예]
DNAT tcp -- 0.0.0.0/0 0.0.0.0/0 to:10.244.1.5:80
```

즉

```
NodePort
 ↓
Service
 ↓
Endpoint
 ↓
Pod
```

### 실제 흐름 (Ingress)

```
Client
 ↓
NodePort
 ↓
iptables
 ↓
Ingress Pod
 ↓
NGINX
 ↓
Service
 ↓
iptables
 ↓
Pod
```

즉 iptables 두 번 사용
