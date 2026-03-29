## 목차

- [1장 Gateway API 란?](#1장-gateway-api-란)
  - [1️⃣ Gateway API 한줄 정의](#1️⃣-gateway-api-한줄-정의)
  - [2️⃣ Gateway API 전체 구조](#2️⃣-gateway-api-전체-구조)
  - [3️⃣ Gateway API 핵심 개념](#3️⃣-gateway-api-핵심-개념)
  - [4️⃣ 어디서 실행되는가](#4️⃣-어디서-실행되는가)
  - [5️⃣ 실제 Linux에서 무엇이 실행되는가](#5️⃣-실제-linux에서-무엇이-실행되는가)
  - [6️⃣ Gateway API CRD](#6️⃣-gateway-api-crd)
    - [CRD 실제 저장 위치 (Linux)](#crd-실제-저장-위치-linux)
    - [실제 확인](#실제-확인)
  - [7️⃣ GatewayClass](#7️⃣-gatewayclass)
  - [8️⃣ Gateway](#8️⃣-gateway)
  - [9️⃣ HTTPRoute](#9️⃣-httproute)
  - [🔟 Gateway API 전체 흐름](#-gateway-api-전체-흐름)
    - [전체 요청 흐름](#전체-요청-흐름)
    - [실제 네트워크 흐름](#실제-네트워크-흐름)
  - [11장 Gateway Controller 내부 동작](#11장-gateway-controller-내부-동작)
  - [12장 실제 config 생성](#12장-실제-config-생성)
  - [13장 Linux에서 실제 확인](#13장-linux에서-실제-확인)
  - [14장 Gateway API vs Ingress](#14장-gateway-api-vs-ingress)
  - [15장 역할 분리 (Gateway API 핵심)](#15장-역할-분리-gateway-api-핵심)
  - [🔥 Kubernetes 네트워크 계층](#-kubernetes-네트워크-계층)
  - [🚀 마지막 핵심](#-마지막-핵심)
- [2장 envoy-gateway-api 가 무엇인가](#2장-envoy-gateway-api-가-무엇인가)
  - [1️⃣ Envoy Gateway 한줄 정리](#1️⃣-envoy-gateway-한줄-정리)
  - [2️⃣ 어디서 실행되는가](#2️⃣-어디서-실행되는가)
  - [3️⃣ 실제 Linux에서 생성되는 것](#3️⃣-실제-linux에서-생성되는-것)
  - [4️⃣ 설치](#4️⃣-설치)
  - [5️⃣ Gateway API CRD 확인](#5️⃣-gateway-api-crd-확인)
  - [6️⃣ Gateway 생성](#6️⃣-gateway-생성)
  - [7️⃣ HTTPRoute 생성](#7️⃣-httproute-생성)
  - [8️⃣ 실제 흐름](#8️⃣-실제-흐름)
  - [9️⃣ iptables 확인](#9️⃣-iptables-확인)
  - [🔟 tcpdump로 실제 패킷 보기](#-tcpdump로-실제-패킷-보기)
  - [11장 Envoy config 확인](#11장-envoy-config-확인)
  - [12️장 Envoy Gateway 내부 구조](#12️장-envoy-gateway-내부-구조)
  - [13장 Linux 네트워크 레벨](#13장-linux-네트워크-레벨)
  - [14장 Envoy Proxy 내부 구조](#14장-envoy-proxy-내부-구조)
  - [15장 핵심 개념 정리](#15장-핵심-개념-정리)
- [3장 좀 더 정확하게 Envoy-gateway 에 대해 알아보자](#3장-좀-더-정확하게-envoy-gateway-에-대해-알아보자)
  - [1️⃣ 지금 떠있는 Pod 2개의 역할](#1️⃣-지금-떠있는-pod-2개의-역할)
    - [① envoy-gateway Pod](#-envoy-gateway-pod)
      - [역할](#역할)
      - [어디서 실행되는가](#어디서-실행되는가)
      - [Linux 내부에서 볼 수 있는 것](#linux-내부에서-볼-수-있는-것)
    - [② envoy-envoy-gateway-system-my-gateway Pod](#-envoy-envoy-gateway-system-my-gateway-pod)
      - [Pod 이름이 긴 이유](#pod-이름이-긴-이유)
      - [Envoy 내부 확인](#envoy-내부-확인)
  - [2️⃣ Gateway API 리소스 구조](#2️⃣-gateway-api-리소스-구조)
  - [3️⃣ GatewayClass](#3️⃣-gatewayclass)
  - [4️⃣ Gateway](#4️⃣-gateway)
  - [5️⃣ HTTPRoute](#5️⃣-httproute)
    - [전체 yaml](#전체-yaml)
    - [HTTPRoute 핵심 필드](#httproute-핵심-필드)
      - [parentRefs](#parentrefs)
      - [hostnames](#hostnames)
      - [rules](#rules)
    - [matches 옵션](#matches-옵션)
      - [path match](#path-match)
      - [method](#method)
      - [header match](#header-match)
      - [query match](#query-match)
    - [filters](#filters)
      - [header 추가](#header-추가)
      - [path rewrite](#path-rewrite)
    - [backendRefs](#backendrefs)
      - [weight](#weight)
  - [6️⃣ TCPRoute](#6️⃣-tcproute)
    - [TCPRoute 전체 YAML](#tcproute-전체-yaml)
  - [7️⃣ TLSRoute](#7️⃣-tlsroute)
    - [TLSRoute가 필요한 이유](#tlsroute가-필요한-이유)
    - [TLSRoute yaml 전체 구조](#tlsroute-yaml-전체-구조)
    - [TLSRoute 전체 YAML 예시](#tlsroute-전체-yaml-예시)
    - [실제 TLS 패킷 흐름](#실제-tls-패킷-흐름)
    - [Envoy 내부 변환](#envoy-내부-변환)
    - [Linux에서 확인](#linux에서-확인)
  - [8️⃣ GRPCRoute](#8️⃣-grpcroute)
    - [gRPC 요청 구조](#grpc-요청-구조)
    - [GRPCRoute 전체 구조](#grpcroute-전체-구조)
    - [GRPCRoute 전체 YAML 예시](#grpcroute-전체-yaml-예시)
    - [실제 gRPC 흐름](#실제-grpc-흐름)
    - [Envoy 내부 변환](#envoy-내부-변환-1)
    - [Linux에서 확인](#linux에서-확인-1)
  - [9️⃣ UDPRoute](#9️⃣-udproute)
    - [언제 사용하나](#언제-사용하나)
    - [UDPRoute 전체 YAML 예시](#udproute-전체-yaml-예시)
    - [Envoy 내부 변환](#envoy-내부-변환-2)
  - [10. 정리](#10-정리)
    - [전체적으로](#전체적으로)
    - [Route 타입 전체 비교](#route-타입-전체-비교)
    - [Gateway API 전체 구조](#gateway-api-전체-구조)
    - [Envoy 내부 구조](#envoy-내부-구조)
  - [11. 실제 전체 흐름](#11-실제-전체-흐름)
  - [12. Linux 네트워크 흐름](#12-linux-네트워크-흐름)
- [4장 핵심 아키텍처](#4장-핵심-아키텍처)
  - [1️⃣ Envoy xDS Protocol (Control Plane 핵심)](#1️⃣-envoy-xds-protocol-control-plane-핵심)
    - [xDS 종류](#xds-종류)
    - [실제 흐름](#실제-흐름)
    - [구조](#구조)
    - [실제 Envoy config 내부 구조](#실제-envoy-config-내부-구조)
  - [2️⃣ Envoy Cluster / Endpoint Discovery](#2️⃣-envoy-cluster--endpoint-discovery)
    - [Cluster Load Balancing](#cluster-load-balancing)
    - [Endpoint update](#endpoint-update)
  - [3️⃣ Gateway API → Envoy Config 변환 과정](#3️⃣-gateway-api--envoy-config-변환-과정)
    - [변환 흐름](#변환-흐름)
  - [4️⃣ Envoy L4 vs L7 Routing 내부 구조](#4️⃣-envoy-l4-vs-l7-routing-내부-구조)
    - [L4 routing](#l4-routing)
    - [L7 routing](#l7-routing)
  - [5️⃣ Envoy Connection Pool / Circuit Breaker](#5️⃣-envoy-connection-pool--circuit-breaker)
    - [Connection Pool](#connection-pool)
    - [Circuit Breaker](#circuit-breaker)
  - [지금까지 전체 구조](#지금까지-전체-구조)
  - [Envoy admin API](#envoy-admin-api)
    - [1️⃣ Envoy admin API (필수)](#1️⃣-envoy-admin-api-필수)
    - [2️⃣ Envoy 실제 config 확인](#2️⃣-envoy-실제-config-확인)
    - [3️⃣ iptables → Envoy 흐름](#3️⃣-iptables--envoy-흐름)

---

| ingress는 2026년 여전히 사용되나 Gateway API를 차세대 API로 밀고있다.

| 기술        | 상태        |
| ----------- | ----------- |
| Ingress     | 여전히 사용 |
| Gateway API | 차세대 API  |

# 1장 Gateway API 란?

## 1️⃣ Gateway API 한줄 정의

```
Ingress보다 확장 가능한 Kubernetes L4/L7 네트워크 API
```

즉

```
Ingress
   ↓
Gateway API (차세대)
```

## 2️⃣ Gateway API 전체 구조

Ingress 구조

```
Ingress
   │
Ingress Controller
```

Gateway API 구조

```
GatewayClass
     │
Gateway
     │
Route (HTTPRoute / TCPRoute)
     │
Service
     │
Pod
```

즉

```
Client
 ↓
Gateway
 ↓
Route
 ↓
Service
 ↓
Pod
```

## 3️⃣ Gateway API 핵심 개념

Gateway API는 4가지 핵심 리소스가 있다.

| 리소스       | 역할                             |
| ------------ | -------------------------------- |
| GatewayClass | 어떤 gateway controller 사용할지 |
| Gateway      | 실제 listener                    |
| Route        | 라우팅 규칙                      |
| Backend      | Service                          |

게이트웨이클래스(GatewayClass): 공통 구성을 가진 게이트웨이들의 집합을 정의하며, 해당 클래스를 구현하는 컨트롤러에 의해 관리된다.

게이트웨이(Gateway): 클라우드 로드 밸런서와 같은 트래픽 처리 인프라의 인스턴스를 정의한다.

HTTP라우트(HTTPRoute): 게이트웨이 리스너에서 오는 트래픽을 백엔드 네트워크 엔드포인트들의 표현으로 매핑하기 위한 HTTP 전용 규칙을 정의한다. 이러한 엔드포인트들은 종종 서비스로 표현된다.

GRPC라우트(GRPCRoute): 게이트웨이 리스너에서 오는 트래픽을 백엔드 네트워크 엔드포인트들의 표현으로 매핑하기 위한 gRPC 전용 규칙을 정의한다. 이러한 엔드포인트들은 종종 서비스로 표현된다.

## 4️⃣ 어디서 실행되는가

중요한 점

Gateway API 자체는

```
Kubernetes resource
```

일 뿐이다.

실제 트래픽 처리는

```
Gateway Controller
```

가 한다.

대표적으로 다음과 같은 것들이 있다.

| Controller    | 내부 proxy |
| ------------- | ---------- |
| NGINX Gateway | NGINX      |
| Envoy Gateway | Envoy      |
| Traefik       | Traefik    |
| Istio         | Envoy      |

예

```
kubectl get pods -n envoy-gateway
```

결과

```
envoy-gateway
envoy-proxy
```

## 5️⃣ 실제 Linux에서 무엇이 실행되는가

예: Envoy Gateway

Pod 내부

```
envoy
```

프로세스 확인

```
➜  kubectl exec -it envoy-proxy -- ps -ef

envoy
```

즉

```
Gateway API
    ↓
Gateway Controller
    ↓
Envoy / NGINX
```

## 6️⃣ Gateway API CRD

Gateway API는 CRD로 설치된다.

확인

```
kubectl get crd | grep gateway
```

예

```
gatewayclasses.gateway.networking.k8s.io
gateways.gateway.networking.k8s.io
httproutes.gateway.networking.k8s.io
tcproutes.gateway.networking.k8s.io
```

### CRD 실제 저장 위치 (Linux)

Kubernetes etcd에 저장된다.

즉

```
/var/lib/etcd
```

안에 저장됨.

하지만 직접 파일로 보기는 어렵고 API로 확인한다.

### 실제 확인

```
kubectl get gateway
kubectl get httproute
```

YAML 확인

```
kubectl get gateway example -o yaml
```

## 7️⃣ GatewayClass

한 줄

```
어떤 Gateway Controller 사용할지 정의
```

예

```
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: nginx
spec:
  controllerName: gateway.nginx.org/nginx-gateway-controller
```

controllerName

```
어떤 controller가 처리할지
```

## 8️⃣ Gateway

Gateway는

```
실제 네트워크 listener
```

예

```
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: example
spec:
  gatewayClassName: nginx
  listeners:
  - port: 80
    protocol: HTTP
```

즉

```
port 80 listener 생성
```

## 9️⃣ HTTPRoute

Ingress와 가장 비슷한 것.

예

```
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: api
spec:
  parentRefs:
  - name: example
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /api
    backendRefs:
    - name: api-service
      port: 80
```

결과

```
/api → api-service
```

## 🔟 Gateway API 전체 흐름

### 전체 요청 흐름

```
Client
  │
  ▼
Gateway (listener)
  │
  ▼
Route
  │
  ▼
Service
  │
  ▼
Pod
```

### 실제 네트워크 흐름

```
Client
 ↓
NodePort / LoadBalancer
 ↓
Gateway Pod
 ↓
Envoy / NGINX
 ↓
Service
 ↓
iptables
 ↓
Pod
```

## 11장 Gateway Controller 내부 동작

Controller는 watch 한다.

watch 대상

```
GatewayClass
Gateway
HTTPRoute
Service
EndpointSlice
```

흐름

```
Kubernetes API
   │
   ▼
Controller watch
   │
   ▼
Proxy config 생성
   │
   ▼
Envoy reload
```

## 12장 실제 config 생성

Envoy Gateway 예

config 파일

```
/etc/envoy/envoy.yaml
```

또는

```
xDS API
```

즉

```
controller
   ↓
generate config
   ↓
envoy 적용
```

## 13장 Linux에서 실제 확인

Gateway

```
kubectl get gateway
```

Route

```
kubectl get httproute
```

Gateway Controller

```
kubectl get pods -n envoy-gateway
```

proxy process

```
kubectl exec -it envoy-proxy -- ps -ef
```

envoy config

```
kubectl exec -it envoy-proxy -- cat /etc/envoy/envoy.yaml
```

## 14장 Gateway API vs Ingress

| 특징      | Ingress       | Gateway API   |
| --------- | ------------- | ------------- |
| 구조      | 단일 resource | 여러 resource |
| 확장성    | 낮음          | 높음          |
| TCP/UDP   | 제한          | 지원          |
| 역할 분리 | 없음          | 가능          |

## 15장 역할 분리 (Gateway API 핵심)

Gateway API는

```
Infra team
App team
```

분리 가능.

예시로

infra team은

```
Gateway
GatewayClass
```

개발자

```
HTTPRoute
```

## 🔥 Kubernetes 네트워크 계층

```
L3  CNI (Calico)
L4  Service (kube-proxy)
L7  Gateway API
```

## 🚀 마지막 핵심

Gateway API는

```
Ingress + Service Mesh 중간
```

개념이다.

즉

```
Ingress
   ↓
Gateway API
   ↓
Service Mesh
```

---

# 2장 envoy-gateway-api 가 무엇인가

## 1️⃣ Envoy Gateway 한줄 정리

Gateway API 리소스를 읽어서 Envoy Proxy 설정을 자동 생성하는 Kubernetes Controller

구성

```
Gateway API
   │
   ▼
Envoy Gateway Controller
   │ (xDS config 생성)
   ▼
Envoy Proxy
   │
   ▼
Service / Pod
```

즉

```
Gateway API = Kubernetes 설정
Envoy Gateway = Controller
Envoy Proxy = 실제 트래픽 처리
```

## 2️⃣ 어디서 실행되는가

Envoy Gateway 설치하면 2가지 Pod가 생김

```
envoy-gateway-system namespace

envoy-gateway
└─ controller

envoy
└─ proxy
```

확인

```
kubectl get pod -n envoy-gateway-system
```

예

```
NAME                             READY
envoy-gateway-7c9f8c5c5d-xxx     1/1
envoy-abc123                     1/1
```

역할

| Pod           | 역할                   |
| ------------- | ---------------------- |
| envoy-gateway | Gateway API Controller |
| envoy         | 실제 L7 proxy          |

## 3️⃣ 실제 Linux에서 생성되는 것

Pod 내부에서 보면

```
/etc/envoy/envoy.yaml
또는
/config/envoy.yaml
```

확인

```
kubectl exec -it envoy-xxxxx -n envoy-gateway-system -- sh
```

그리고

```
ls /etc/envoy
또는
curl localhost:9901/config_dump
```

Envoy admin port

```
9901
```

## 4️⃣ 설치

https://gateway.envoyproxy.io/docs/install/install-helm/

확인

```
kubectl get pods -n envoy-gateway-system
```

```
➜  envoy-gateway k get all -n envoy-gateway-system
NAME                                                                  READY   STATUS    RESTARTS   AGE
pod/envoy-envoy-gateway-system-my-gateway-1d35c6f5-6df4dd5f4b-fcbrf   2/2     Running   0          32s
pod/envoy-gateway-85b697dc46-787jk                                    1/1     Running   0          3m57s

NAME                                                     TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                                   AGE
service/envoy-envoy-gateway-system-my-gateway-1d35c6f5   LoadBalancer   10.111.24.21   192.168.0.100   80:32520/TCP                              32s
service/envoy-gateway                                    ClusterIP      10.96.75.81    <none>          18000/TCP,18001/TCP,18002/TCP,19001/TCP   3m57s

NAME                                                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/envoy-envoy-gateway-system-my-gateway-1d35c6f5   1/1     1            1           32s
deployment.apps/envoy-gateway                                    1/1     1            1           3m57s

NAME                                                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/envoy-envoy-gateway-system-my-gateway-1d35c6f5-6df4dd5f4b   1         1         1       32s
replicaset.apps/envoy-gateway-85b697dc46                                    1         1         1       3m57s

```

## 5️⃣ Gateway API CRD 확인

Envoy Gateway 설치하면 CRD 생성됨

```
kubectl get crd | grep gateway
```

예

```
gatewayclasses.gateway.networking.k8s.io
gateways.gateway.networking.k8s.io
httproutes.gateway.networking.k8s.io
tcproutes.gateway.networking.k8s.io
```

핵심 리소스

| 리소스       | 역할                    |
| ------------ | ----------------------- |
| GatewayClass | Gateway controller 정의 |
| Gateway      | 실제 LB                 |
| HTTPRoute    | HTTP routing            |

## 6️⃣ Gateway 생성

GatewayClass

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: eg
spec:
  controllerName: gateway.envoyproxy.io/gatewayclass-controller
```

확인

```
kubectl get gatewayclass
```

Gateway 생성

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: my-gateway
spec:
  gatewayClassName: eg
  listeners:
    - name: http
      port: 80
      protocol: HTTP
```

생성

```
kubectl apply -f gateway.yaml
```

## 7️⃣ HTTPRoute 생성

서비스 생성

```
kubectl create deployment demo --image=nginx
kubectl expose deployment demo --port=80
```

HTTPRoute

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: demo-route
spec:
  parentRefs:
    - name: my-gateway
  rules:
    - backendRefs:
        - name: demo
          port: 80
```

## 8️⃣ 실제 흐름

전체 네트워크 흐름

```
Client
   │
   ▼
Node
   │
   ▼
iptables
   │
   ▼
Envoy Pod
   │
   ▼
Service
   │
   ▼
Pod
```

## 9️⃣ iptables 확인

Envoy Service 확인

```
kubectl get svc -n envoy-gateway-system
```

예

```
envoy
TYPE: LoadBalancer
CLUSTER-IP: 10.96.100.10
```

Node iptables 확인

```
sudo iptables -t nat -L -n
```

보면

```
KUBE-SVC-XXXXX
```

Service → Pod NAT

```
➜  envoy-gateway iptables -t nat -L KUBE-SERVICES
Chain KUBE-SERVICES (2 references)
target     prot opt source               destination
KUBE-SVC-C4XVZGJMAV66GLUE  tcp  --  anywhere             10.96.75.81          /* envoy-gateway-system/envoy-gateway:grpc cluster IP */
KUBE-SVC-OUT6WRQRF44YY4NE  tcp  --  anywhere             10.96.75.81          /* envoy-gateway-system/envoy-gateway:metrics cluster IP */
KUBE-SVC-ALBMZOVDCJFWH6PI  tcp  --  anywhere             10.96.75.81          /* envoy-gateway-system/envoy-gateway:wasm cluster IP */
KUBE-SVC-YJZCXBRBLW6VBGV6  tcp  --  anywhere             10.111.24.21         /* envoy-gateway-system/envoy-envoy-gateway-system-my-gateway-1d35c6f5:http-80 cluster IP */
KUBE-EXT-YJZCXBRBLW6VBGV6  tcp  --  anywhere             192.168.0.100        /* envoy-gateway-system/envoy-envoy-gateway-system-my-gateway-1d35c6f5:http-80 loadbalancer IP */
KUBE-SVC-UCI7C4AJAJMKGXO3  tcp  --  anywhere             10.96.75.81          /* envoy-gateway-system/envoy-gateway:ratelimit cluster IP */
KUBE-NODEPORTS  all  --  anywhere             anywhere             /* kubernetes service nodeports; NOTE: this must be the last rule in this chain */ ADDRTYPE match dst-type LOCAL

➜  envoy-gateway iptables -t nat -L KUBE-NODEPORTS
Chain KUBE-NODEPORTS (1 references)
target     prot opt source               destination
KUBE-EXT-YJZCXBRBLW6VBGV6  tcp  --  anywhere             anywhere             /* envoy-gateway-system/envoy-envoy-gateway-system-my-gateway-1d35c6f5:http-80 */

➜  envoy-gateway iptables -t nat -L KUBE-EXT-YJZCXBRBLW6VBGV6
Chain KUBE-EXT-YJZCXBRBLW6VBGV6 (2 references)
target     prot opt source               destination
KUBE-SVC-YJZCXBRBLW6VBGV6  all  --  10.244.0.0/16        anywhere             /* pod traffic for envoy-gateway-system/envoy-envoy-gateway-system-my-gateway-1d35c6f5:http-80 external destinations */
KUBE-MARK-MASQ  all  --  anywhere             anywhere             /* masquerade LOCAL traffic for envoy-gateway-system/envoy-envoy-gateway-system-my-gateway-1d35c6f5:http-80 external destinations */ ADDRTYPE match src-type LOCAL
KUBE-SVC-YJZCXBRBLW6VBGV6  all  --  anywhere             anywhere             /* route LOCAL traffic for envoy-gateway-system/envoy-envoy-gateway-system-my-gateway-1d35c6f5:http-80 external destinations */ ADDRTYPE match src-type LOCAL
KUBE-SVL-YJZCXBRBLW6VBGV6  all  --  anywhere             anywhere
```

## 🔟 tcpdump로 실제 패킷 보기

Node에서

```
tcpdump -i any port 80
```

또는

Envoy Pod

```
kubectl exec -it envoy-xxxx -n envoy-gateway-system -- tcpdump -i eth0
```

## 11장 Envoy config 확인

Admin API

```
kubectl port-forward svc/envoy -n envoy-gateway-system 9901:9901
```

브라우저

```
http://localhost:9901
```

확인 가능

```
/clusters
/listeners
/routes
/config_dump
```

## 12️장 Envoy Gateway 내부 구조

```
Gateway API
   │
   ▼
Envoy Gateway Controller
   │
   ├─ Kubernetes Watch
   │
   ├─ GatewayClass
   ├─ Gateway
   ├─ HTTPRoute
   │
   ▼
IR (Intermediate Representation)
   │
   ▼
xDS config
   │
   ▼
Envoy Proxy
```

## 13장 Linux 네트워크 레벨

Envoy pod 들어가서

interface

```
ip a
```

route

```
ip route
```

socket

```
ss -lntp
```

예

```
0.0.0.0:80
0.0.0.0:10000
127.0.0.1:9901
```

## 14장 Envoy Proxy 내부 구조

Envoy는 event-driven proxy

구성

```
listener
   │
   ▼
filter chain
   │
   ▼
route
   │
   ▼
cluster
   │
   ▼
endpoint
```

예

```
listener : 0.0.0.0:80
route : / -> demo-service
cluster : demo-service
endpoint : pod ip
```

## 15장 핵심 개념 정리

| 개념          | 설명       |
| ------------- | ---------- |
| GatewayClass  | controller |
| Gateway       | LB         |
| HTTPRoute     | routing    |
| Envoy Gateway | controller |
| Envoy Proxy   | data plane |

---

# 3장 좀 더 정확하게 Envoy-gateway 에 대해 알아보자

| 현재 kubernetes 버전이 1.29이므로 envoy-gateway v1.3.1 을 사용했다

```
➜  envoy-gateway k get all -n envoy-gateway-system
NAME                                                                  READY   STATUS    RESTARTS   AGE
pod/envoy-envoy-gateway-system-my-gateway-1d35c6f5-6df4dd5f4b-fcbrf   2/2     Running   0          11m
pod/envoy-gateway-85b697dc46-787jk                                    1/1     Running   0          15m

NAME                                                     TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                                   AGE
service/envoy-envoy-gateway-system-my-gateway-1d35c6f5   LoadBalancer   10.111.24.21   192.168.0.100   80:32520/TCP                              11m
service/envoy-gateway                                    ClusterIP      10.96.75.81    <none>          18000/TCP,18001/TCP,18002/TCP,19001/TCP   15m

NAME                                                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/envoy-envoy-gateway-system-my-gateway-1d35c6f5   1/1     1            1           11m
deployment.apps/envoy-gateway                                    1/1     1            1           15m

NAME                                                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/envoy-envoy-gateway-system-my-gateway-1d35c6f5-6df4dd5f4b   1         1         1       11m
replicaset.apps/envoy-gateway-85b697dc46                                    1         1         1       15m
```

## 1️⃣ 지금 떠있는 Pod 2개의 역할

환경

```
pod/envoy-envoy-gateway-system-my-gateway-xxxx   2/2
pod/envoy-gateway-xxxxx                          1/1
```

### ① envoy-gateway Pod

```
envoy-gateway-85b697dc46-787jk
```

#### 역할

```
Gateway API Controller
```

즉 Kubernetes API를 감시한다.

한줄정리

```
Gateway API 리소스를 읽어 Envoy Proxy 설정을 생성하는 Controller
```

감시하는 리소스

```
GatewayClass
Gateway
HTTPRoute
TCPRoute
```

그리고 이것들을 Envoy 설정(xDS) 으로 변환한다.

실제 동작 흐름

```
kube-apiserver
       │
       ▼
envoy-gateway controller
       │
       ▼
xDS config 생성
       │
       ▼
Envoy proxy에 전달
```

#### 어디서 실행되는가

```
envoy-gateway-system namespace
```

확인

```
kubectl get pod -n envoy-gateway-system
```

#### Linux 내부에서 볼 수 있는 것

Pod 들어가기

```
kubectl exec -it envoy-gateway-xxxxx -n envoy-gateway-system -- sh
```

프로세스

```
ps aux
```

보면

```
envoy-gateway
```

로그

```
kubectl logs envoy-gateway-xxxxx -n envoy-gateway-system
```

### ② envoy-envoy-gateway-system-my-gateway Pod

```
envoy-envoy-gateway-system-my-gateway-xxxx
```

역할

```
Envoy Proxy Pod

실제 트래픽 처리
```

한줄정리

```
실제 HTTP / TCP 트래픽을 처리하는 L4/L7 proxy
```

예

```
Client → Envoy → Service → Pod
```

#### Pod 이름이 긴 이유

Gateway 생성하면

```
Gateway
   │
   ▼
Envoy Deployment 생성
```

즉

```
Gateway = Envoy proxy instance
```

#### Envoy 내부 확인

들어가기

```
kubectl exec -it envoy-envoy-gateway-system-my-gateway-xxx -n envoy-gateway-system -- sh
```

포트 확인

```
ss -lntp
```

예

```
0.0.0.0:80
0.0.0.0:10000
127.0.0.1:9901

9901 → Envoy admin API
```

## 2️⃣ Gateway API 리소스 구조

전체 구조

```
GatewayClass
     │
     ▼
Gateway
     │
     ▼
Route
 ├─ HTTPRoute
 ├─ TCPRoute
 └─ GRPCRoute
```

## 3️⃣ GatewayClass

역할

```
어떤 Gateway Controller가 처리할지 정의
```

한줄정리

```
Gateway Controller 종류 정의
```

예

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: eg
spec:
  controllerName: gateway.envoyproxy.io/gatewayclass-controller
```

의미

```
controllerName:
gateway.envoyproxy.io/gatewayclass-controller
```

이걸 보고

```
envoy-gateway controller
```

가 처리함.

확인

```
kubectl get gatewayclass
```

## 4️⃣ Gateway

역할

```
LoadBalancer / Entry point
```

예

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: my-gateway
spec:
  gatewayClassName: eg
  listeners:
    - name: http
      port: 80
      protocol: HTTP
```

의미

```
80 포트로 들어오는 트래픽 받겠다
```

실제로 일어나는 일

```
Gateway 생성 →

Envoy Deployment
Envoy Service

자동 생성
```

확인

```
kubectl get svc -n envoy-gateway-system
```

```
➜  k get gateway -A
NAMESPACE              NAME         CLASS   ADDRESS         PROGRAMMED   AGE
envoy-gateway-system   my-gateway   eg      192.168.0.100   True         39m
```

## 5️⃣ HTTPRoute

역할

```
HTTP routing rule
```

한줄정리

```
HTTP 트래픽 라우팅 규칙
```

예

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: demo-route
spec:
  parentRefs:
    - name: my-gateway
  rules:
    - backendRefs:
        - name: demo
          port: 80
```

의미

```
HTTP 요청 → demo service
```

구조

```
HTTP request
     │
     ▼
Envoy listener
     │
     ▼
route match
     │
     ▼
backend service
```

### 전체 yaml

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: example-route
  namespace: default

spec:
  parentRefs:
    - name: my-gateway
      namespace: default
      sectionName: http
      port: 80

  hostnames:
    - example.com
    - api.example.com

  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /api

          method: GET

          headers:
            - name: x-version
              value: v1

          queryParams:
            - name: user
              value: admin

      filters:
        - type: RequestHeaderModifier
          requestHeaderModifier:
            add:
              - name: x-added
                value: envoy

        - type: URLRewrite
          urlRewrite:
            path:
              type: ReplacePrefixMatch
              replacePrefixMatch: /backend

      backendRefs:
        - name: api-service
          port: 80
          weight: 90

        - name: api-service-v2
          port: 80
          weight: 10
```

### HTTPRoute 핵심 필드

#### parentRefs

```
어떤 Gateway에 붙을지

parentRefs:
- name: my-gateway
```

즉

```
HTTPRoute → Gateway 연결
```

#### hostnames

```
Host 기반 라우팅

hostnames:
- example.com
```

HTTP 요청

```
Host: example.com
```

매칭

#### rules

라우팅 규칙

```
rules:
- matches:
```

### matches 옵션

HTTP 매칭 조건

| 조건        | 설명         |
| ----------- | ------------ |
| path        | URL path     |
| method      | HTTP method  |
| headers     | header match |
| queryParams | query string |

#### path match

```
path:
  type: PathPrefix
  value: /api
```

지원 타입

| 타입              | 의미        |
| ----------------- | ----------- |
| Exact             | 정확히 일치 |
| PathPrefix        | prefix      |
| RegularExpression | regex       |

#### method

```
method: GET
```

#### header match

```
headers:
- name: x-version
  value: v1
```

#### query match

```
queryParams:
- name: user
  value: admin

-> 요청
/api?user=admin
```

### filters

HTTP 요청 변경

| 필터                   | 설명            |
| ---------------------- | --------------- |
| RequestHeaderModifier  | header 변경     |
| ResponseHeaderModifier | response header |
| URLRewrite             | path rewrite    |
| RequestRedirect        | redirect        |
| ExtensionRef           | custom filter   |

#### header 추가

```
filters:
- type: RequestHeaderModifier
```

#### path rewrite

```
filters:
- type: URLRewrite

[예]
/api/test
↓
/backend/test
```

### backendRefs

어디로 보낼지

```
backendRefs:
- name: api-service
  port: 80
```

#### weight

```
트래픽 분산
```

```
weight: 90

[예]
api-service = 90%
api-service-v2 = 10%
```

## 6️⃣ TCPRoute

역할

```
L4 TCP routing
```

한줄정리

```
TCP 트래픽 라우팅
```

예

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: TCPRoute
metadata:
  name: tcp-route
spec:
  parentRefs:
    - name: my-gateway
  rules:
    - backendRefs:
        - name: mysql
          port: 3306
```

의미

```
TCP packet → mysql service
```

특징

```
HTTP parsing 없음
단순 L4 forwarding
```

### TCPRoute 전체 YAML

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: TCPRoute
metadata:
  name: mysql-route

spec:
  parentRefs:
    - name: my-gateway

  rules:
    - backendRefs:
        - name: mysql
          port: 3306
          weight: 100
```

## 7️⃣ TLSRoute

한줄정리

```
TLS SNI(Server Name Indication) 를 기반으로 TCP 트래픽을 라우팅하는 규칙
```

즉

```
TLS handshake
     │
     ▼
SNI 확인
     │
     ▼
Service 선택
```

### TLSRoute가 필요한 이유

일반 TCPRoute는 SNI를 못 본다.

예

```
443 포트
 ├ api.example.com
 ├ admin.example.com
 └ shop.example.com
```

TCPRoute는 포트만 보고 라우팅

하지만 TLSRoute는 SNI 보고 라우팅이 가능하다.

즉

```
TLSRoute = TCPRoute + SNI routing
```

### TLSRoute yaml 전체 구조

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: TLSRoute
metadata:
spec: parentRefs
  hostnames
  rules
```

### TLSRoute 전체 YAML 예시

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: TLSRoute
metadata:
  name: tls-route
  namespace: default

spec:
  parentRefs: # 어떤 Gateway listener를 사용할지
    - name: my-gateway
      sectionName: tls

  hostnames: # TLS SNI 매칭
    - api.example.com
    - admin.example.com

  rules: # 트래픽 전달 대상
    - backendRefs:
        - name: api-service
          port: 443
          weight: 80

        - name: api-service-v2
          port: 443
          weight: 20
```

### 실제 TLS 패킷 흐름

```
Client
   │
   ▼
TLS ClientHello
   │
   ▼
SNI = api.example.com
   │
   ▼
Envoy TLSRoute match
   │
   ▼
backend service
```

### Envoy 내부 변환

TLSRoute → Envoy config

```
listener
   │
   ▼
filter_chain_match
   │
   ▼
server_names
   │
   ▼
tcp_proxy
```

예

```
filter_chain_match:
  server_names:
  - api.example.com
```

### Linux에서 확인

Envoy config

```
curl localhost:9901/config_dump
```

또는

```
curl localhost:9901/listeners
```

TLS listener 확인 가능

## 8️⃣ GRPCRoute

한줄정리

```
gRPC service / method 기반 라우팅
```

gRPC는 사실 HTTP/2 위에서 동작한다.

그래서 HTTPRoute도 사용 가능하지만

```
gRPC service / method match
```

하려면 GRPCRoute를 사용한다.

### gRPC 요청 구조

gRPC는 이런 구조다.

```
/package.service/method
```

예

```
/user.UserService/GetUser
```

### GRPCRoute 전체 구조

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GRPCRoute
metadata:
spec: parentRefs
  hostnames
  rules
```

### GRPCRoute 전체 YAML 예시

```yaml
metadata:
  name: grpc-route

spec:
  parentRefs:
    - name: my-gateway

  hostnames:
    - grpc.example.com

  rules:
    - matches: # /user.UserService/GetUser 매칭
        - method:
            service: UserService
            method: GetUser

      backendRefs:
        - name: user-service
          port: 50051
          weight: 100
```

### 실제 gRPC 흐름

```
Client
   │
   ▼
HTTP/2 request
   │
   ▼
:authority = grpc.example.com
:path = /UserService/GetUser
   │
   ▼
Envoy GRPCRoute
   │
   ▼
backend service
```

### Envoy 내부 변환

Envoy는 gRPC도 HTTP filter chain 사용

```
listener
   │
   ▼
http_connection_manager
   │
   ▼
route
   │
   ▼
cluster
```

match

```
:path = /UserService/GetUser
```

### Linux에서 확인

Envoy route

```
curl localhost:9901/routes
```

## 9️⃣ UDPRoute

한 줄 정리

```
UDP 포트 기반으로 트래픽을 Service로 전달하는 L4 라우팅 규칙
```

즉

```
UDP packet
   │
   ▼
Gateway listener
   │
   ▼
UDPRoute
   │
   ▼
Service
   │
   ▼
Pod
```

```
HTTP parsing 없음
SNI 없음
header 없음

단순 UDP forwarding
```

### 언제 사용하나

UDP 기반 프로토콜 예

| 프로토콜   | 포트 |
| ---------- | ---- |
| DNS        | 53   |
| QUIC       | 443  |
| VoIP (RTP) | 다양 |
| 게임 서버  | 다양 |
| syslog     | 514  |

### UDPRoute 전체 YAML 예시

```yaml
# 아직 정식은 아닌가보다 alpha 인것을 보니
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: UDPRoute
metadata:
  name: dns-route
  namespace: default

spec:
  parentRefs:
    - name: my-gateway
      sectionName: dns
      port: 53

  rules:
    - backendRefs:
        - name: dns-service
          port: 53
          weight: 100
```

### Envoy 내부 변환

UDPRoute → Envoy

```
listener
   │
   ▼
udp_listener
   │
   ▼
udp_proxy
   │
   ▼
cluster
```

예

```
listener: 0.0.0.0:53
filter: udp_proxy
cluster: dns-service
```

## 10. 정리

### 전체적으로

| 리소스       | 역할            |
| ------------ | --------------- |
| GatewayClass | controller 종류 |
| Gateway      | LB              |
| HTTPRoute    | HTTP routing    |
| TCPRoute     | TCP routing     |

### Route 타입 전체 비교

| Route     | Layer | 특징                 |
| --------- | ----- | -------------------- |
| HTTPRoute | L7    | HTTP routing         |
| GRPCRoute | L7    | gRPC service routing |
| TCPRoute  | L4    | TCP forwarding       |
| TLSRoute  | L4    | TLS SNI routing      |
| UDPRoute  | L4    | UDP forwarding       |

### Gateway API 전체 구조

```
GatewayClass
      │
      ▼
Gateway
      │
      ▼
Routes
 ├ HTTPRoute
 ├ TCPRoute
 ├ TLSRoute
 ├ GRPCRoute
 └ UDPRoute
```

### Envoy 내부 구조

모든 Route는 결국 Envoy에서 이렇게 된다.

```
listener
   │
   ▼
filter chain
   │
   ▼
route
   │
   ▼
cluster
   │
   ▼
endpoint
```

## 11. 실제 전체 흐름

```
Client
   │
   ▼
Node iptables
   │
   ▼
Envoy Service
   │
   ▼
Envoy Pod
   │
   ▼
Route match
   │
   ▼
Service
   │
   ▼
Pod
```

## 12. Linux 네트워크 흐름

Node에서

```
iptables -t nat -L
```

보면

```
KUBE-SVC-xxxxx
```

Service → Pod NAT

패킷 확인

```
tcpdump -i any port 80
```

---

# 4장 핵심 아키텍처

전체 흐름

```
사용자
  ↓
DNS
  ↓
Node IP:80
  ↓
iptables (kube-proxy)
  ↓
envoy gateway service
  ↓
Envoy proxy (data plane)
  ↓
route table
  ↓
cluster
  ↓
endpoint
  ↓
pod
```

## 1️⃣ Envoy xDS Protocol (Control Plane 핵심)

Envoy는 자체 설정 파일을 직접 읽지 않는다. 대신 Control Plane이 설정을 push한다.

이 프로토콜을 xDS라고 한다.

```
xDS = Envoy configuration discovery protocol
```

Envoy는 이렇게 물어본다.

```
Envoy
  ↓
Control Plane
"나한테 라우팅 설정 좀 줘"
```

### xDS 종류

| 이름 | 의미               |
| ---- | ------------------ |
| LDS  | Listener Discovery |
| RDS  | Route Discovery    |
| CDS  | Cluster Discovery  |
| EDS  | Endpoint Discovery |
| SDS  | Secret Discovery   |

### 실제 흐름

```
1️⃣ Listener 요청 (LDS)

Control Plane →
"80 port listener 만들어라"

2️⃣ Route 요청 (RDS)

Control Plane →
"example.com 은 serviceA"

3️⃣ Cluster 요청 (CDS)

Control Plane →
"serviceA 라는 upstream 있음"

4️⃣ Endpoint 요청 (EDS)

Control Plane →
"pod IP 목록"
```

### 구조

```
           Control Plane
        (Envoy Gateway)
                │
                │ xDS (gRPC)
                │
        ┌───────────────┐
        │ Envoy Proxy   │
        │ (Data Plane)  │
        └───────────────┘
```

### 실제 Envoy config 내부 구조

Envoy는 내부적으로 이렇게 저장한다.

```
Listener
 └── Route
      └── Cluster
            └── Endpoint
```

예:

```
listener: 80
   route: example.com
      cluster: serviceA
          endpoints:
            - 10.244.1.3
            - 10.244.2.5
```

## 2️⃣ Envoy Cluster / Endpoint Discovery

Envoy에서 cluster = upstream service

예:

```
service: user-service
pods:
 10.244.1.2
 10.244.2.4
```

Envoy 내부:

```
cluster: user-service
endpoints:
 10.244.1.2:8080
 10.244.2.4:8080
```

### Cluster Load Balancing

Envoy는 여러 LB 알고리즘을 지원한다.

| 알고리즘      | 설명                  |
| ------------- | --------------------- |
| round_robin   | 기본                  |
| least_request | connection 적은 곳    |
| ring_hash     | consistent hash       |
| maglev        | high performance hash |

### Endpoint update

Endpoint update

```
Pod 생성
 ↓
Kubernetes endpoint 변경
 ↓
Envoy Gateway 감지
 ↓
EDS 업데이트
 ↓
Envoy에 push
```

## 3️⃣ Gateway API → Envoy Config 변환 과정

이게 Envoy Gateway의 핵심 기능이다.

만든 것:

```
GatewayClass
Gateway
HTTPRoute
```

이걸 Envoy config로 변환한다.

### 변환 흐름

```
Kubernetes API Server
        │
        │ watch
        ▼
Envoy Gateway Controller
        │
        │ translate
        ▼
Envoy xDS config
        │
        │ gRPC
        ▼
Envoy Proxy
```

실제 변환 예

```
[HTTPRoute]
host: example.com
path: /login
backend: user-service

↓

[Envoy config]
listener 80
   route example.com
      match /login
      cluster user-service
```

## 4️⃣ Envoy L4 vs L7 Routing 내부 구조

Envoy는 L4 / L7 둘 다 가능하다.

### L4 routing

```
TCPRoute
UDPRoute
TLSRoute
```

기준:

```
IP + Port
```

예

```
:3306 → mysql
:6379 → redis
```

Envoy 내부

```
listener 3306
  tcp_proxy
    cluster mysql
```

### L7 routing

```
HTTPRoute
GRPCRoute
```

기준

```
Host
Path
Header
Method
```

예

```
example.com/login → user
example.com/pay → payment
```

Envoy 내부

```
listener 80
  http_connection_manager
    route_config
```

Envoy HTTP 구조

```
Listener
  ↓
HTTP Connection Manager
  ↓
Route Table
  ↓
Cluster
```

## 5️⃣ Envoy Connection Pool / Circuit Breaker

이건 대규모 트래픽에서 핵심이다.

### Connection Pool

Envoy는 매 요청마다 connection을 새로 만들지 않는다.

```
client
 ↓
Envoy
 ↓
connection pool
 ↓
upstream
```

예

```
max connections: 100
```

Envoy가 connection 재사용한다.

장점

```
latency 감소
TCP handshake 감소
```

### Circuit Breaker

서비스 보호 기능이다.

예

```
user-service 가 터짐
```

Envoy는

```
max connections = 100
max pending requests = 1000
```

넘으면 503 반환

Envoy circuit breaker 예

```
cluster:
  name: user-service
  circuit_breakers:
    thresholds:
      max_connections: 100
      max_pending_requests: 1000
```

## 지금까지 전체 구조

```
Gateway API
 (HTTPRoute / TCPRoute)
        │
        ▼
Envoy Gateway Controller
        │
        │ translate
        ▼
Envoy xDS config
        │
        ▼
Envoy Proxy
        │
        ▼
Listener
  ↓
Route
  ↓
Cluster
  ↓
Endpoint
  ↓
Pod
```

## Envoy admin API

### 1️⃣ Envoy admin API (필수)

```
localhost:9901
```

여기서

```
/listeners
/routes
/clusters
/config_dump
```

전부 볼 수 있다.

### 2️⃣ Envoy 실제 config 확인

```
kubectl port-forward envoy-pod 9901:9901
```

### 3️⃣ iptables → Envoy 흐름

```
user
 ↓
node:80
 ↓
iptables
 ↓
envoy service
 ↓
envoy pod
```
