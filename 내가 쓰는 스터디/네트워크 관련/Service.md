## 목차

- [1장 Service 란 무엇인가](#1장-service-란-무엇인가)
  - [1️⃣ Service란 무엇인가](#1️⃣-service란-무엇인가)
    - [왜 필요한가](#왜-필요한가)
  - [2️⃣ Service 전체 구조](#2️⃣-service-전체-구조)
  - [3️⃣ Service 종류](#3️⃣-service-종류)
    - [1️⃣ ClusterIP](#1️⃣-clusterip)
    - [2️⃣ NodePort](#2️⃣-nodeport)
    - [3️⃣ LoadBalancer](#3️⃣-loadbalancer)
  - [4️⃣ Endpoint / EndpointSlice](#4️⃣-endpoint--endpointslice)
    - [Endpoint](#endpoint)
    - [EndpointSlice](#endpointslice)
  - [5️⃣ kube-proxy](#5️⃣-kube-proxy)
  - [6️⃣ iptables / ipvs](#6️⃣-iptables--ipvs)
    - [iptables mode](#iptables-mode)
    - [ipvs mode](#ipvs-mode)
  - [7️⃣ KUBE-SVC / KUBE-SEP](#7️⃣-kube-svc--kube-sep)
    - [KUBE-SVC](#kube-svc)
    - [KUBE-SEP](#kube-sep)
  - [8️⃣ 실제 패킷 흐름](#8️⃣-실제-패킷-흐름)
    - [실제 DNAT](#실제-dnat)
  - [9️⃣ Linux에서 실제 확인](#9️⃣-linux에서-실제-확인)
  - [🔟 Service 생성 실습](#-service-생성-실습)
  - [🔥 Service 핵심 개념 정리](#-service-핵심-개념-정리)
  - [⭐ Kubernetes Service 전체 흐름 (진짜 중요)](#-kubernetes-service-전체-흐름-진짜-중요)
- [2장 iptables packet flow](#2장-iptables-packet-flow)
  - [1️⃣ Linux Packet Flow (iptables 기본 구조)](#1️⃣-linux-packet-flow-iptables-기본-구조)
  - [2️⃣ kube-proxy가 만드는 iptables 구조](#2️⃣-kube-proxy가-만드는-iptables-구조)
  - [3️⃣ ClusterIP Packet Flow](#3️⃣-clusterip-packet-flow)
    - [1️⃣ OUTPUT chain](#1️⃣-output-chain)
    - [2️⃣ Service 찾기](#2️⃣-service-찾기)
    - [4️⃣ KUBE-SVC Chain](#4️⃣-kube-svc-chain)
    - [5️⃣ KUBE-SEP Chain](#5️⃣-kube-sep-chain)
    - [6️⃣ 전체 흐름 (ClusterIP)](#6️⃣-전체-흐름-clusterip)
  - [4️⃣ NodePort Packet Flow](#4️⃣-nodeport-packet-flow)
    - [kube-proxy rule](#kube-proxy-rule)
  - [5️⃣ 실제 Linux에서 보는 방법](#5️⃣-실제-linux에서-보는-방법)
    - [iptables 확인](#iptables-확인)
    - [Endpoint 확인](#endpoint-확인)
  - [6️⃣ kube-proxy 내부 동작](#6️⃣-kube-proxy-내부-동작)
  - [7️⃣ ipvs mode](#7️⃣-ipvs-mode)
  - [🔥 Service 네트워크 핵심 요약](#-service-네트워크-핵심-요약)
- [3장 kube-proxy](#3장-kube-proxy)
  - [1️⃣ kube-proxy](#1️⃣-kube-proxy)
  - [2️⃣ kube-proxy가 보는 것](#2️⃣-kube-proxy가-보는-것)
  - [3️⃣ kube-proxy iptables 구조](#3️⃣-kube-proxy-iptables-구조)
  - [4️⃣ 실제 iptables rule 분석](#4️⃣-실제-iptables-rule-분석)
  - [5️⃣ Service 생성 → iptables 생성 추적 실습](#5️⃣-service-생성--iptables-생성-추적-실습)
    - [Step1](#step1)
    - [Step2](#step2)
    - [Step3](#step3)
  - [6️⃣ Pod 생성 → Endpoint 추가 → iptables 변경](#6️⃣-pod-생성--endpoint-추가--iptables-변경)
  - [7️⃣ Pod 삭제 → iptables 변경](#7️⃣-pod-삭제--iptables-변경)
  - [8️⃣ Service 삭제 → iptables 삭제](#8️⃣-service-삭제--iptables-삭제)
  - [9️⃣ kube-proxy 내부 동작](#9️⃣-kube-proxy-내부-동작)
  - [🔟 kube-proxy iptables 생성 알고리즘](#-kube-proxy-iptables-생성-알고리즘)
    - [실제 Linux에서 추적하는 방법](#실제-linux에서-추적하는-방법)
      - [kube-proxy 로그](#kube-proxy-로그)
      - [iptables 변화 보기](#iptables-변화-보기)
      - [특정 service 찾기](#특정-service-찾기)
      - [Endpoint chain 찾기](#endpoint-chain-찾기)
    - [🔥 진짜 Kubernetes 네트워크 디버깅 방법](#-진짜-kubernetes-네트워크-디버깅-방법)
    - [⭐ kube-proxy rule 흐름 (완전 중요)](#-kube-proxy-rule-흐름-완전-중요)
    - [🔥 가장 중요한 kube-proxy iptables 구조](#-가장-중요한-kube-proxy-iptables-구조)
- [4장 (로컬) MetalLB](#4장-로컬-metallb)
  - [1️⃣ MetalLB](#1️⃣-metallb)
  - [2️⃣ MetalLB 구조](#2️⃣-metallb-구조)
    - [Layer2 Mode 동작](#layer2-mode-동작)
  - [3️⃣ MetalLB IP 할당 과정](#3️⃣-metallb-ip-할당-과정)
  - [4️⃣ 외부 → LoadBalancer → Pod 패킷 흐름](#4️⃣-외부--loadbalancer--pod-패킷-흐름)
    - [Step 1](#step-1)
    - [Step 2](#step-2)
    - [Step 3](#step-3)
    - [Step 4](#step-4)
    - [Step 5](#step-5)
- [5장 MetalLB L2 mode ARP](#5장-metallb-l2-mode-arp)
  - [1️⃣ MetalLB L2 Mode 구조](#1️⃣-metallb-l2-mode-구조)
  - [2️⃣ ARP 패킷 흐름 (실제)](#2️⃣-arp-패킷-흐름-실제)
    - [Client가 요청](#client가-요청)
    - [MetalLB Speaker 응답](#metallb-speaker-응답)
  - [3️⃣ tcpdump로 ARP 확인](#3️⃣-tcpdump로-arp-확인)
  - [4️⃣ ARP 이후 패킷 흐름](#4️⃣-arp-이후-패킷-흐름)
- [5장 ExternalTrafficPolicy](#5장-externaltrafficpolicy)
  - [1⃣ ExternalTrafficPolicy 를 알아보자](#1⃣-externaltrafficpolicy-를-알아보자)
  - [2⃣ ExternalTrafficPolicy = Cluster](#2⃣-externaltrafficpolicy--cluster)
    - [실제 패킷](#실제-패킷)
    - [Cluster 모드 흐름](#cluster-모드-흐름)
  - [3⃣ ExternalTrafficPolicy = Local](#3⃣-externaltrafficpolicy--local)
    - [Local 모드 패킷 흐름](#local-모드-패킷-흐름)
  - [4⃣ ExternalTrafficPolicy 표](#4⃣-externaltrafficpolicy-표)
- [6장 Pod → Service → 다른 Node Pod](#6장-pod--service--다른-node-pod)
  - [패킷 흐름](#패킷-흐름)
    - [Step1](#step1-1)
    - [Step2](#step2-1)
    - [Step3](#step3-1)
    - [Step4](#step4)
    - [Step5](#step5)
    - [Step6](#step6)
    - [Step7](#step7)
    - [Step8](#step8)
    - [Step9](#step9)
    - [전체 흐름](#전체-흐름)
  - [실제 Linux에서 추적](#실제-linux에서-추적)
- [6장 EXTERNAL-IP는 왜 필요한것인가](#6장-external-ip는-왜-필요한것인가)
  - [1️⃣ Kubernetes Service IP 종류](#1️⃣-kubernetes-service-ip-종류)
  - [2️⃣ ClusterIP는 외부에서 접근 불가능](#2️⃣-clusterip는-외부에서-접근-불가능)
  - [3️⃣ EXTERNAL-IP의 역할](#3️⃣-external-ip의-역할)
  - [4️⃣ MetalLB에서 EXTERNAL-IP](#4️⃣-metallb에서-external-ip)
  - [5️⃣ 패킷 흐름](#5️⃣-패킷-흐름)
  - [6️⃣ EXTERNAL-IP가 없다면?](#6️⃣-external-ip가-없다면)
  - [7️⃣ NodePort vs EXTERNAL-IP](#7️⃣-nodeport-vs-external-ip)
    - [NodePort](#nodeport)
    - [EXTERNAL-IP](#external-ip)
  - [8️⃣ 실제 iptables](#8️⃣-실제-iptables)
  - [9️⃣ Kubernetes 네트워크 관점](#9️⃣-kubernetes-네트워크-관점)
  - [🔟 전체 구조](#-전체-구조)
  - [🔥 핵심 요약](#-핵심-요약)
  - [⭐ 하나 더 중요한 사실 (많은 사람들이 모름)](#-하나-더-중요한-사실-많은-사람들이-모름)
- [궁금한것 : 일반사용자는 어떻게 접속하는가?](#궁금한것--일반사용자는-어떻게-접속하는가)
  - [1️⃣ 현재 네트워크 구조](#1️⃣-현재-네트워크-구조)
  - [2️⃣ 사용자가 브라우저에서 접속](#2️⃣-사용자가-브라우저에서-접속)
  - [3️⃣ ARP Broadcast 발생](#3️⃣-arp-broadcast-발생)
  - [4️⃣ MetalLB Speaker가 응답](#4️⃣-metallb-speaker가-응답)
  - [5️⃣ 사용자 컴퓨터 ARP 캐시 저장](#5️⃣-사용자-컴퓨터-arp-캐시-저장)
  - [6️⃣ 이제 실제 TCP 요청](#6️⃣-이제-실제-tcp-요청)
  - [7️⃣ node2에서 kube-proxy 동작](#7️⃣-node2에서-kube-proxy-동작)
  - [8️⃣ 핵심 포인트](#8️⃣-핵심-포인트)
  - [9️⃣ 그래서 브라우저 접속 가능](#9️⃣-그래서-브라우저-접속-가능)
  - [🔟 직접 확인 실험 (강력 추천)](#-직접-확인-실험-강력-추천)

---

# 1장 Service 란 무엇인가

## 1️⃣ Service란 무엇인가

🔹 한 줄 정리

Pod의 IP가 바뀌어도 안정적으로 접근할 수 있도록 만들어주는 “가상 IP + 로드밸런싱 시스템”

### 왜 필요한가

Pod 특징

```
Pod IP는 영구적이지 않음
Pod는 언제든지 재생성됨

[예]
pod1 10.244.1.5 [terminated] -> 10.244.1.9
pod2 10.244.2.7

[문제]
client → 10.244.1.5

이 주소는 깨진다.
```

그래서 Service가 중간에 가상 IP를 만든다

```
Client
   ↓
Service (Virtual IP)
   ↓
Pod1
Pod2
Pod3
```

## 2️⃣ Service 전체 구조

Service는 실제로 여러 컴포넌트가 협력해서 동작한다

전체 구조

```
Client
   ↓
Service IP (ClusterIP)
   ↓
kube-proxy
   ↓
iptables / ipvs rule
   ↓
Endpoint
   ↓
Pod
```

구성요소

```
Service (virtual IP)
Endpoint / EndpointSlice (Pod 목록)
kube-proxy (iptables rule 생성)
iptables / ipvs (실제 packet forwarding)
```

## 3️⃣ Service 종류

### 1️⃣ ClusterIP

한줄 정리

클러스터 내부에서만 접근 가능한 Service

구조

```
Pod
 ↓
Service (ClusterIP)
 ↓
Pod
```

예

```
10.96.0.10
```

이 IP는 virtual IP로 실제로 존재하지 않는다.

YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  type: ClusterIP
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
```

흐름

```
Client Pod
↓
10.96.0.10:80
↓
kube-proxy iptables
↓
Pod1 10.244.1.5:8080
Pod2 10.244.2.7:8080
```

### 2️⃣ NodePort

Node의 특정 port를 열어서 외부에서 접근 가능

구조

```
Internet
  ↓
NodeIP:NodePort
  ↓
ClusterIP
  ↓
Pod
```

예

```
NodeIP:30080
```

YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  type: NodePort
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
```

실제 흐름

```
Client
↓
192.168.0.10:30080
↓
iptables
↓
ClusterIP
↓
Pod
```

NodePort range

기본

```
30000 - 32767
```

확인

```
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

옵션

```
--service-node-port-range
```

### 3️⃣ LoadBalancer

클라우드 LoadBalancer와 연결되는 Service

예시로

AWS / GCP

```
Internet
 ↓
Cloud LoadBalancer
 ↓
NodePort
 ↓
Pod
```

실제 Kubernetes 내부

```
LoadBalancer
↓
NodePort
↓
ClusterIP
↓
Pod
```

즉

```
LoadBalancer = NodePort + external LB
```

## 4️⃣ Endpoint / EndpointSlice

### Endpoint

Service가 연결할 Pod들의 실제 IP 목록

예

```
Service
  ↓
Endpoints
   ├ Pod1 10.244.1.5
   ├ Pod2 10.244.1.6
   └ Pod3 10.244.2.3
```

확인

```
kubectl get endpoints
또는
kubectl describe svc myservice
```

실제 저장 위치

```
etcd
또는
API object

/registry/services/endpoints/
```

### EndpointSlice

왜 생겼났을까

```
Endpoint 문제로 예시를 들면
Pod 10000개라면 Endpoint object 하나가 너무 커짐.

그래서 분할했다.
```

EndpointSlice

```
EndpointSlice1
EndpointSlice2
EndpointSlice3

각 slice

100 endpoints
```

확인

```
kubectl get endpointslices
```

## 5️⃣ kube-proxy

Service → Pod 트래픽을 연결하는 iptables/ipvs 규칙을 만드는 컴포넌트

어디서 실행되나

```
Node

각 Node마다 실행
```

확인

```
kubectl get pods -n kube-system
```

역할

```
kube-proxy는

watch Service
watch Endpoint

그리고

iptables rule 생성
```

## 6️⃣ iptables / ipvs

Service 트래픽을 실제로 전달하는건 Linux kernel이다.

```
kube-proxy는 iptables rule 생성만 한다
```

### iptables mode

예

```
➜  iptables -t nat -L

KUBE-SERVICES
KUBE-SVC-xxxxx
KUBE-SEP-xxxxx
```

### ipvs mode

더 고성능

확인

```
ipvsadm -L
```

## 7️⃣ KUBE-SVC / KUBE-SEP

이게 Service 로드밸런싱 핵심

### KUBE-SVC

Service 로드밸런싱 체인

예

```
KUBE-SVC-ABCD
```

역할

```
Pod 선택
```

### KUBE-SEP

특정 Pod endpoint로 보내는 체인

예

```
KUBE-SEP-XYZ
```

실제 iptables 구조

```
KUBE-SERVICES
   ↓
KUBE-SVC-1234
   ↓
KUBE-SEP-A
KUBE-SEP-B
KUBE-SEP-C
```

## 8️⃣ 실제 패킷 흐름

예

```
curl 10.96.0.10
```

흐름

```
Client Pod
↓
Service IP
↓
iptables PREROUTING
↓
KUBE-SERVICES
↓
KUBE-SVC-xxxx
↓
random load balance
↓
KUBE-SEP-xxxx
↓
DNAT
↓
Pod IP
```

### 실제 DNAT

iptables rule 에 따른다.

```
DNAT 10.96.0.10:80
→
10.244.1.5:8080
```

## 9️⃣ Linux에서 실제 확인

Service 확인

```
kubectl get svc
```

Endpoint

```
kubectl get endpoints
```

EndpointSlice

```
kubectl get endpointslices
```

kube-proxy

```
kubectl get pods -n kube-system
```

iptables 확인

```
iptables -t nat -L -n
```

kube chain

```
KUBE-SERVICES
KUBE-NODEPORTS
KUBE-SVC
KUBE-SEP
```

특정 service 찾기

```
iptables-save | grep KUBE
```

## 🔟 Service 생성 실습

1️⃣ Pod 생성

```
kubectl run nginx --image=nginx
```

2️⃣ Service 생성

```
kubectl expose pod nginx --port=80
```

3️⃣ 확인

```
kubectl get svc
```

4️⃣ endpoint 확인

```
kubectl get endpoints
```

5️⃣ iptables 확인

```
iptables -t nat -L
```

```
➜ iptables -t nat -L PREROUTING
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
KUBE-SERVICES  all  --  anywhere             anywhere             /* kubernetes service portals */
```

```
➜ iptables -t nat -L KUBE-SERVICES
Chain KUBE-SERVICES (2 references)
target     prot opt source               destination
...
KUBE-SVC-2CMXP7HKUVJN7L6M  tcp  --  anywhere             10.109.158.83        /* default/nginx cluster IP */
...
```

```
➜ iptables -t nat -L KUBE-SVC-2CMXP7HKUVJN7L6M
Chain KUBE-SVC-2CMXP7HKUVJN7L6M (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  tcp  -- !10.244.0.0/16        10.109.158.83        /* default/nginx cluster IP */
KUBE-SEP-YV43TXGDTRSKWB44  all  --  anywhere             anywhere             /* default/nginx -> 10.244.71.40:80 */
```

```
➜ iptables -t nat -L KUBE-SEP-YV43TXGDTRSKWB44
Chain KUBE-SEP-YV43TXGDTRSKWB44 (1 references)
target     prot opt source               destination
KUBE-MARK-MASQ  all  --  10.244.71.40         anywhere             /* default/nginx */
DNAT       tcp  --  anywhere             anywhere             /* default/nginx */ tcp to:10.244.71.40:80
```

## 🔥 Service 핵심 개념 정리

Service는 실제 서버가 아니다

```
virtual IP
```

트래픽 전달은 누가?

```
kube-proxy
+
iptables / ipvs
```

Pod 목록은 어디?

```
Endpoint
EndpointSlice
```

로드밸런싱

```
iptables random
```

## ⭐ Kubernetes Service 전체 흐름 (진짜 중요)

```
Pod 생성
 ↓
Endpoint 생성
 ↓
Service 생성
 ↓
kube-proxy 감지
 ↓
iptables rule 생성
 ↓
Client → Service IP
 ↓
iptables
 ↓
Pod
```

---

# 2장 iptables packet flow

## 1️⃣ Linux Packet Flow (iptables 기본 구조)

먼저 쿠버네티스가 아니라 리눅스 기본 구조를 알아야 한다.

패킷 흐름

```
Network
   ↓
PREROUTING
   ↓
Routing decision
   ↓
INPUT → local process
FORWARD → 다른 interface
OUTPUT → local process
   ↓
POSTROUTING
```

iptables NAT table

```
PREROUTING
OUTPUT
POSTROUTING
```

쿠버네티스 Service는 여기서 DNAT을 사용한다. 즉,

```
Service IP
↓
Pod IP 로 변경
```

## 2️⃣ kube-proxy가 만드는 iptables 구조

kube-proxy는 iptables chain을 생성한다.

확인

```
iptables -t nat -L
```

보면 이런 chain들이 생긴다.

```
KUBE-SERVICES
KUBE-NODEPORTS
KUBE-SVC-XXXXX
KUBE-SEP-XXXXX
```

구조

```
PREROUTING
   ↓
KUBE-SERVICES
   ↓
KUBE-SVC-XXXXX
   ↓
KUBE-SEP-XXXXX
```

## 3️⃣ ClusterIP Packet Flow

예를 들어 보자.

Service

```
Service IP: 10.96.0.10
Port: 80
```

Pod

```
Pod1 10.244.1.5:8080
Pod2 10.244.2.7:8080
```

패킷 발생

```
Pod에서 요청

curl 10.96.0.10:80
```

### 1️⃣ OUTPUT chain

Pod → Service

```
OUTPUT
 ↓
KUBE-SERVICES
```

### 2️⃣ Service 찾기

iptables rule

```
-A KUBE-SERVICES \
-d 10.96.0.10/32 \
-p tcp \
--dport 80 \
-j KUBE-SVC-ABCD
```

의미

```
Service IP이면
KUBE-SVC chain으로 이동
```

### 4️⃣ KUBE-SVC Chain

여기서 로드밸런싱이 이루어진다.

예

```
KUBE-SVC-ABCD

-A KUBE-SVC-ABCD -m statistic --mode random --probability 0.5 -j KUBE-SEP-AAAA
-A KUBE-SVC-ABCD -j KUBE-SEP-BBBB
```

의미

```
50% → Pod1
50% → Pod2
```

### 5️⃣ KUBE-SEP Chain

```
SEP = Service Endpoint
```

각 Pod마다 하나씩 존재한다.

예

```
KUBE-SEP-AAAA
```

iptables rule

```
-A KUBE-SEP-AAAA \
-j DNAT --to-destination 10.244.1.5:8080
```

즉

```
ServiceIP → PodIP
```

으로 변환한다.

### 6️⃣ 전체 흐름 (ClusterIP)

```
Pod
 ↓
10.96.0.10:80
 ↓
OUTPUT
 ↓
KUBE-SERVICES
 ↓
KUBE-SVC-ABCD
 ↓
KUBE-SEP-AAAA
 ↓
DNAT
 ↓
10.244.1.5:8080
 ↓
Pod
```

```
즉, Service IP는 실제 존재하지 않는다
iptables가 IP를 바꿔준다.
```

## 4️⃣ NodePort Packet Flow

이번엔 외부에서 NodePort 접속

예

```
NodeIP:30080
```

외부 요청

```
curl 192.168.0.10:30080
```

### kube-proxy rule

iptables

```
PREROUTING
 ↓
KUBE-NODEPORTS
```

rule

```
-A KUBE-NODEPORTS \
-p tcp \
--dport 30080 \
-j KUBE-SVC-ABCD
```

이후 흐름

```
External
 ↓
NodePort
 ↓
KUBE-NODEPORTS
 ↓
KUBE-SVC
 ↓
KUBE-SEP
 ↓
DNAT
 ↓
Pod
```

## 5️⃣ 실제 Linux에서 보는 방법

Service 생성

```
kubectl create deployment nginx --image=nginx
```

Service 생성

```
kubectl expose deployment nginx --port=80
```

확인

```
kubectl get svc
```

예

```
10.96.15.120
```

### iptables 확인

```
iptables -t nat -L -n
또는
iptables-save
```

특정 service 찾기

```
➜  iptables-save | grep 10.96

-A KUBE-SERVICES \
-d 10.96.15.120/32 \
-p tcp \
--dport 80 \
-j KUBE-SVC-XYZ
```

### Endpoint 확인

```
➜  kubectl get endpoints

10.244.1.5
10.244.2.7
```

## 6️⃣ kube-proxy 내부 동작

kube-proxy는 watcher이다.

```
[watch]

Service
Endpoint
EndpointSlice
```

변경 발생하면

```
iptables rule regenerate
```

kube-proxy 실행 위치

```
Node
```

확인

```
kubectl -n kube-system get pods
```

로그

```
kubectl logs kube-proxy-xxxxx -n kube-system
```

## 7️⃣ ipvs mode

iptables 문제

```
서비스 10000개

iptables rule 10000 + A 개가 되어
성능 문제가 크다.

그래서 나온 것이 IPVS이다.
```

확인

```
ipvsadm -L
```

IPVS 구조

```
Service IP
 ↓
IPVS table
 ↓
Pod
```

## 🔥 Service 네트워크 핵심 요약

Service는 실제 IP가 아니며 (virtual IP) 실제 forwarding 은

```
iptables
or
ipvs
```

이 두개가 한다.

kube-proxy 역할

```
iptables rule 생성
```

endpoint

```
Pod 목록
```

로드밸런싱

```
iptables random
```

⭐ 진짜 중요한 Kubernetes Service 흐름

```
Pod 생성
 ↓
Endpoint 생성
 ↓
Service 생성
 ↓
kube-proxy 감지
 ↓
iptables rule 생성
 ↓
Client → ServiceIP
 ↓
iptables
 ↓
Pod
```

---

# 3장 kube-proxy

## 1️⃣ kube-proxy

🔹 한 줄 정리

Service 트래픽을 Pod로 전달하기 위해 iptables/ipvs 규칙을 생성하는 Node 컴포넌트

어디서 실행되는가

```
Node마다 하나씩 실행
```

확인

```
kubectl get pods -n kube-system -o wide
```

예

```
kube-proxy-x8sd2
```

DaemonSet 형태

```
kubectl get daemonset -n kube-system
```

## 2️⃣ kube-proxy가 보는 것

kube-proxy는 watcher다.

watch 대상

```
Service
Endpoint
EndpointSlice
Node
```

즉

```
Service 변경
Pod 변경
```

이 발생하면

```
iptables rule 재생성
```

## 3️⃣ kube-proxy iptables 구조

kube-proxy가 만드는 주요 chain

```
KUBE-SERVICES
KUBE-NODEPORTS
KUBE-SVC-xxxxx
KUBE-SEP-xxxxx
KUBE-MARK-MASQ
```

구조

```
PREROUTING
   ↓
KUBE-SERVICES
   ↓
KUBE-SVC
   ↓
KUBE-SEP
   ↓
DNAT
```

## 4️⃣ 실제 iptables rule 분석

Service

```
ClusterIP: 10.96.0.10
port: 80
```

Pod

```
10.244.1.5
10.244.2.7

```

rule 1

```
-A KUBE-SERVICES \
-d 10.96.0.10/32 \
-p tcp \
--dport 80 \
-j KUBE-SVC-ABCD
```

해석

```
destination = 10.96.0.10
port = 80
```

이면

```
KUBE-SVC-ABCD로 이동
```

rule 1

```
-A KUBE-SERVICES \
-d 10.96.0.10/32 \
-p tcp \
--dport 80 \
-j KUBE-SVC-ABCD
```

해석

```
destination = 10.96.0.10
port = 80

이면
KUBE-SVC-ABCD로 이동
```

rule 2

```
-A KUBE-SVC-ABCD \
-m statistic --mode random --probability 0.5 \
-j KUBE-SEP-AAAA
```

해석

```
50% 확률로 Pod1
```

rule 3

```
-A KUBE-SVC-ABCD \
-j KUBE-SEP-BBBB
```

해석

```
나머지 Pod2
```

rule 4

```
-A KUBE-SEP-AAAA \
-j DNAT --to-destination 10.244.1.5:8080
```

해석

```
Pod1으로 DNAT
```

## 5️⃣ Service 생성 → iptables 생성 추적 실습

### Step1

Deployment 생성

```
kubectl create deployment nginx --image=nginx
```

### Step2

Service 생성

```
kubectl expose deployment nginx --port=80
```

### Step3

iptables 확인

```
iptables-save | grep KUBE
```

새로 생기는 것

```
KUBE-SVC-XXXXX
KUBE-SEP-XXXXX
```

kube-proxy 로그

```
kubectl logs -n kube-system kube-proxy-xxxxx
```

로그 예

```
Syncing iptables rules
Adding new service
```

## 6️⃣ Pod 생성 → Endpoint 추가 → iptables 변경

```
Pod가 늘어나면 Endpoint 변경
```

예

```
kubectl scale deployment nginx --replicas=3
```

확인

```
kubectl get endpoints
```

예

```
10.244.1.5
10.244.2.7
10.244.3.8
```

iptables 확인

```
iptables-save | grep KUBE-SEP
```

새 rule 생성

```
KUBE-SEP-CCCC
```

## 7️⃣ Pod 삭제 → iptables 변경

Pod 삭제

```
kubectl delete pod nginx-xxxxx
```

Endpoint 변경

```
kubectl get endpoints
```

iptables rule 변경

```
iptables-save
```

삭제된 것

```
KUBE-SEP-AAAA
```

## 8️⃣ Service 삭제 → iptables 삭제

Service 삭제

```
kubectl delete svc nginx
```

iptables 확인

```
iptables-save | grep KUBE
```

삭제되는 것

```
KUBE-SVC-XXXXX
KUBE-SEP-XXXXX
```

## 9️⃣ kube-proxy 내부 동작

kube-proxy 내부 구조

```
API Server watch
     ↓
Service cache
Endpoint cache
     ↓
sync loop
     ↓
iptables regenerate
```

코드 흐름

```
syncProxyRules()
```

이 함수가 iptables rule 생성

## 🔟 kube-proxy iptables 생성 알고리즘

중요 특징

```
kube-proxy는 incremental update가 아니라
전체 재계산이다.

즉

Service list
Endpoint list

를 읽고 iptables rule 전체 rebuild한다.
```

### 실제 Linux에서 추적하는 방법

#### kube-proxy 로그

```
kubectl logs -n kube-system kube-proxy-xxxx
```

#### iptables 변화 보기

추천

```
watch -n1 iptables-save
```

#### 특정 service 찾기

```
iptables-save | grep <ClusterIP>
```

#### Endpoint chain 찾기

```
iptables-save | grep SEP
```

### 🔥 진짜 Kubernetes 네트워크 디버깅 방법

이 순서로 본다

1️⃣ Service 확인

```
kubectl get svc
```

2️⃣ Endpoint 확인

```
kubectl get endpoints
```

3️⃣ iptables 확인

```
iptables-save
```

4️⃣ kube-proxy 확인

```
kubectl logs kube-proxy
```

### ⭐ kube-proxy rule 흐름 (완전 중요)

```
Service 생성
 ↓
API server
 ↓
Endpoint controller
 ↓
Endpoint 생성
 ↓
kube-proxy watch
 ↓
iptables rule 생성
```

### 🔥 가장 중요한 kube-proxy iptables 구조

```
PREROUTING
   ↓
KUBE-SERVICES
   ↓
KUBE-SVC
   ↓
KUBE-SEP
   ↓
DNAT
```

---

# 4장 (로컬) MetalLB

## 1️⃣ MetalLB

클라우드 LoadBalancer가 없는 환경에서 LoadBalancer Service를 구현하는 Kubernetes 컴포넌트

보통 클라우드에서는

```
AWS ELB
GCP LB
Azure LB
```

가 자동으로 생성된다.

하지만 온프레미스 / 로컬 클러스터에서는 없다.

그래서 등장한 것이

```
MetalLB
```

구성

```
MetalLB Controller
MetalLB Speaker
```

## 2️⃣ MetalLB 구조

MetalLB는 두 가지 모드가 있다.

```
Layer2 mode
BGP mode
```

대부분 로컬에서는

```
Layer2 mode
```

를 사용한다.

```
➜  metallb k get all -n metallb-system
NAME                                      READY   STATUS    RESTARTS   AGE
pod/metallb-controller-68b4f55f99-mz7hs   1/1     Running   0          2m25s
pod/metallb-speaker-8zsdz                 4/4     Running   0          2m25s

NAME                              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/metallb-webhook-service   ClusterIP   10.111.114.18   <none>        443/TCP   2m26s

NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/metallb-speaker   1         1         1       1            1           kubernetes.io/os=linux   2m26s

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/metallb-controller   1/1     1            1           2m25s

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/metallb-controller-68b4f55f99   1         1         1       2m25s
```

### Layer2 Mode 동작

MetalLB Speaker가 ARP 응답을 한다.

예

```
LoadBalancer IP = 192.168.0.240
```

Client가

```
ARP who has 192.168.0.240
```

하면 Speaker Node가 응답

즉

```
LoadBalancer IP → 특정 Node
```

로 트래픽이 들어온다.

## 3️⃣ MetalLB IP 할당 과정

Service 생성

```
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  type: LoadBalancer
```

흐름

```
Service 생성
  ↓
API Server
  ↓
MetalLB Controller 감지
  ↓
IP Pool에서 IP 할당
  ↓
Service status 업데이트
```

확인

```
kubectl get svc
```

예

```
EXTERNAL-IP 192.168.0.240
```

## 4️⃣ 외부 → LoadBalancer → Pod 패킷 흐름

예

```
LoadBalancer IP = 192.168.0.240
```

Client

```
curl 192.168.0.240
```

### Step 1

ARP

```
who has 192.168.0.240
```

Speaker Node 응답

```
MAC address
```

### Step 2

패킷 도착

```
Client
  ↓
Node (MetalLB speaker)
```

### Step 3

iptables

```
PREROUTING
  ↓
KUBE-SERVICES
```

rule

```
-d 192.168.0.240
→
KUBE-SVC
```

### Step 4

Service chain

```
KUBE-SVC
```

로드밸런싱

```
Pod1
Pod2
Pod3
```

### Step 5

KUBE-SEP

```
DNAT
```

예

```
192.168.0.240:80
→
10.244.2.5:8080
```

---

# 5장 MetalLB L2 mode ARP

## 1️⃣ MetalLB L2 Mode 구조

MetalLB L2 mode는 LoadBalancer IP를 특정 Node가 “소유한 것처럼” ARP 응답한다.

예

```
Service
type: LoadBalancer

EXTERNAL-IP
192.168.0.240
```

클러스터

```
Node1 192.168.0.10
Node2 192.168.0.11
Node3 192.168.0.12
```

MetalLB Speaker는 한 Node를 leader로 선택 예

```
Node2 가 leader
```

## 2️⃣ ARP 패킷 흐름 (실제)

### Client가 요청

```
curl 192.168.0.240
```

먼저 ARP 발생

```
Who has 192.168.0.240?
Tell 192.168.0.100
```

ARP Request

```
src MAC = client
dst MAC = ff:ff:ff:ff:ff:ff
```

네트워크 전체 broadcast

### MetalLB Speaker 응답

Leader Node가 응답

```
ARP Reply

192.168.0.240 is-at aa:bb:cc:dd:ee:ff
```

여기서

```
aa:bb:cc:dd:ee:ff
```

는 Node2 NIC MAC

즉

```
LoadBalancer IP → Node2 MAC
```

으로 매핑된다.

## 3️⃣ tcpdump로 ARP 확인

Node에서 확인

```
tcpdump -i eth0 arp
```

예

```
ARP, Request who-has 192.168.0.240 tell 192.168.0.100
ARP, Reply 192.168.0.240 is-at aa:bb:cc:dd:ee:ff
```

이 Reply가 바로 MetalLB Speaker이다.

## 4️⃣ ARP 이후 패킷 흐름

ARP 이후 Client는

```
dst IP = 192.168.0.240
dst MAC = Node2
```

패킷 전송

```
Client
   ↓
Node2 (MetalLB leader)
```

여기서 kube-proxy가 동작한다.

---

# 5장 ExternalTrafficPolicy

## 1⃣ ExternalTrafficPolicy 를 알아보자

Service 예

```
apiVersion: v1
kind: Service
spec:
  type: LoadBalancer
  externalTrafficPolicy: Cluster
```

또는

```
externalTrafficPolicy: Local
```

이 옵션이 패킷 흐름을 완전히 바꾼다.

## 2⃣ ExternalTrafficPolicy = Cluster

기본값이다.

구조

```
Client
 ↓
MetalLB Node
 ↓
kube-proxy
 ↓
ANY Pod (다른 Node 포함)
```

패킷 흐름

```
Client
 ↓
Node2
 ↓
iptables
 ↓
DNAT
 ↓
PodB (Node3)
```

즉

```
Node2 → Node3
```

로 forwarding 발생

### 실제 패킷

```
src = ClientIP
dst = PodIP
```

그러나 문제

```
SNAT 발생
```

iptables rule

```
KUBE-MARK-MASQ
```

결과

```
Pod sees source = Node2
```

즉

```
Client IP lost
```

### Cluster 모드 흐름

```
Client
 ↓
MetalLB Node
 ↓
kube-proxy
 ↓
DNAT
 ↓
다른 Node Pod
```

장점

```
Load balancing across cluster
```

단점

```
Client IP lost
```

## 3⃣ ExternalTrafficPolicy = Local

이 모드는 완전히 다르다.

목적

```
Client IP preserve
```

구조

```
Client
 ↓
Node with Pod only
 ↓
Pod
```

즉

```
Pod 없는 Node는 트래픽 drop
```

예

```
Node1 PodA
Node2 (no pod) [MetalLB leader]
Node3 PodB
```

```
하지만 Node2에는 Pod 없음
결과: iptables drop
```

그래서 MetalLB는 보통 Pod 있는 Node로 ARP를 조정한다.

### Local 모드 패킷 흐름

```
Client
 ↓
Node1
 ↓
PodA
```

Pod sees

```
src = Client IP
```

즉

```
SNAT 없음
```

## 4⃣ ExternalTrafficPolicy 표

| 항목                  | Cluster   | Local         |
| --------------------- | --------- | ------------- |
| Load balance          | 모든 Node | Pod 있는 Node |
| Client IP             | ❌ lost   | ✅ preserved  |
| Cross node forwarding | 있음      | 없음          |
| latency               | 조금 높음 | 낮음          |

---

# 6장 Pod → Service → 다른 Node Pod

이게 진짜 중요하다.

예

```

Node1
Pod A

Node2
Pod B

```

Service

```

ClusterIP 10.96.0.10

```

Pod A

```

curl 10.96.0.10

```

## 패킷 흐름

### Step1

Pod A 패킷 생성

```

src = 10.244.1.5
dst = 10.96.0.10

```

### Step2

iptables OUTPUT

```

OUTPUT
↓
KUBE-SERVICES

```

### Step3

Service chain

```

KUBE-SVC

```

로드밸런싱 선택

```

Pod B

```

### Step4

DNAT

```

10.96.0.10
→
10.244.2.7

```

### Step5

Routing (커널 라우팅)

```

10.244.2.7

```

이 Pod는 다른 Node에 있음

### Step6

```

Calico 처리

```

Calico는 Pod CIDR기반 라우팅 한다.

예

```

10.244.2.0/24 (Node2)

```

### Step7

Encapsulation

```

Calico IPIP or VXLAN

```

예

```

outer src = node1
outer dst = node2
inner src = podA
inner dst = podB

```

### Step8

Node2 도착

```

decapsulation

```

### Step9

veth

```

PodB

```

도착

### 전체 흐름

```

PodA
↓
Service IP
↓
iptables
↓
DNAT
↓
PodB IP
↓
routing
↓
Calico encapsulation
↓
Node2
↓
decap
↓
PodB

```

## 실제 Linux에서 추적

Service 확인

```

kubectl get svc

```

endpoint 확인

```

kubectl get endpoints

```

iptables

```

iptables -t nat -L

```

MetalLB

```

kubectl get pods -n metallb-system

```

ARP 확인

```

arp -an

```

routing

```

➜ ip route

10.244.2.0/24 via 192.168.0.11

```

Calico interface

```

➜ ip link

vxlan.calico
tunl0

```

tcpdump 추적

Node1

```

tcpdump -i any host 10.244.2.7

```

VXLAN

```

tcpdump -i vxlan.calico

```

---

# 6장 EXTERNAL-IP는 왜 필요한것인가

## 1️⃣ Kubernetes Service IP 종류

Service에는 보통 3개의 접근 방식이 있다.

| 타입         | 접근 위치     | IP          |
| ------------ | ------------- | ----------- |
| ClusterIP    | 클러스터 내부 | 10.x.x.x    |
| NodePort     | Node IP       | nodeIP:port |
| LoadBalancer | 외부          | EXTERNAL-IP |

예

```
kubectl get svc
```

예시

```
NAME   TYPE           CLUSTER-IP     EXTERNAL-IP      PORT
web    LoadBalancer   10.96.120.10   192.168.0.240    80
```

의미

```
ClusterIP   = 내부 접근
ExternalIP  = 외부 접근
```

## 2️⃣ ClusterIP는 외부에서 접근 불가능

ClusterIP 예

```
10.96.0.10
```

이 IP는 Service CIDR에 속한다.

보통 10.96.0.0/12 이 네트워크는 Kubernetes 내부 virtual network이다.

그래서 외부에서는

```
curl 10.96.0.10
```

하면 접근 불가능

## 3️⃣ EXTERNAL-IP의 역할

EXTERNAL-IP는

```
외부 → Kubernetes Service
```

로 트래픽을 보내기 위한 entry point다.

흐름

```
Client
  ↓
EXTERNAL-IP
  ↓
Node
  ↓
kube-proxy
  ↓
Pod
```

즉

```
클러스터 외부 → 내부 서비스 연결
```

이다.

## 4️⃣ MetalLB에서 EXTERNAL-IP

클라우드에서는

```
AWS ELB
GCP LB
Azure LB
```

가 이 역할을 한다.

하지만 로컬에서는 없기 때문에

```
MetalLB
```

가 EXTERNAL-IP를 할당한다.

예

```
EXTERNAL-IP
192.168.0.240
```

MetalLB는 ARP로 이 IP를 Node에 연결한다.

## 5️⃣ 패킷 흐름

Client

```
curl 192.168.0.240
```

흐름

```
Client
 ↓
ARP
 ↓
MetalLB Speaker Node
 ↓
iptables
 ↓
Service
 ↓
Pod
```

즉

```
EXTERNAL-IP → Node → Service
```

## 6️⃣ EXTERNAL-IP가 없다면?

외부에서 접근하려면

```
NodePort 사용
```

예

```
curl 192.168.0.10:30080
```

문제

```
포트 기억해야 함
Node마다 다름
로드밸런싱 없음
```

그래서

```
LoadBalancer Service
```

를 사용한다.

## 7️⃣ NodePort vs EXTERNAL-IP

### NodePort

접근

```
nodeIP:nodePort
```

예

```
192.168.0.10:30080
```

문제

```
port ugly
```

### EXTERNAL-IP

접근

```
192.168.0.240:80
```

장점

```
real load balancer 느낌
```

## 8️⃣ 실제 iptables

EXTERNAL-IP도 결국

```
iptables DNAT
```

된다.

예

```
192.168.0.240:80
↓
10.244.2.5:8080
```

## 9️⃣ Kubernetes 네트워크 관점

클러스터에는 3개의 네트워크 영역이 있다.

```
External Network
↓
Node Network
↓
Pod Network
```

Service는

```
virtual layer
```

이다.

그래서 EXTERNAL-IP는

```
External → Service
```

연결을 만든다.

## 🔟 전체 구조

```
Client
  ↓
EXTERNAL-IP (MetalLB)
  ↓
Node
  ↓
kube-proxy
  ↓
ClusterIP
  ↓
Pod
```

## 🔥 핵심 요약

EXTERNAL-IP는

```
클러스터 외부에서 Service에 접근하기 위한 IP
```

이다.

즉

```
External Network
        ↓
    EXTERNAL-IP
        ↓
     Service
        ↓
       Pod
```

## ⭐ 하나 더 중요한 사실 (많은 사람들이 모름)

LoadBalancer Service는 내부적으로

```
LoadBalancer
   ↓
NodePort
   ↓
ClusterIP
```

구조다.

즉

```
LoadBalancer = NodePort + External IP
```

이다.

---

# 궁금한것 : 일반사용자는 어떻게 접속하는가?

## 1️⃣ 현재 네트워크 구조

네가 준 구조

```
wifi gateway  192.168.0.1
node1         192.168.0.10   (k8s master)
node2         192.168.0.11   (k8s worker)
user laptop   192.168.0.12
```

MetalLB pool

```
192.168.0.100 - 192.168.0.250
```

Service

```
LoadBalancer
EXTERNAL-IP = 192.168.0.100
```

## 2️⃣ 사용자가 브라우저에서 접속

사용자

```
http://192.168.0.100
```

입력

사용자 컴퓨터는 먼저 생각한다.

```
192.168.0.100 이 내 네트워크인가?
```

서브넷

```
192.168.0.0/24
```

이므로 같은 LAN 이다.

그래서 라우터로 보내지 않는다.

대신

```
ARP 요청
```

을 보낸다.

## 3️⃣ ARP Broadcast 발생

사용자 컴퓨터가 LAN 전체에 묻는다.

```
Who has 192.168.0.100?
Tell 192.168.0.12
```

패킷

```
dst MAC = ff:ff:ff:ff:ff:ff
```

즉 LAN 전체 broadcast

## 4️⃣ MetalLB Speaker가 응답

MetalLB는 speaker pod가 각 node에 있다.

예

```
speaker node1
speaker node2
```

하지만 leader 하나만 응답한다.

예

```
node2 가 leader
```

node2가 응답

```
192.168.0.100 is-at aa:bb:cc:dd:ee
```

여기서

```
aa:bb:cc:dd:ee = node2 NIC MAC
```

## 5️⃣ 사용자 컴퓨터 ARP 캐시 저장

사용자 컴퓨터는 이제 이렇게 기억한다.

```
192.168.0.100 → aa:bb:cc:dd:ee
```

즉

```
LoadBalancer IP → node2
```

## 6️⃣ 이제 실제 TCP 요청

브라우저 요청

```
dst IP = 192.168.0.100
dst MAC = node2
```

패킷

```
user laptop
   ↓
node2 NIC
```

도착

## 7️⃣ node2에서 kube-proxy 동작

node2 iptables

```
PREROUTING
↓
KUBE-SERVICES
```

rule

```
192.168.0.100:80
↓
ClusterIP
↓
Pod
```

결과

```
Pod 로 DNAT
```

## 8️⃣ 핵심 포인트

여기서 중요한 사실

```
192.168.0.100 은 실제로 node interface에 존재하지 않는다.
```

확인

```
ip addr
```

봐도 없다.

하지만 ARP로 node MAC을 광고하기 때문에

```
IP → Node
```

처럼 동작한다.

## 9️⃣ 그래서 브라우저 접속 가능

흐름 전체

```
Laptop (192.168.0.12)
    │
    │ ARP broadcast
    ▼
Who has 192.168.0.100
    │
    ▼
MetalLB speaker node2
    │
    │ ARP reply
    ▼
192.168.0.100 is node2 MAC
    │
    ▼
Laptop → node2
    │
    ▼
iptables
    │
    ▼
Pod
```

라우터는 한 번도 관여하지 않는다.

## 🔟 직접 확인 실험 (강력 추천)

사용자 컴퓨터에서

```
arp -a
```

LoadBalancer 접속 전

```
192.168.0.100 없음
```

접속

```
curl 192.168.0.100
```

다시

```
arp -a
```

결과

```
192.168.0.100 aa:bb:cc:dd:ee
```

이 MAC이

```
node NIC
```

이다.
