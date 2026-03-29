## 목차

- [1장 CoreDNS 란](#1장-coredns-란)
  - [1️⃣ DNS가 왜 필요한가](#1️⃣-dns가-왜-필요한가)
  - [2️⃣ CoreDNS란 무엇인가](#2️⃣-coredns란-무엇인가)
  - [3️⃣ CoreDNS 전체 구조](#3️⃣-coredns-전체-구조)
  - [4️⃣ 실제 Linux에서 어디서 실행되나](#4️⃣-실제-linux에서-어디서-실행되나)
  - [5️⃣ CoreDNS 설정파일 (Corefile)](#5️⃣-coredns-설정파일-corefile)
  - [6️⃣ Pod에서 DNS 설정](#6️⃣-pod에서-dns-설정)
  - [7️⃣ Service DNS](#7️⃣-service-dns)
  - [8️⃣ Service DNS 패킷 흐름](#8️⃣-service-dns-패킷-흐름)
  - [9️⃣ Headless Service DNS](#9️⃣-headless-service-dns)
  - [🔟 StatefulSet DNS](#-statefulset-dns)
  - [11장 kube-proxy 와 관계](#11장-kube-proxy-와-관계)
  - [12장 실제 패킷 흐름](#12장-실제-패킷-흐름)
  - [13장 Linux에서 실제 확인](#13장-linux에서-실제-확인)
  - [14장 핵심 개념 정리](#14장-핵심-개념-정리)
  - [🔥 Kubernetes DNS 전체 흐름 (진짜 핵심)](#-kubernetes-dns-전체-흐름-진짜-핵심)
- [2장 좀 더 자세히 알아보자](#2장-좀-더-자세히-알아보자)
  - [1️⃣ Pod 내부에 어떻게 DNS 서버 IP가 붙는가](#1️⃣-pod-내부에-어떻게-dns-서버-ip가-붙는가)
    - [/etc/resolv.conf 생성 전체 흐름](#etcresolvconf-생성-전체-흐름)
    - [kubelet DNS 설정 위치](#kubelet-dns-설정-위치)
    - [kubelet이 Pod에 넣는 resolv.conf](#kubelet이-pod에-넣는-resolvconf)
    - [실제 Linux에서 어떻게 들어가나](#실제-linux에서-어떻게-들어가나)
    - [확인 방법](#확인-방법)
  - [2️⃣ Headless Service DNS](#2️⃣-headless-service-dns)
    - [Headless Service 생성](#headless-service-생성)
    - [CoreDNS 내부 동작](#coredns-내부-동작)
  - [3️⃣ StatefulSet DNS](#3️⃣-statefulset-dns)
    - [구성](#구성)
    - [왜 필요한가](#왜-필요한가)
  - [4️⃣ 일반 Service vs Headless vs StatefulSet](#4️⃣-일반-service-vs-headless-vs-statefulset)
    - [DNS 결과 비교](#dns-결과-비교)
  - [5️⃣ CoreDNS 패킷 흐름 tcpdump로 보기](#5️⃣-coredns-패킷-흐름-tcpdump로-보기)
  - [6️⃣ CoreDNS plugin 구조](#6️⃣-coredns-plugin-구조)
    - [Corefile](#corefile)
    - [흐름](#흐름)
    - [kubernetes plugin 역할](#kubernetes-plugin-역할)
    - [실제 코드 구조](#실제-코드-구조)
  - [7️⃣ Pod DNS 정책 (dnsPolicy)](#7️⃣-pod-dns-정책-dnspolicy)
    - [ClusterFirst](#clusterfirst)
    - [Default](#default)
    - [None](#none)
  - [8️⃣ NodeLocal DNSCache](#8️⃣-nodelocal-dnscache)
  - [🔥 Kubernetes DNS 전체 흐름 (진짜 핵심)](#-kubernetes-dns-전체-흐름-진짜-핵심-1)
  - [🔥 Kubernetes DNS 핵심 개념](#-kubernetes-dns-핵심-개념)

---

# 1장 CoreDNS 란

## 1️⃣ DNS가 왜 필요한가

예를 들어 Pod가 있다고 하자.

```
pod-a → pod-b 로 통신

[pod-b의 IP]
10.244.2.15
```

⚠️ 근데 문제는 Pod IP는 계속 바뀜

```
pod-b 삭제 → 재생성
IP = 10.244.3.21
```

그러면 모든 프로그램이 깨짐.

그래서 사용하는게

```
DNS
```

즉

```
pod-a → db-service.default.svc.cluster.local
```

이걸

```
10.96.0.10
```

같은 IP로 변환해주는 것.

이걸 Kubernetes에서 하는 DNS 서버가

```
👉 CoreDNS
```

## 2️⃣ CoreDNS란 무엇인가

한줄 정의

```
Kubernetes 내부 DNS 서버
```

조금 더 정확히

```
Kubernetes Service / Pod 이름을 IP로 변환하는 DNS 서버
```

## 3️⃣ CoreDNS 전체 구조

Kubernetes에서 DNS 구조

```
Pod
 │
 │ DNS Query
 ▼
nameserver 10.96.0.10
 │
 ▼
CoreDNS Service
 │
 ▼
CoreDNS Pod
 │
 ▼
Kubernetes API 조회
 │
 ▼
Service / Pod IP 반환
```

즉

```
Pod
 → CoreDNS
    → Kubernetes API
        → 결과 반환
```

## 4️⃣ 실제 Linux에서 어디서 실행되나

CoreDNS는 Pod 로 실행된다.

확인

```
kubectl get pods -n kube-system
```

예

```
coredns-7c8f9b6f8c-abcde
coredns-7c8f9b6f8c-xyz12
```

즉

```
Deployment
   ↓
CoreDNS Pod
   ↓
DNS Server process
```

컨테이너 내부

```
/coredns
```

## 5️⃣ CoreDNS 설정파일 (Corefile)

CoreDNS는

```
Corefile
```

이라는 설정파일을 사용한다.

Kubernetes에서는 이것이

```
ConfigMap
```

으로 저장된다.

확인

```
kubectl get configmap -n kube-system

에서 coredns라는 이름으로 있다.
```

내용 보기

```
kubectl edit configmap coredns -n kube-system
```

예

```
.:53 {
    errors
    health
    kubernetes cluster.local
    forward . /etc/resolv.conf
    cache 30
}

[설명]

.:53
: 모든 DNS 요청 처리

kubernetes cluster.local
: Kubernetes DNS plugin

forward . /etc/resolv.conf
: 외부 DNS forwarding
```

## 6️⃣ Pod에서 DNS 설정

Pod 안에 들어가보면

```
cat /etc/resolv.conf
```

결과

```
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
```

여기서

```
10.96.0.10
```

이것이

```
👉 CoreDNS Service
```

예시

```
➜  kubectl exec -it nginx-7854ff8877-cfg6c -- cat /etc/resolv.conf

search default.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5
```

```
➜  kubectl get service -n kube-system
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   6d21h


➜  iptables -t nat -L KUBE-SERVICES | grep 10.96
KUBE-SVC-JD5MR3NA4I4DYORP  tcp  --  anywhere             10.96.0.10           /* kube-system/kube-dns:metrics cluster IP */
KUBE-SVC-TCOU7JCQXEZGVUNU  udp  --  anywhere             10.96.0.10           /* kube-system/kube-dns:dns cluster IP */
KUBE-SVC-ERIFXISQEP7F7OF4  tcp  --  anywhere             10.96.0.10           /* kube-system/kube-dns:dns-tcp cluster IP */
```

## 7️⃣ Service DNS

Service 만들면 자동 DNS 생성됨

예

```
kubectl create service clusterip redis
```

그러면 DNS 이름

```
redis.default.svc.cluster.local
```

구조

```
<service>.<namespace>.svc.cluster.local
```

예

```
mysql.database.svc.cluster.local
```

실제 DNS 결과

```
nslookup redis
```

결과

```
Name: redis.default.svc.cluster.local
Address: 10.96.15.23
```

이 IP는

```
👉 Service ClusterIP
```

예시

```
➜  rocky k get all  -o wide
NAME                         READY   STATUS    RESTARTS      AGE     IP             NODE   NOMINATED NODE   READINESS GATES
pod/network-toolbox          1/1     Running   4 (23m ago)   6d19h   10.244.71.58   tiny   <none>           <none>
pod/nginx-7854ff8877-cfg6c   1/1     Running   1 (23m ago)   29h     10.244.71.57   tiny   <none>           <none>
pod/nginx-7854ff8877-gm97g   1/1     Running   1 (23m ago)   29h     10.244.71.55   tiny   <none>           <none>

NAME                         TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)          AGE     SELECTOR
service/kubernetes           ClusterIP      10.96.0.1        <none>          443/TCP          6d21h   <none>
service/nginx                ClusterIP      10.107.129.9     <none>          80/TCP           29h     app=nginx

➜  kubectl exec -it network-toolbox -- nslookup nginx
;; Got recursion not available from 10.96.0.10
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   nginx.default.svc.cluster.local
Address: 10.107.129.9
;; Got recursion not available from 10.96.0.10
```

## 8️⃣ Service DNS 패킷 흐름

Pod에서

```
curl redis
```

흐름

```
Pod
 │
 │ DNS Query
 ▼
CoreDNS
 │
 │ Kubernetes API 조회
 ▼
Service ClusterIP 반환
 │
 ▼
Pod
 │
 ▼
kube-proxy
 │
 ▼
iptables
 │
 ▼
Pod backend
```

즉

```
DNS = CoreDNS
Load balancing = kube-proxy
```

## 9️⃣ Headless Service DNS

일반 Service

```
ClusterIP 있음
```

DNS 결과

```
redis → 10.96.0.23
```

하지만 Headless는

```
clusterIP: None
```

DNS 결과

```
redis → pod IP 목록
```

예

```
10.244.1.2
10.244.2.3
10.244.3.4
```

즉

```
Load balancing 없음.
```

## 🔟 StatefulSet DNS

StatefulSet은 Pod이 이름 고정

예

```
mysql-0
mysql-1
mysql-2
```

Headless service와 함께 사용.

DNS

```
mysql-0.mysql.default.svc.cluster.local
mysql-1.mysql.default.svc.cluster.local
```

각각

```
mysql-0 → 10.244.1.2
mysql-1 → 10.244.1.3
```

## 11장 kube-proxy 와 관계

CoreDNS는

```
IP만 알려준다
```

트래픽 전달은

```
kube-proxy
```

예

```
Pod → Service
```

흐름

```
Pod
 ↓
DNS (CoreDNS)
 ↓
ClusterIP
 ↓
iptables
 ↓
backend pod
```

## 12장 실제 패킷 흐름

Pod에서

```
curl redis
```

패킷 흐름

```
Pod
 │
 │ UDP 53
 ▼
CoreDNS
 │
 │ DNS response
 ▼
Pod
 │
 │ TCP 6379
 ▼
Service IP
 │
 ▼
iptables
 │
 ▼
Pod backend
```

## 13장 Linux에서 실제 확인

CoreDNS Pod

```
➜  kubectl get pods -n kube-system
```

CoreDNS 로그

```
➜  kubectl logs -n kube-system coredns-xxxxx
```

DNS 확인

Pod 생성

```
➜  kubectl run test --image=busybox -it --rm
➜  nslookup kubernetes
```

resolv.conf 확인

```
➜  cat /etc/resolv.conf
```

CoreDNS Service

```
➜  kubectl get svc -n kube-system

kube-dns   ClusterIP   10.96.0.10
```

iptables 확인

```
iptables -t nat -L | grep 10.96.0.10

찾기
KUBE-SVC
```

## 14장 핵심 개념 정리

CoreDNS

한 줄 정의

```
Kubernetes 내부 DNS 서버
```

Service DNS

```
service.namespace.svc.cluster.local
```

Pod DNS

```
pod-ip.namespace.pod.cluster.local
```

Headless

```
Service → Pod IP 목록 반환
```

StatefulSet

```
pod-0.service.namespace.svc.cluster.local
```

## 🔥 Kubernetes DNS 전체 흐름 (진짜 핵심)

```
Pod
 │
 │ DNS Query
 ▼
CoreDNS Service
 │
 ▼
CoreDNS Pod
 │
 ▼
Kubernetes API
 │
 ▼
Service IP 반환
 │
 ▼
Pod
 │
 ▼
kube-proxy
 │
 ▼
iptables
 │
 ▼
backend pod
```

---

# 2장 좀 더 자세히 알아보자

## 1️⃣ Pod 내부에 어떻게 DNS 서버 IP가 붙는가

결론부터 한 줄

```
kubelet이 Pod 생성 시 /etc/resolv.conf 를 자동 생성해서 넣는다
```

즉 DNS 설정은

```
CoreDNS → kubelet → Pod /etc/resolv.conf
```

### /etc/resolv.conf 생성 전체 흐름

Pod 생성

```
kubectl run nginx --image=nginx
```

흐름

```
kubectl
   ↓
API Server
   ↓
Scheduler
   ↓
Node 선택
   ↓
kubelet
   ↓
Pod 생성
```

이때 kubelet이 DNS 설정을 넣는다

### kubelet DNS 설정 위치

노드에서 kubelet 설정 확인

```
ps -ef | grep kubelet
```

보면 이런 옵션 있음

```
--cluster-dns=10.96.0.10
--cluster-domain=cluster.local
```

이게 의미하는 것

```
cluster DNS server
```

### kubelet이 Pod에 넣는 resolv.conf

Pod 내부

```
cat /etc/resolv.conf
```

결과

```
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

설명

```
nameserver
10.96.0.10
```

→ CoreDNS Service IP

search domain

```
default.svc.cluster.local
svc.cluster.local
cluster.local
```

그래서

```
curl redis
```

하면 자동으로

```
redis.default.svc.cluster.local
```

로 확장됨

### 실제 Linux에서 어떻게 들어가나

Pod 생성 시

```
kubelet
  ↓
container runtime
  ↓
network namespace 생성
  ↓
resolv.conf 생성
```

실제 container runtime 경로 (containerd)

```
/var/lib/containerd/
```

또는

```
/run/containerd/
```

### 확인 방법

노드에서

```
crictl ps
```

container id 확인

```
crictl inspect <container-id>
```

DNS 설정 확인 가능

## 2️⃣ Headless Service DNS

먼저 한 줄 정의

```
ClusterIP 없이 Pod IP를 직접 DNS로 반환하는 Service
```

### Headless Service 생성

yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  clusterIP: None
  selector:
    app: redis
  ports:
    - port: 6379
```

핵심

```
clusterIP: None
```

```
➜  k get pods -o wide --show-labels -l app=nginx
NAME                     READY   STATUS    RESTARTS      AGE   IP             NODE   NOMINATED NODE   READINESS GATES   LABELS
nginx-7854ff8877-cfg6c   1/1     Running   1 (58m ago)   29h   10.244.71.57   tiny   <none>           <none>            app=nginx,pod-template-hash=7854ff8877
nginx-7854ff8877-gm97g   1/1     Running   1 (58m ago)   29h   10.244.71.55   tiny   <none>           <none>            app=nginx,pod-template-hash=7854ff8877

➜  k get svc
NAME                     TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
headless-service-nginx   ClusterIP   None         <none>        80/TCP    7m12s

➜  k exec -it network-toolbox -- nslookup headless-service-nginx
;; Got recursion not available from 10.96.0.10
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   headless-service-nginx.default.svc.cluster.local
Address: 10.244.71.55
Name:   headless-service-nginx.default.svc.cluster.local
Address: 10.244.71.57
;; Got recursion not available from 10.96.0.10
```

```
Service IP 없음
Pod IP 직접 반환
```

### CoreDNS 내부 동작

CoreDNS plugin

```
kubernetes
```

이 plugin이

```
Endpoint
```

를 조회한다.

Headless Service는

```
Endpoint → Pod IP 목록
```

을 그대로 반환.

## 3️⃣ StatefulSet DNS

StatefulSet은

```
Pod마다 고정 DNS
```

### 구성

필수

```
Headless Service
+
StatefulSet
```

Headless Service

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

StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql
```

생성되는 Pod

```
mysql-0
mysql-1
mysql-2
```

DNS

```
각 Pod

mysql-0.mysql.default.svc.cluster.local
mysql-1.mysql.default.svc.cluster.local
mysql-2.mysql.default.svc.cluster.local
```

DNS 결과

```
nslookup mysql-0.mysql
```

결과

```
10.244.1.3
```

### 왜 필요한가

```
DB cluster
```

예

```
redis cluster
zookeeper
kafka
mysql replication
```

[예시]

```
➜  k get pods -o wide -l app=stateful-nginx
NAME               READY   STATUS    RESTARTS   AGE   IP              NODE          NOMINATED NODE   READINESS GATES
stateful-nginx-0   1/1     Running   0          2m    10.244.194.65   k8s-worker1   <none>           <none>
stateful-nginx-1   1/1     Running   0          87s   10.244.71.59    tiny          <none>           <none>

➜  k get svc
NAME                     TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
stateful-nginx           ClusterIP   None         <none>        80/TCP    2m7s

➜  k exec -it network-toolbox -- nslookup stateful-nginx
;; Got recursion not available from 10.96.0.10
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   stateful-nginx.default.svc.cluster.local
Address: 10.244.71.59
Name:   stateful-nginx.default.svc.cluster.local
Address: 10.244.194.65
;; Got recursion not available from 10.96.0.10
```

## 4️⃣ 일반 Service vs Headless vs StatefulSet

| 특징           | Service    | Headless    | StatefulSet |
| -------------- | ---------- | ----------- | ----------- |
| ClusterIP      | 있음       | 없음        | 없음        |
| DNS 반환       | Service IP | Pod IP 목록 | Pod별 DNS   |
| Load balancing | kube-proxy | 없음        | 없음        |
| Pod identity   | 없음       | 없음        | 있음        |

### DNS 결과 비교

Service

```
redis
↓
10.96.0.20
```

Headless

```
redis
↓
10.244.1.5
10.244.2.7
```

StatefulSet

```
redis-0.redis
↓
10.244.1.5
```

## 5️⃣ CoreDNS 패킷 흐름 tcpdump로 보기

테스트 Pod

```
kubectl run dns-test --image=busybox -it --rm
```

DNS query

```
nslookup kubernetes
```

Node에서

```
tcpdump -i any port 53
```

보면

```
Pod IP → CoreDNS UDP 53
```

예

```
10.244.1.5 → 10.96.0.10 UDP 53
```

응답

```
10.96.0.10 → 10.244.1.5
```

## 6️⃣ CoreDNS plugin 구조

CoreDNS는

```
plugin chain
```

구조

### Corefile

```
.:53 {
    errors
    health
    kubernetes cluster.local
    forward . /etc/resolv.conf
    cache 30
}
```

### 흐름

```
DNS query
 ↓
errors
 ↓
kubernetes plugin
 ↓
cache
 ↓
forward
```

### kubernetes plugin 역할

이 plugin이

```
Service
Endpoint
Pod
Namespace

정보를 API server watch 한다.
```

즉

```
CoreDNS
   ↓
watch API
   ↓
Service DB 생성
```

### 실제 코드 구조

CoreDNS plugin

```
plugin/kubernetes
```

작동

```
API watcher
 ↓
local cache
 ↓
DNS response
```

## 7️⃣ Pod DNS 정책 (dnsPolicy)

Pod spec에 해당 정책을 넣을 수 있다.

```
dnsPolicy
```

종류

| 정책         | 의미      |
| ------------ | --------- |
| ClusterFirst | 기본      |
| Default      | Node DNS  |
| None         | 직접 설정 |

### ClusterFirst

기본

```
nameserver → CoreDNS
```

### Default

```
Node /etc/resolv.conf 사용
```

### None

사용자가 직접 설정

```
dnsConfig
```

예

```
dnsPolicy: None
dnsConfig:
  nameservers:
  - 8.8.8.8
```

## 8️⃣ NodeLocal DNSCache

대규모 cluster 문제

```
모든 Pod → CoreDNS
```

그래서 해당 문제가 생긴다.

```
DNS bottleneck
```

해결책은 다음과 같다.

```
NodeLocal DNSCache
```

구조

```
Pod
 ↓
NodeLocal DNS
 ↓
CoreDNS
```

즉

```
DNS cache node에 생성
```

Pod DNS

```
169.254.20.10

-> NodeLocal DNS ip
```

장점

```
DNS latency 감소
CoreDNS 부하 감소
```

## 🔥 Kubernetes DNS 전체 흐름 (진짜 핵심)

```
Pod
 │
 │ DNS query
 ▼
CoreDNS service
 │
 ▼
CoreDNS pod
 │
 ▼
kubernetes plugin
 │
 ▼
API server
 │
 ▼
Service / Endpoint
 │
 ▼
IP 반환
 │
 ▼
Pod
 │
 ▼
Service IP
 │
 ▼
kube-proxy
 │
 ▼
iptables
 │
 ▼
Pod backend
```

## 🔥 Kubernetes DNS 핵심 개념

CoreDNS

```
DNS server
```

kubelet

```
Pod DNS 설정 생성
```

Service

```
DNS 이름 제공
```

kube-proxy

```
Load balancing
```
