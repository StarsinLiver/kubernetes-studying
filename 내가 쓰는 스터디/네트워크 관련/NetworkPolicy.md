## 목차

- [1장 NetworkPolicy 란?](#1장-networkpolicy-란)
  - [✅ 1. NetworkPolicy](#-1-networkpolicy)
  - [✅ 2. 전제 조건 (중요🔥)](#-2-전제-조건-중요)
  - [✅ 3. 어디서 실행되냐 (아키텍처 핵심🔥)](#-3-어디서-실행되냐-아키텍처-핵심)
  - [✅ 4. Linux 어디에 저장되냐](#-4-linux-어디에-저장되냐)
    - [✔ iptables 기반](#-iptables-기반)
    - [✔ Calico 확인](#-calico-확인)
  - [✅ 5. 확인 방법 (실무 중요🔥)](#-5-확인-방법-실무-중요)
  - [✅ 6. 핵심 개념](#-6-핵심-개념)
  - [✅ 7. 전체 구조](#-7-전체-구조)
  - [✅ 8. 중요한 흐름 (실제 통신 과정🔥)](#-8-중요한-흐름-실제-통신-과정)
  - [✅ 9. YAML 예시](#-9-yaml-예시)
    - [실전 예제 1 (frontend → backend만 허용)](#실전-예제-1-frontend--backend만-허용)
    - [실전 예제 2 (모든 차단 = default deny)](#실전-예제-2-모든-차단--default-deny)
    - [실전 예제 3 (외부 IP 허용)](#실전-예제-3-외부-ip-허용)
  - [✅ 10. 디버깅 방법 (실무 핵심🔥)](#-10-디버깅-방법-실무-핵심)
- [2장 eBPF 란?](#2장-ebpf-란)
  - [✅ 1. eBPF 란?](#-1-ebpf-란)
  - [✅ 2. 어디서 실행되냐](#-2-어디서-실행되냐)
    - [실행 위치 (Hook Point)](#실행-위치-hook-point)
  - [✅ 3. Linux 어디에 저장되냐](#-3-linux-어디에-저장되냐)
    - [확인 방법](#확인-방법)
      - [1) bpftool (핵심🔥)](#1-bpftool-핵심)
      - [2) map 확인](#2-map-확인)
      - [3) 파일 시스템](#3-파일-시스템)
  - [✅ 4. 핵심 개념](#-4-핵심-개념)
  - [✅ 5. 전체 구조](#-5-전체-구조)
  - [✅ 6. 중요한 흐름 (패킷 처리🔥)](#-6-중요한-흐름-패킷-처리)
    - [예시: Pod → Pod 통신](#예시-pod--pod-통신)
  - [✅ 7. NetworkPolicy와 관계](#-7-networkpolicy와-관계)
  - [✅ 8. 실제 예시 (개념)](#-8-실제-예시-개념)
  - [✅ 9. 어떻게 만드냐 (개발 관점)](#-9-어떻게-만드냐-개발-관점)
  - [✅ 10. 실무에서 중요한 포인트](#-10-실무에서-중요한-포인트)
- [3장 Calico 내부 구조 (NetworkPolicy + iptables) 시나리오](#3장-calico-내부-구조-networkpolicy--iptables-시나리오)
  - [✅ 1. Calico 내부 구조 (NetworkPolicy + iptables)](#-1-calico-내부-구조-networkpolicy--iptables)
  - [✅ 2. 전체 구조 (중요🔥)](#-2-전체-구조-중요)
    - [✔ Calico 각 구성 요소](#-calico-각-구성-요소)
  - [✅ 3. iptables 체인 구조 (실제 핵심🔥)](#-3-iptables-체인-구조-실제-핵심)

---

# 1장 NetworkPolicy 란?

## ✅ 1. NetworkPolicy

✔ 한줄 정리

```
👉 Pod 간 네트워크 통신을 제어하는 방화벽
👉 Pod 간 트래픽을 “허용 목록 기반(Whitelist)”으로 제어하는 네트워크 방화벽
```

✔ 역할

```
Pod ↔ Pod 통신 제어
Namespace 간 통신 제한
외부 ↔ Pod 접근 제어
Zero Trust 네트워크 구현

👉 핵심:
“허용된 것만 통신 가능”
```

## ✅ 2. 전제 조건 (중요🔥)

```
👉 CNI가 지원해야 함
```

예:

```
Calico ✔
Flannel ❌ (기본은 미지원)
```

## ✅ 3. 어디서 실행되냐 (아키텍처 핵심🔥)

```
NetworkPolicy는 “쿠버네티스 자체 기능”이 아님
```

👉 실제 실행 주체:

```
CNI Plugin (예: Calico, Cilium)
```

Calico 사용 중 이라면

```
👉 Calico 사용 중 → 실제 네트워크 제어는 Calico가 수행
```

내부 흐름

```
kubectl apply
   ↓
API Server (정책 저장)
   ↓
etcd (데이터 저장)
   ↓
CNI Plugin (Calico)가 감지
   ↓
iptables / eBPF 규칙 생성
   ↓
Linux Kernel에서 패킷 필터링
```

## ✅ 4. Linux 어디에 저장되냐

```
👉 YAML 자체는 etcd에 저장됨 (파일 X)
```

👉 실제 네트워크 룰은 다음과 같이 생성

### ✔ iptables 기반

```
iptables -L -n
iptables-save
```

### ✔ Calico 확인

```
calicoctl get networkpolicy -A
```

👉 실제 규칙 예시:

```
cali-xxxx chain 생성됨
```

## ✅ 5. 확인 방법 (실무 중요🔥)

Kubernetes 레벨

```
kubectl get networkpolicy -A
kubectl describe networkpolicy <name> -n <ns>
```

Linux 레벨

```
iptables -L -n | grep cali
```

Pod 내부 테스트

```
curl <pod-ip>
```

## ✅ 6. 핵심 개념

✔ 핵심 개념 (매우 중요🔥)

```
👉 기본은 “모두 허용”

👉 NetworkPolicy 하나라도 적용되면:
→ 기본이 “모두 차단”으로 바뀜
```

✔ 예시 흐름

```
NetworkPolicy 없음 → 전부 통신 가능
NetworkPolicy 생성 → 지정된 것만 허용
```

🔥 방향성 존재

```
Ingress → 들어오는 트래픽
Egress → 나가는 트래픽
```

🔥 선택된 Pod만 적용됨

```
podSelector
👉 선택된 Pod만 영향 받음
```

🔥 Namespace 기준 분리

```
1. 기본적으로 같은 Namespace 기준
2. cross-namespace는 명시 필요
```

## ✅ 7. 전체 구조

```
Namespace
 ├── Pod A (frontend)
 ├── Pod B (backend)
 ├── NetworkPolicy
       ↓
   Calico
       ↓
   iptables / eBPF
       ↓
   Linux Kernel filtering
```

## ✅ 8. 중요한 흐름 (실제 통신 과정🔥)

예:

```
Pod A → Pod B 요청
```

```
1. Pod A에서 패킷 생성
↓
2. Node의 네트워크 스택 진입
↓
3. Calico rule 확인
↓
4. NetworkPolicy match 확인
↓
5. 허용이면 전달 / 아니면 DROP
```

## ✅ 9. YAML 예시

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-nginx
  namespace: dev
spec:
  # 정책 적용대상 Pod (같은 namespace 일 시)
  podSelector:
    matchLabels:
      app: nginx

  # Ingress (네트워크가 들어오는 트래픽 제어)
  # Egress (네트워크가 나가는 트래픽 제어)
  policyTypes:
    - Ingress
    - Egress

  # ingress / egress
  ingress:
    # from / to : 허용 대상
    - from:
        # 정책 적용대상 Pod (같은 namespace 일 시)
        - podSelector:
            matchLabels:
              app: frontend
        # 정책 적용대상 Namespace (Namespace 지정 시)
        - namespaceSelector:
            matchLabels:
              team: backend
              kube-system: "true" # kube-dns 허용

        # 외부 IP 허용
        - ipBlock:
            cidr: 192.168.1.0/24
            except:
              - 192.168.1.10/32

      # 허용 포트
      ports:
        - protocol: TCP
          port: 80
```

### 실전 예제 1 (frontend → backend만 허용)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: dev

spec:
  podSelector:
    matchLabels:
      app: backend

  policyTypes:
    - Ingress

  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
```

### 실전 예제 2 (모든 차단 = default deny)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: dev

spec:
  podSelector: {}
  policyTypes:
    - Ingress
```

```
👉 결과:

모든 Pod incoming 차단
```

### 실전 예제 3 (외부 IP 허용)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: dev

spec:
  podSelector:
    matchLabels:
      app: backend

  policyTypes:
    - Ingress

  ingress:
    - from:
        - ipBlock:
            cidr: 0.0.0.0/0
      ports:
        - protocol: TCP
          port: 8080
```

## ✅ 10. 디버깅 방법 (실무 핵심🔥)

1. 정책 확인

```
kubectl get netpol -A
```

2. Pod label 확인

```
kubectl get pod --show-labels
```

3. iptables 확인

```
iptables -L -n
```

4. 실제 테스트

```
kubectl exec -it pod -- curl <target>
```

---

# 2장 eBPF 란?

## ✅ 1. eBPF 란?

한줄 정리

```
👉 Linux 커널 안에서 안전하게 실행되는 “초경량 프로그램”으로,
네트워크/보안/관찰을 고성능으로 처리하는 기술
```

어떤 역할을 하는가

```
eBPF는 원래 패킷 필터링에서 시작했는데 지금은 거의 “커널 확장 플랫폼” 수준이다.
```

주요 역할

```
네트워크 패킷 필터링 (방화벽)
트래픽 제어 (NetworkPolicy)
로드밸런싱
모니터링 (latency, syscall tracing)
보안 (runtime protection)
```

👉 한마디로

```
“iptables보다 훨씬 빠르고 유연한 커널 내부 로직”
```

## ✅ 2. 어디서 실행되냐

```
👉 Linux Kernel 내부에서 실행됨
```

구조:

```
User Space
   ↓
eBPF 프로그램 로드
   ↓
Kernel Space (여기서 실행🔥)
   ↓
네트워크 / 시스템 이벤트 처리
```

### 실행 위치 (Hook Point)

eBPF는 특정 지점에 붙는다

```
XDP (네트워크 카드 바로 앞)
TC (Traffic Control)
Socket
System Call
Tracepoint
```

👉 NetworkPolicy는 보통:

```
TC / XDP 레벨에서 처리됨
```

## ✅ 3. Linux 어디에 저장되냐

```
👉 파일로 저장 ❌
```

대신

```
커널 메모리에 로드됨
/sys/fs/bpf (BPF virtual filesystem)에 일부 노출됨
```

### 확인 방법

#### 1) bpftool (핵심🔥)

```
bpftool prog show
```

#### 2) map 확인

```
bpftool map show
```

#### 3) 파일 시스템

```
ls /sys/fs/bpf/
```

## ✅ 4. 핵심 개념

1. “커널에 코드를 넣는다”

```
→ 기존:
iptables = 룰 테이블

→ eBPF:
코드 자체 실행
```

🔥 2) Sandbox (안전성)

```
무한 루프 금지
메모리 제한
verifier 검사 통과해야 실행
```

🔥 3) Map (데이터 저장소)

eBPF는 상태 저장 가능

```
Program ↔ Map
```

예:

```
IP 목록
정책 정보
```

## ✅ 5. 전체 구조

```
Kubernetes
   ↓
CNI (Cilium / Calico eBPF mode)
   ↓
eBPF Program 생성
   ↓
Kernel에 로드
   ↓
패킷 처리
```

## ✅ 6. 중요한 흐름 (패킷 처리🔥)

### 예시: Pod → Pod 통신

iptables 방식

```
패킷 → iptables chain → rule match → 결정
```

eBPF 방식

```
패킷 → eBPF program 실행 → 바로 판단
```

👉 차이:

```
iptables = 룰 순차 탐색 (느림)
eBPF = 코드 실행 (빠름🔥)
```

## ✅ 7. NetworkPolicy와 관계

iptables 기반 (Calico 기본)

```
NetworkPolicy → iptables rule 생성
```

eBPF 기반 (Cilium / Calico eBPF mode)

```
NetworkPolicy → eBPF program 생성
```

결과

| 방식     | 특징              |
| -------- | ----------------- |
| iptables | 느림, 단순        |
| eBPF     | 빠름, 확장성 높음 |

## ✅ 8. 실제 예시 (개념)

iptables

```
ALLOW tcp 10.0.0.1 → 10.0.0.2:80
```

eBPF

```
if (src == 10.0.0.1 && dst == 10.0.0.2 && port == 80) {
    allow();
} else {
    drop();
}
```

👉 코드로 처리됨

## ✅ 9. 어떻게 만드냐 (개발 관점)

```
일반 사용자는 직접 안 만듦 ❌
```

👉 대신:

```
Cilium
Calico (eBPF mode)

이 자동 생성
```

직접 만들면 (참고)

```
1. C 코드 작성
2. LLVM으로 컴파일
3. bpftool로 로드
```

## ✅ 10. 실무에서 중요한 포인트

❗ 1) 성능

```
iptables: O(n)
eBPF: 거의 O(1)
```

❗ 2) 대규모 클러스터

```
→ eBPF 필수 수준
```

❗ 3) 디버깅 어려움

```
iptables보다 복잡함
```

---

# 3장 Calico 내부 구조 (NetworkPolicy + iptables) 시나리오

## ✅ 1. Calico 내부 구조 (NetworkPolicy + iptables)

✔ 한줄 정리

```
👉 NetworkPolicy를 iptables 체인으로 변환해서 Linux 커널에서 패킷을 필터링하는 구조
```

## ✅ 2. 전체 구조 (중요🔥)

```
Kubernetes NetworkPolicy
        ↓
API Server (etcd 저장)
        ↓
Calico (Felix)
        ↓
iptables rule 생성
        ↓
Linux Kernel (netfilter)
        ↓
패킷 허용/차단
```

### ✔ Calico 각 구성 요소

🔹 Felix (핵심🔥)

```
각 노드에서 실행
NetworkPolicy 감지
iptables 규칙 생성
```

🔹 BIRD (라우팅)

```
Pod 네트워크 라우팅 (BGP)
```

🔹 datastore

```
etcd or Kubernetes API
```

## ✅ 3. iptables 체인 구조 (실제 핵심🔥)

```
Calico는 자체 체인을 만든다
```

✔ 주요 체인

```
cali-INPUT
cali-OUTPUT
cali-FORWARD
cali-from-wl
cali-to-wl
cali-pi-<hash>   (policy ingress)
cali-po-<hash>   (policy egress)
```

✔ 흐름 구조

```
패킷 들어옴
   ↓
cali-FORWARD
   ↓
cali-from-wl (출발 Pod 검사)
   ↓
cali-pi-* (Ingress 정책)
   ↓
cali-po-* (Egress 정책)
   ↓
허용 or DROP
```

✔ 실제 확인

```
iptables -L -n | grep cali
iptables-save | grep cali
```

실제 예시

```
Chain cali-pi-xxxx (1 references)
 target     prot opt source      destination
 ACCEPT     tcp  --  10.0.0.1    10.0.0.2 tcp dpt:80
 DROP       all  --  0.0.0.0/0   0.0.0.0/0
```
