➜ ## 목차

- [1장 CNI 란 무엇인가](#1장-cni-란-무엇인가)
  - [1️⃣ CNI가 무엇인가](#1️⃣-cni가-무엇인가)
  - [2️⃣ Kubernetes에서 왜 필요한가](#2️⃣-kubernetes에서-왜-필요한가)
  - [3️⃣ Kubernetes Pod 생성 네트워크 흐름](#3️⃣-kubernetes-pod-생성-네트워크-흐름)
  - [4️⃣ CNI가 실제 하는 일](#4️⃣-cni가-실제-하는-일)
  - [5️⃣ 실제 Linux 구조](#5️⃣-실제-linux-구조)
  - [6️⃣ veth pair](#6️⃣-veth-pair)
  - [7️⃣ Pod 내부 구조](#7️⃣-pod-내부-구조)
  - [8️⃣ CNI Plugin 구조](#8️⃣-cni-plugin-구조)
  - [9️⃣ CNI Plugin 위치](#9️⃣-cni-plugin-위치)
  - [10. CNI 설정 파일](#10-cni-설정-파일)
  - [11. CNI 호출 방식](#11-cni-호출-방식)
  - [12. CNI command](#12-cni-command)
    - [ADD](#add)
    - [DEL](#del)
  - [CHECK](#check)
  - [13. 실제 CNI 호출 흐름](#13-실제-cni-호출-흐름)
  - [14. Pod IP는 누가 주는가](#14-pod-ip는-누가-주는가)
  - [15. Kubernetes 네트워크 규칙](#15-kubernetes-네트워크-규칙)
  - [16. 실제 Calico 구조 (간단히)](#16-실제-calico-구조-간단히)
  - [17. 🔥 지금까지 흐름 정리](#17--지금까지-흐름-정리)
- [2장. Linux 기준으로 어떻게 실제로 생성하는가](#2장-linux-기준으로-어떻게-실제로-생성하는가)
  - [1️⃣ veth pair란 무엇인가](#1️⃣-veth-pair란-무엇인가)
  - [2️⃣ veth pair 생성 명령어](#2️⃣-veth-pair-생성-명령어)
  - [3️⃣ veth 생성은 어디 코드에서 하는가](#3️⃣-veth-생성은-어디-코드에서-하는가)
  - [4️⃣ Pod network namespace](#4️⃣-pod-network-namespace)
  - [5️⃣ veth 한쪽을 Pod namespace로 이동](#5️⃣-veth-한쪽을-pod-namespace로-이동)
  - [6️⃣ Pod 내부 인터페이스 이름 변경](#6️⃣-pod-내부-인터페이스-이름-변경)
  - [7️⃣ IPAM (Pod IP 할당)](#7️⃣-ipam-pod-ip-할당)
  - [8️⃣ IPAM 설정파일 위치](#8️⃣-ipam-설정파일-위치)
  - [9️⃣ host-local IPAM 예시](#9️⃣-host-local-ipam-예시)
  - [10. IPAM 상태 저장 위치](#10-ipam-상태-저장-위치)
  - [11. Pod에 IP 할당](#11-pod에-ip-할당)
  - [12. 라우팅 생성](#12-라우팅-생성)
  - [13. Node 라우팅](#13-node-라우팅)
  - [14. bridge란 무엇인가](#14-bridge란-무엇인가)
  - [15. bridge 생성 명령어](#15-bridge-생성-명령어)
  - [16. veth와 bridge 연결](#16-veth와-bridge-연결)
  - [17. overlay network](#17-overlay-network)
  - [18. VXLAN 인터페이스 생성](#18-vxlan-인터페이스-생성)
  - [19. overlay 패킷 흐름](#19-overlay-패킷-흐름)
  - [20. 전체 구조](#20-전체-구조)
- [3장 Calico 전체 아키텍처](#3장-calico-전체-아키텍처)
  - [1️⃣ Calico 전체 아키텍처](#1️⃣-calico-전체-아키텍처)
  - [2️⃣ Calico 주요 컴포넌트](#2️⃣-calico-주요-컴포넌트)
  - [3️⃣ Calico CNI Plugin](#3️⃣-calico-cni-plugin)
  - [4️⃣ Felix (Calico의 핵심)](#4️⃣-felix-calico의-핵심)
    - [Felix의 역할](#felix의-역할)
    - [Felix가 하는 일 (가장 중요)](#felix가-하는-일-가장-중요)
      - [Felix는 routing table을 만든다.](#felix는-routing-table을-만든다)
      - [Felix는 iptables도 만든다.](#felix는-iptables도-만든다)
  - [6️⃣ BIRD (BGP daemon)](#6️⃣-bird-bgp-daemon)
  - [7️⃣ Typha](#7️⃣-typha)
    - [Felix 의 문제점](#felix-의-문제점)
  - [8️⃣ Calico 네트워크 모드](#8️⃣-calico-네트워크-모드)
  - [9️⃣ Pod 생성 시 실제 흐름](#9️⃣-pod-생성-시-실제-흐름)
  - [🔥 Calico 구조 핵심 정리](#-calico-구조-핵심-정리)
- [4장 Calico 는 보통 host-ipam 을 사용하지 않는다.](#4장-calico-는-보통-host-ipam-을-사용하지-않는다)
  - [1️⃣ /var/lib/cni/networks 는 언제 사용되는가](#1️⃣-varlibcninetworks-는-언제-사용되는가)
  - [2️⃣ Calico는 보통 host-local IPAM을 안 쓴다](#2️⃣-calico는-보통-host-local-ipam을-안-쓴다)
  - [3️⃣ calico-ipam 은 어디에 저장하나](#3️⃣-calico-ipam-은-어디에-저장하나)
  - [4️⃣ Calico IP 할당 구조](#4️⃣-calico-ip-할당-구조)
  - [5️⃣ 실제 확인 명령어](#5️⃣-실제-확인-명령어)
  - [6️⃣ Calico가 host-local을 안쓰는 이유](#6️⃣-calico가-host-local을-안쓰는-이유)
  - [7️⃣ 실제 Calico 관련 파일 위치](#7️⃣-실제-calico-관련-파일-위치)
  - [8️⃣ Calico 실제 네트워크 인터페이스](#8️⃣-calico-실제-네트워크-인터페이스)
  - [9️⃣ Pod veth 이름](#9️⃣-pod-veth-이름)
  - [10. Calico의 Pod IP 할당 흐름](#10-calico의-pod-ip-할당-흐름)
- [5장 Calico가 Pod 생성시 실제로 하는 일](#5장-calico가-pod-생성시-실제로-하는-일)
  - [1️⃣ Pod 생성 → kubelet → CNI 호출](#1️⃣-pod-생성--kubelet--cni-호출)
  - [2️⃣ CNI가 받는 정보](#2️⃣-cni가-받는-정보)
  - [3️⃣ Calico IPAM 동작](#3️⃣-calico-ipam-동작)
  - [4️⃣ veth pair 생성](#4️⃣-veth-pair-생성)
  - [5️⃣ veth 이동](#5️⃣-veth-이동)
  - [6️⃣ Pod IP 설정](#6️⃣-pod-ip-설정)
  - [7️⃣ Pod default route](#7️⃣-pod-default-route)
  - [8️⃣ Node 쪽 인터페이스](#8️⃣-node-쪽-인터페이스)
  - [9️⃣ Node routing table](#9️⃣-node-routing-table)
  - [🔟 Pod → Pod (같은 Node) 패킷 흐름](#-pod--pod-같은-node-패킷-흐름)
  - [11. Pod → Pod (다른 Node) 패킷 흐름](#11-pod--pod-다른-node-패킷-흐름)
  - [12. vxlan interface](#12-vxlan-interface)
  - [13. VXLAN FDB](#13-vxlan-fdb)
  - [14. Calico routing 방식](#14-calico-routing-방식)
  - [15. 실제 Node 인터페이스들](#15-실제-node-인터페이스들)
  - [16. 실제 route table 예](#16-실제-route-table-예)
  - [🔥 실제로 확인하는 방법](#-실제로-확인하는-방법)
- [6장 아니 나는 Calico 의 IPIP Mode 라구요!](#6장-아니-나는-calico-의-ipip-mode-라구요)
  - [1️⃣ IPIP 모드란 무엇인가](#1️⃣-ipip-모드란-무엇인가)
  - [2️⃣ 왜 이런걸 사용하나](#2️⃣-왜-이런걸-사용하나)
  - [3️⃣ 실제 패킷 흐름](#3️⃣-실제-패킷-흐름)
    - [Step1 PodA 패킷 생성](#step1-poda-패킷-생성)
    - [Step2 Node routing](#step2-node-routing)
    - [Step3 IPIP encapsulation](#step3-ipip-encapsulation)
    - [Step4 물리 네트워크 이동](#step4-물리-네트워크-이동)
    - [Step5 Node2 도착](#step5-node2-도착)
    - [Step6 PodB 전달](#step6-podb-전달)
  - [4️⃣ 실제 리눅스 인터페이스](#4️⃣-실제-리눅스-인터페이스)
  - [5️⃣ tunl0 생성되는 방식](#5️⃣-tunl0-생성되는-방식)
  - [6️⃣ 실제 routing table](#6️⃣-실제-routing-table)
  - [7️⃣ 실제 패킷 확인](#7️⃣-실제-패킷-확인)
  - [8️⃣ Calico IPIP 모드 종류](#8️⃣-calico-ipip-모드-종류)
  - [9️⃣ Calico 현재 설정 확인](#9️⃣-calico-현재-설정-확인)
  - [🔟 IPIP vs VXLAN](#-ipip-vs-vxlan)
  - [🔥 Calico IPIP는 Linux kernel 기능](#-calico-ipip는-linux-kernel-기능)
- [7장 그렇다면 CNI 는 외부 node 와 통신할 때 어떻게 동작할까](#7장-그렇다면-cni-는-외부-node-와-통신할-때-어떻게-동작할까)
  - [1️⃣ Pod → Pod 통신에서 실제 흐름](#1️⃣-pod--pod-통신에서-실제-흐름)
  - [2️⃣ Calico IPIP가 하는 일](#2️⃣-calico-ipip가-하는-일)
  - [3️⃣ 실제 패킷 구조](#3️⃣-실제-패킷-구조)
  - [4️⃣ 그래서 방화벽에서 중요한 것](#4️⃣-그래서-방화벽에서-중요한-것)
  - [5️⃣ 패킷 흐름 전체](#5️⃣-패킷-흐름-전체)
  - [6️⃣ 실제 Linux 인터페이스](#6️⃣-실제-linux-인터페이스)
  - [7️⃣ 핵심 정리](#7️⃣-핵심-정리)
  - [8️⃣ Calico에는 3가지 모드가 있다.](#8️⃣-calico에는-3가지-모드가-있다)

# 1장 CNI 란 무엇인가

## 1️⃣ CNI가 무엇인가

CNI는 Container Network Interface 이다.

역할

```
컨테이너에 네트워크 연결을 만들어주는 표준 인터페이스
```

즉

```
컨테이너 생성
→ 네트워크 연결
→ IP 할당
→ 라우팅 설정
```

을 수행한다.

## 2️⃣ Kubernetes에서 왜 필요한가

컨테이너는 기본적으로 네트워크가 없다.

예

```
➜  docker run nginx
```

위의 커맨드 입력시 Docker가 자동으로

```
docker0 bridge
```

를 만들어 준다.

하지만 Kubernetes는 Docker에 의존하지 않기 때문에 container runtime만 있고 네트워크 구현은 없음
<br>그래서 Kubernetes는 이렇게 설계했다.

```
Container runtime → 컨테이너 실행
CNI plugin       → 네트워크 생성
```

## 3️⃣ Kubernetes Pod 생성 네트워크 흐름

Pod 생성 시 실제 흐름

```
Pod 생성 요청
 ↓
API Server
 ↓
Scheduler (Node 선택)
 ↓
kubelet
 ↓
container runtime
 ↓
pause container 생성
 ↓
network namespace 생성
 ↓
CNI 호출
```

여기서 CNI plugin 실행

## 4️⃣ CNI가 실제 하는 일

CNI plugin은 다음 작업을 한다.

```
1️⃣ veth pair 생성
2️⃣ 한쪽을 Pod namespace로 이동
3️⃣ Pod IP 할당 (IPAM, IP Address Manager)
4️⃣ 라우팅 설정
5️⃣ bridge 또는 overlay 연결
```

결과

```
Pod ↔ Node network 연결
```

## 5️⃣ 실제 Linux 구조

Pod 네트워크는 이렇게 생긴다.

```
Pod namespace
    │
    eth0
    │
    veth
    │
Node namespace
    │
cni bridge
    │
node network
```

예

```
[Pod]
eth0
  │
veth1234
  │
cni0
  │
eth0
[Node]
```

## 6️⃣ veth pair

CNI가 만드는 핵심

```
veth pair
```

```
veth는 virtual ethernet cable이다.
```

예

```
vethA  <──────>  vethB

로써 한쪽은 Pod namespace 에 다른 한쪽은 Host namespace 에 놓는다.
```

## 7️⃣ Pod 내부 구조

Pod namespace 안에서 보면 (nsenter cli tool이(가) 필요하다.)

```
ip a
```

결과 (예시)

```
eth0 10.244.1.3

이 eth0는 veth interface다.
```

## 8️⃣ CNI Plugin 구조

CNI는 plugin 시스템이다.

대표 CNI

```
Calico
Flannel
Weave
Cilium
Canal
Kube-router
```

Kubernetes는 CNI spec만 정의하고 실제 네트워크 구현은 plugin이 한다.

## 9️⃣ CNI Plugin 위치

보통

```
/opt/cni/bin
```

여기 있다.

예

```
ls /opt/cni/bin
```

결과

```
bridge
loopback
host-local
calico
flannel
```

## 10. CNI 설정 파일

설정 파일

```
/etc/cni/net.d/
```

예

```
10-calico.conflist
```

내용 예

```
{
  "cniVersion": "0.3.1",
  "name": "k8s-pod-network",
  "plugins": [
    {
      "type": "calico"
    }
  ]
}
```

kubelet이 이걸 읽는다.

## 11. CNI 호출 방식

kubelet이 CNI plugin 실행한다.

[예시]

```
/opt/cni/bin/calico {json...}
```

JSON을 전달한다.

## 12. CNI command

CNI Spec에는 3가지 기본 command가 있다.

| Command | 역할               |
| ------- | ------------------ |
| `ADD`   | Pod 네트워크 생성  |
| `DEL`   | Pod 네트워크 제거  |
| `CHECK` | 네트워크 상태 확인 |

### ADD

역할

```
Pod 네트워크 생성
```

즉

```
veth 생성
IP 할당
route 설정
iptables 설정
```

이 모든 것이 여기서 일어난다.

[ADD JSON 예시]

```
{
  "cniVersion": "0.4.0",
  "name": "k8s-pod-network",
  "type": "calico",
  "container_id": "abc123",
  "netns": "/proc/1234/ns/net",
  "ifname": "eth0",
  "args": {
    "K8S_POD_NAME": "nginx-pod",
    "K8S_POD_NAMESPACE": "default"
  },
  "ipam": {
    "type": "calico-ipam"
  }
}
```

### DEL

역할
Pod 삭제 시

```
veth 삭제
IP 반환
route 삭제
iptables 삭제
```

kubelet 흐름

```
Pod 삭제
 ↓
kubelet
 ↓
CNI_COMMAND=DEL
```

DEL JSON 예시

```
{
  "cniVersion": "0.4.0",
  "name": "k8s-pod-network",
  "type": "calico",
  "container_id": "abc123",
  "netns": "/proc/1234/ns/net",
  "ifname": "eth0"
}
```

실행 결과

삭제되는 것

```
caliabc123
route
iptables
IPAM lease
```

## CHECK

역할

```
네트워크 상태 검증
```

예

```
veth 존재 확인
IP 확인
route 확인
```

CHECK JSON 예시

```
{
  "cniVersion": "0.4.0",
  "name": "k8s-pod-network",
  "type": "calico",
  "container_id": "abc123",
  "netns": "/proc/1234/ns/net",
  "ifname": "eth0"
}
```

실제 확인

예

```
eth0 존재?
IP 맞음?
route 존재?
```

문제 있으면

```
error 반환
```

## 13. 실제 CNI 호출 흐름

```
kubelet
  │
  │ ADD
  ↓
CNI plugin 실행
  │
  ├ veth 생성
  ├ Pod IP 할당
  ├ route 추가
  └ bridge 연결
```

## 14. Pod IP는 누가 주는가

```
많은 사람들이 헷갈리는 것이 Pod IP는 누가 할당하냐인데 이것은 CNI plugin이 할당한다.
```

Calico

```
IPAM
```

이 IP 풀에서 할당.

[예시]

```
10.244.1.0/24
```

## 15. Kubernetes 네트워크 규칙

Kubernetes는 다음 규칙을 요구한다.

Pod Network Model

```
1️⃣ 모든 Pod은 고유 IP
2️⃣ Pod ↔ Pod NAT 없이 통신
3️⃣ Node 상관없이 Pod 통신
4️⃣ Pod 내부는 localhost 공유
```

```
그래서 CNI plugin이 overlay network를 만든다.
```

## 16. 실제 Calico 구조 (간단히)

Calico는

```
Pod veth
 ↓
Linux routing
 ↓
BGP or VXLAN
 ↓
다른 node Pod
```

으로 통신한다.

## 17. 🔥 지금까지 흐름 정리

Pod 생성 시 네트워크

```
Pod 생성
 ↓
pause container
 ↓
network namespace
 ↓
CNI ADD
 ↓
veth 생성
 ↓
Pod IP 할당
 ↓
route 설정
```

---

# 2장. Linux 기준으로 어떻게 실제로 생성하는가

## 1️⃣ veth pair란 무엇인가

```
veth = virtual ethernet pair

즉, 가상의 네트워크 케이블이다.
```

구조

```
vethA  <──────>  vethB
```

특징

```
한쪽으로 패킷 보내면
반대쪽에서 받는다
```

## 2️⃣ veth pair 생성 명령어

리눅스 명령어

```
➜  ip link add veth0 type veth peer name veth1
```

뜻

```
ip link add      → 인터페이스 생성
veth0            → 첫 번째 인터페이스 이름
type veth        → veth 타입
peer name veth1  → 반대편 인터페이스
```

결과 확인

```
➜  ip link
veth0@if3
veth1@if4
```

## 3️⃣ veth 생성은 어디 코드에서 하는가

CNI plugin 내부 코드에서 실행된다.

보통

```
/opt/cni/bin/
```

예

```
/opt/cni/bin/bridge
/opt/cni/bin/calico
```

이 바이너리가 실행되면서 내부적으로

```
netlink.LinkAdd() 같은 netlink syscall을 사용한다.

즉, 실제로는 ip link add와 동일한 커널 API를 호출한다.
```

## 4️⃣ Pod network namespace

Pod 생성 시 pause container가 먼저 생성된다. 즉, pause container가 network namespace를 가진다.

확인

```
➜  ls /proc/<pause-pid>/ns
```

예

```
net
ipc
pid
mnt
```

network namespace 확인

```
➜  ls -l /proc/<pause-pid>/ns/net
```

## 5️⃣ veth 한쪽을 Pod namespace로 이동

명령어

```
➜  ip link set veth1 netns <PID>
```

예시

```
➜  ip link set veth1 netns 3245
```

뜻

```
veth1을 PID 3245 namespace로 이동
```

이후 Pod 내부에서 보면

```
➜  ip a
```

결과

```
eth0
```

사실 이게 veth1이다.

## 6️⃣ Pod 내부 인터페이스 이름 변경

namespace 안에서

```
➜  ip netns exec <ns> ip link set veth1 name eth0

즉, veth1의 이름을 가진 veth 를 eth0로 명칭을 바꾼다.
```

그리고

```
➜  ip netns exec <ns> ip link set eth0 up

eth0 활성화
```

## 7️⃣ IPAM (Pod IP 할당)

```
IPAM = IP Address Management

Pod IP를 관리한다.
```

## 8️⃣ IPAM 설정파일 위치

보통 CNI config

```
/etc/cni/net.d/
```

예

```
10-calico.conflist
10-flannel.conflist
```

예시

```
{
 "cniVersion": "0.3.1",
 "name": "k8s-pod-network",
 "plugins": [
  {
   "type": "calico",
   "ipam": {
     "type": "calico-ipam"
   }
  }
 ]
}
```

## 9️⃣ host-local IPAM 예시

```
{
 "ipam": {
   "type": "host-local",
   "ranges": [
     [
       {
         "subnet": "10.244.1.0/24"
       }
     ]
   ]
 }
}
```

IP pool

```
10.244.1.0/24
```

## 10. IPAM 상태 저장 위치

host-local IPAM은

```
/var/lib/cni/networks/
```

예

```
/var/lib/cni/networks/cni0/
```

파일 예

```
10.244.1.2
10.244.1.3
10.244.1.4
```

파일 이름 자체가 할당된 IP이다.

## 11. Pod에 IP 할당

명령어

namespace 내부에서 실행

```
➜ ip addr add 10.244.1.5/24 dev eth0
```

확인

```
ip a
```

결과

```
eth0 10.244.1.5/24
```

## 12. 라우팅 생성

Pod namespace 내부

```
➜ ip route add default via 10.244.1.1
```

확인

```
➜ ip route
```

예

```
default via 10.244.1.1 dev eth0
```

## 13. Node 라우팅

host에서

```
➜ ip route
```

예

```
10.244.1.0/24 dev cni0
```

의미

```
Pod network → cni0 bridge
```

## 14. bridge란 무엇인가

```
bridge = software switch
Linux L2 switch다.
```

예

```
cni0
docker0
```

확인

```
➜ ip link show type bridge
```

또는

```
➜  bridge link
```

## 15. bridge 생성 명령어

bridge 생성

```
➜  ip link add cni0 type bridge
```

활성화

```
➜  ip link set cni0 up
```

IP 할당

```
➜  ip addr add 10.244.1.1/24 dev cni0
```

## 16. veth와 bridge 연결

명령어

```
➜  ip link set veth0 master cni0
```

즉

```
veth0 → cni0 bridge
```

구조

```
Pod eth0
   │
veth1
   │
veth0
   │
cni0
   │
node network
```

## 17. overlay network

Node 간 Pod 통신용 네트워크

예시

```
VXLAN
IPIP
Geneve
```

## 18. VXLAN 인터페이스 생성

명령어

```
➜  ip link add vxlan.calico type vxlan id 100 dev eth0 dstport 4789
```

확인

```
➜  ip link
```

예

```
vxlan.calico
```

## 19. overlay 패킷 흐름

예

```
pod1 → node1
node1 → vxlan encapsulation
node2 → decapsulation
pod2
```

패킷

```
inner packet = pod IP
outer packet = node IP
```

## 20. 전체 구조

Pod 네트워크 구조

```
Pod namespace
   │
   eth0
   │
   veth
   │
Node namespace
   │
   cni0 bridge
   │
   vxlan.calico
   │
Node network
   │
다른 Node
```

# 3장 Calico 전체 아키텍처

## 1️⃣ Calico 전체 아키텍처

Calico는 크게 4개의 주요 컴포넌트로 구성된다.

```
Kubernetes Node

 ┌──────────────────────────┐
 │ kubelet                  │
 │ container runtime        │
 └──────────┬───────────────┘
            │
            │ CNI call
            ▼
        Calico CNI
            │
            ▼
         Felix
            │
     ┌──────┴────────┐
     │               │
 routing table     iptables
     │               │
     ▼               ▼
 Linux Kernel Networking
     │
     ▼
 tunl0 / vxlan.calico / eth0
```

```
추가로 cluster 레벨에는

Typha
BIRD

가 있다.
```

## 2️⃣ Calico 주요 컴포넌트

| 컴포넌트   | 역할                   |
| ---------- | ---------------------- |
| CNI plugin | Pod 네트워크 생성      |
| Felix      | 라우팅 + iptables 관리 |
| BIRD       | BGP 라우팅             |
| Typha      | API server 부하 감소   |

## 3️⃣ Calico CNI Plugin

Pod 생성 시 kubelet이 실행한다.

흐름

```
Pod 생성
   ↓
kubelet
   ↓
CNI plugin 실행
   ↓
Calico CNI
```

CNI가 하는 일

```
1. veth pair 생성
2. Pod network namespace 연결
3. Pod IP 할당
4. route 설정
```

예

```
pod
  eth0
    ↓
veth
    ↓
host interface
```

그리고

```
10.244.1.5
```

같은 Pod IP를 설정한다.

## 4️⃣ Felix (Calico의 핵심)

Felix는 Calico node agent다.

DaemonSet으로 모든 node에 실행된다.

확인

```
➜  kubectl get pods -n kube-system | grep calico
```

예

```
calico-node-abcde
calico-node-fghij
```

이 Pod 안에 felix가 있다.

### Felix의 역할

```
1️⃣ routing table 관리
2️⃣ iptables rule 관리
3️⃣ network policy 적용
4️⃣ tunnel interface 관리
```

```
즉, Pod networking의 실제 executor이다.
```

### Felix가 하는 일 (가장 중요)

예

```
node1 PodCIDR = 10.244.1.0/24
node2 PodCIDR = 10.244.2.0/24
```

#### Felix는 routing table을 만든다.

확인

```
➜  ip route
10.244.2.0/24 via 192.168.10.12 dev tunl0
```

즉

```
node2 pod network → tunl0
```

이걸 Felix가 만든다.

#### Felix는 iptables도 만든다.

예

```
iptables -t nat -L
또는
iptables -L
```

Calico 체인

```
cali-xxxxx
```

예

```
cali-FORWARD
cali-INPUT
cali-OUTPUT
```

## 6️⃣ BIRD (BGP daemon)

Calico의 또 다른 핵심.

BIRD는 BGP routing daemon이다.

역할

```
Node 간 Pod CIDR 광고
```

예

```
node1
10.244.1.0/24

node2
10.244.2.0/24
```

node1이 BGP로 광고

```
10.244.1.0/24 reachable via node1
```

node2도 광고

```
10.244.2.0/24 reachable via node2
```

그래서 라우팅 테이블이 만들어진다.

확인

```
➜  calicoctl node status
```

또는

```
birdcl show route
```

## 7️⃣ Typha

Typha는 대규모 cluster에서 사용한다.

### Felix 의 문제점

Felix → API Server watch node가 많으면

```
1000 nodes
→ 1000 watch
```

API server 부담.

그래서

```
API Server
    ↓
Typha
    ↓
Felix들
```

즉, watch proxy다.

## 8️⃣ Calico 네트워크 모드

| 모드  | 설명               |
| ----- | ------------------ |
| BGP   | encapsulation 없음 |
| IPIP  | L3 tunnel          |
| VXLAN | L2 over UDP        |

IPIP

```
Pod packet
   ↓
IPIP encapsulation
   ↓
node transport
```

VXLAN

```
UDP 4789
```

BGP

```
direct routing
```

## 9️⃣ Pod 생성 시 실제 흐름

전체 흐름

```
Pod 생성
   ↓
kubelet
   ↓
CNI plugin 호출
   ↓
Calico CNI
   ↓
veth 생성
   ↓
Pod IP 할당
   ↓
Felix
   ↓
route + iptables 설정
   ↓
Pod 네트워크 활성화
```

## 🔥 Calico 구조 핵심 정리

```
CNI
  ↓
Pod interface 생성

Felix
  ↓
iptables + routing

BIRD
  ↓
BGP route exchange

Typha
  ↓
watch scaling
```

---

# 4장 Calico 는 보통 host-ipam 을 사용하지 않는다.

### 1️⃣ /var/lib/cni/networks 는 언제 사용되는가

이 경로는 보통 host-local IPAM이 사용한다.

예 CNI 설정

```
{
  "ipam": {
    "type": "host-local",
    "subnet": "10.244.1.0/24"
  }
}
```

이 경우

IP 할당 상태를

```
/var/lib/cni/networks/<network-name>/
```

에 저장한다.

예시

```
/var/lib/cni/networks/cni0/
  10.244.1.2
  10.244.1.3
  10.244.1.4
```

파일 이름 자체가 할당된 IP다.

## 2️⃣ Calico는 보통 host-local IPAM을 안 쓴다

Helm으로 설치한 Calico는 보통

```
calico-ipam
```

을 사용한다.

CNI 설정 확인해보면

```
/etc/cni/net.d/
```

여기에 있다.

확인

```
➜  ls /etc/cni/net.d/
```

보통

```
10-calico.conflist
```

열어보면 대략 이런 형태다.

```
➜  cat /etc/cni/net.d/10-calico.conflist

{
  "name": "k8s-pod-network",
  "cniVersion": "0.3.1",
  "plugins": [
    {
      "type": "calico",
      "ipam": {
        "type": "calico-ipam"
      }
    }
  ]
}
```

여기서 중요한 것

```
"type": "calico-ipam"
```

## 3️⃣ calico-ipam 은 어디에 저장하나

Calico는 IP 정보를 Kubernetes API datastore에 저장한다.

즉

```
etcd
또는
Kubernetes API

에 저장된다.
```

실제 리소스

```
IPAMBlock
IPAMHandle
IPAMConfig
```

확인 가능

```
kubectl get ipamblocks.crd.projectcalico.org
```

또는

```
kubectl get ipamblocks -A
```

## 4️⃣ Calico IP 할당 구조

Calico는 이렇게 한다.

```
Node CIDR
   ↓
IPAM block
   ↓
Pod IP
```

예

```
Node CIDR
10.244.1.0/24
```

Calico가 block을 만든다.

```
10.244.1.0/26
10.244.1.64/26
10.244.1.128/26
10.244.1.192/26
```

그리고 Pod에게 할당

```
10.244.1.2
10.244.1.3
10.244.1.4
```

## 5️⃣ 실제 확인 명령어

Pod IP 확인

```
➜  kubectl get pods -o wide
```

Calico block 확인

```
➜  kubectl get ipamblocks.crd.projectcalico.org
또는
➜  kubectl get ipamblocks
```

## 6️⃣ Calico가 host-local을 안쓰는 이유

host-local IPAM 문제는 다음과 같다.

```
Node local state

즉, Node 장애나면 IP 상태 유실이 가능하다.
```

Calico는 cluster-wide IPAM을 쓴다.

```
그래서 IP 충돌 방지와 Node간 IP 관리가 가능하다.
```

## 7️⃣ 실제 Calico 관련 파일 위치

CNI binary

```
/opt/cni/bin/calico
```

CNI config

```
/etc/cni/net.d/10-calico.conflist
```

Calico node agent 명칭

```
calico-node (DaemonSet)
```

확인

```
kubectl get pods -n kube-system
```

## 8️⃣ Calico 실제 네트워크 인터페이스

Node에서 확인

```
➜  ip link
```

보통 이런게 보인다.

```
caliXXXXXXXX
vxlan.calico
tunl0
```

설명

```
caliXXXX → Pod veth
vxlan.calico → overlay network
tunl0 → IPIP tunnel
```

## 9️⃣ Pod veth 이름

Calico는 caliXXXXX형태로 만든다.

예

```
cali9a7c2f3b1d
```

확인

```
➜  ip link | grep cali
```

## 10. Calico의 Pod IP 할당 흐름

Pod IP 할당 흐름이 이렇게 된다

```
kubelet
   ↓
CNI plugin (/opt/cni/bin/calico)
   ↓
calico-node agent 호출
   ↓
Kubernetes API
   ↓
IPAM block 조회
   ↓
IP 할당
```

---

# 5장 Calico가 Pod 생성시 실제로 하는 일

## 1️⃣ Pod 생성 → kubelet → CNI 호출

Pod 생성 흐름

```
kubectl apply
     ↓
API Server
     ↓
Scheduler (Node 선택)
     ↓
kubelet
     ↓
container runtime
     ↓
pause container 생성
     ↓
CNI ADD 호출
```

kubelet이 실제로 실행하는 것

```
/opt/cni/bin/calico
```

설정파일

```
/etc/cni/net.d/10-calico.conflist
```

## 2️⃣ CNI가 받는 정보

CNI는 JSON을 stdin으로 받는다.

예

```
{
  "container_id": "abc123",
  "netns": "/proc/1234/ns/net",
  "ifName": "eth0",
  "args": {
    "K8S_POD_NAME": "nginx-123",
    "K8S_POD_NAMESPACE": "default"
  }
}
```

핵심

```
netns = /proc/<pid>/ns/net
```

즉 Pod network namespace 위치와 기타 등을 준다.

## 3️⃣ Calico IPAM 동작

Calico는 IP를 여기서 가져온다.

```
Kubernetes datastore
또는
etcd
```

리소스

```
IPAMBlock
```

예

```
10.244.1.0/26
```

Pod IP 할당

```
10.244.1.5
```

확인 가능

```
➜  kubectl get ipamblocks
```

## 4️⃣ veth pair 생성

Calico가 netlink로 생성한다.

Linux 명령으로 표현하면

```
➜  ip link add cali1234 type veth peer name eth0
```

실제 구조

```
Node namespace
   cali1234
      │
      │ veth pair
      │
Pod namespace
      eth0
```

## 5️⃣ veth 이동

Pod namespace로 이동

```
➜  ip link set eth0 netns <pause-pid>
```

확인

```
➜  lsns -t net
```

Pod namespace에서 보면

```
➜  ip a
```

결과

```
eth0@if23
10.244.1.5/32
```

## 6️⃣ Pod IP 설정

Calico가 Pod namespace 안에서 실행

```
➜  ip addr add 10.244.1.5/32 dev eth0
```

중요한 점

```
/32
```

를 사용한다. 왜냐하면 Calico는 L3 routing 모델이다.

## 7️⃣ Pod default route

namespace 내부

```
➜  ip route add default dev eth0
```

Pod routing table

```
default dev eth0
```

## 8️⃣ Node 쪽 인터페이스

Node에서 보면

```
➜  ip link
```

예

```
cali7a9c1f3
```

이게 Pod의 veth host side다.

## 9️⃣ Node routing table

Node에서

```
➜  ip route
```

예

```
10.244.1.5 dev cali7a9c1f3
```

의미

```
Pod IP → cali interface
```

즉 Node가 Pod IP routing table을 가지고 있다.

## 🔟 Pod → Pod (같은 Node) 패킷 흐름

패킷 흐름

```
Pod1
  eth0
   │
veth
   │
cali interface
   │
Linux routing
   │
cali interface
   │
veth
   │
Pod2
```

즉 Linux router가 forwarding

## 11. Pod → Pod (다른 Node) 패킷 흐름

예

```
pod1 (node1)
pod2 (node2)
```

Node1 routing

```
10.244.2.0/24 via vxlan.calico
```

패킷 흐름

```
pod1
 ↓
eth0
 ↓
cali interface
 ↓
node1 routing
 ↓
vxlan.calico
 ↓
encapsulation
 ↓
node2
 ↓
vxlan decapsulation
 ↓
cali interface
 ↓
pod2
```

## 12. vxlan interface

Node에서 확인

```
➜  ip link show vxlan.calico
```

예

```
vxlan.calico@NONE
```

VXLAN port

```
UDP 4789
```

## 13. VXLAN FDB

확인

```
➜  bridge fdb show dev vxlan.calico
```

예

```
aa:bb:cc:dd dst 192.168.0.12
```

의미

```
remote node mac
```

## 14. Calico routing 방식

Calico는 두 가지 모드

BGP mode

```
Node route exchange
```

VXLAN mode

```
overlay network
```

가 있으며

Helm 기본은 보통

```
VXLAN
```

으로 동작한다.

## 15. 실제 Node 인터페이스들

Node에서 보면 보통 이것들이 있다.

```
➜  ip link
```

예

```
caliXXXXX
vxlan.calico
eth0
lo
```

설명

```
caliXXXXX → Pod veth
vxlan.calico → overlay tunnel
eth0 → node NIC
```

## 16. 실제 route table 예

```
➜  ip route
```

예

```
10.244.1.5 dev cali7a9c1f3
10.244.2.0/24 via vxlan.calico
```

## 🔥 실제로 확인하는 방법

Pod namespace

```
➜  lsns -t net
```

➜ Pod route

```
➜  nsenter -t <pid> -n ip route
```

➜ Node route

```
➜  ip route
```

veth 확인

```
➜  ip link | grep cali
```

vxlan 확인

```
➜  ip -d link show vxlan.calico
```

---

# 6장 아니 나는 Calico 의 IPIP Mode 라구요!

```
ip link
```

에서

```
➜  tunl0
```

가 보였다면 거의 확실히 Calico IPIP 모드가 켜져 있는 상태다.

## 1️⃣ IPIP 모드란 무엇인가

IPIP는 IP-in-IP 터널링이다.

정식 이름은

```
IP-in-IP
```

의미는

```
IP 패킷 안에
또 다른 IP 패킷을 넣는다
```

구조

```
Outer IP Header (Node IP)
   ↓
Inner IP Header (Pod IP)
   ↓
Payload
```

즉

```
Node ↔ Node 통신으로 위장해서
Pod ↔ Pod 통신을 하는 방식
```

## 2️⃣ 왜 이런걸 사용하나

쿠버네티스 Pod 네트워크 목표

```
모든 Pod는 서로 직접 통신 가능해야 한다
```

하지만 실제 네트워크는

```
Node1
  PodA (10.244.1.2)

Node2
  PodB (10.244.2.3)
```

일반 네트워크는

```
10.244.x.x
```

같은 Pod CIDR을 모른다.

그래서 Node1 → Node2로 패킷을 보내면서 Pod IP를 숨겨야 한다.

그래서

```
IPIP tunnel
```

을 사용한다.

## 3️⃣ 실제 패킷 흐름

예시

```
PodA → PodB
```

```
PodA IP = 10.244.1.2
PodB IP = 10.244.2.3
```

Node IP

```
Node1 = 192.168.0.10
Node2 = 192.168.0.11
```

### Step1 PodA 패킷 생성

```
SRC 10.244.1.2
DST 10.244.2.3
```

### Step2 Node routing

Node routing table

```
10.244.2.0/24 via tunl0
```

확인

```
ip route
```

예

```
10.244.2.0/24 via 192.168.0.11 dev tunl0
```

### Step3 IPIP encapsulation

Node1 kernel이 패킷을 바꾼다.

원래 패킷

```
SRC 10.244.1.2
DST 10.244.2.3
```

IPIP 적용 후

```
Outer IP
SRC 192.168.0.10
DST 192.168.0.11

Inner IP
SRC 10.244.1.2
DST 10.244.2.3
```

### Step4 물리 네트워크 이동

실제 네트워크는

```
192.168.0.10 → 192.168.0.11
```

으로 보인다.

그래서 라우터가 이해 가능하다.

### Step5 Node2 도착

Node2 kernel

```
tunl0
```

인터페이스가 패킷을 받는다. 그리고 decapsulation을 한다.

Outer IP 제거

```
SRC 10.244.1.2
DST 10.244.2.3
```

### Step6 PodB 전달

라우팅

```
10.244.2.3 → veth
```

PodB 전달

## 4️⃣ 실제 리눅스 인터페이스

확인

```
➜  ip link show tunl0
```

예

```
tunl0@NONE: <NOARP,UP>
```

설명

```
IPIP tunnel interface
```

## 5️⃣ tunl0 생성되는 방식

Calico agent

```
calico-node
```

DaemonSet이 만든다.

확인

```
➜  kubectl get ds -n kube-system
```

여기서 calico-node Pod가 있다.

이 안에서 felix라는 프로세스가

```
route
tunnel
iptables
```

전부 설정한다.

## 6️⃣ 실제 routing table

확인

```
➜  ip route
```

예

```
10.244.1.0/24 dev cali123 scope link
10.244.2.0/24 via 192.168.0.11 dev tunl0
```

설명

```
같은 노드 Pod → cali interface
다른 노드 Pod → tunl0
```

## 7️⃣ 실제 패킷 확인

캡쳐

```
➜  tcpdump -i tunl0
```

그러면 IPIP packets 보인다.

예

```
IP 192.168.0.10 > 192.168.0.11: IPIP
```

## 8️⃣ Calico IPIP 모드 종류

Calico 설정

```
IPIPMode
```

옵션

```
Always
CrossSubnet
Never
```

설명

```
Always
모든 노드 IPIP

CrossSubnet
서브넷 다르면 IPIP

Never
IPIP 없음
```

## 9️⃣ Calico 현재 설정 확인

Calico IPPool 확인

```
➜  kubectl get ippool -o yaml
```

여기 보면 ipipMode가 있다.

예

```
ipipMode: Always
```

## 🔟 IPIP vs VXLAN

Calico는 두가지 overlay 지원한다.

| 방식  | 특징       |
| ----- | ---------- |
| IPIP  | L3 tunnel  |
| VXLAN | L2 overlay |

VXLAN 인터페이스

```
vxlan.calico
```

IPIP 인터페이스

```
tunl0
```

## 🔥 Calico IPIP는 Linux kernel 기능

Calico IPIP는 Linux kernel 기능이다.

즉

```
ip tunnel
```

로 생성 가능하다.

예

```
ip tunnel add tunl0 mode ipip
```

---

# 7장 그렇다면 CNI 는 외부 node 와 통신할 때 어떻게 동작할까

질문

```
한가지 궁금한게 생겼어 calico ipipmode 라고 치고,
node1 의 pod1 에서 node2의 pod2 로 통신을 할건데
ipipmode 는 layer 3 에서 동작하고 그러면
실제 애플리케이션에서 통신할 때 port 는 필요없이 ip 만 있으면 되는 거야?
```

결론부터 말하면

```
IPIP (Calico IPIP mode)는 L3 터널이기 때문에 통신 자체에는 port가 필요 없다.
하지만 실제 애플리케이션 통신은 여전히 L4 port를 사용한다.
```

즉 두 레이어를 분리해서 봐야 한다.

## 1️⃣ Pod → Pod 통신에서 실제 흐름

상황

```
node1
  pod1 (10.244.1.2)

node2
  pod2 (10.244.2.3)
```

pod1이 pod2로 요청

```
10.244.1.2 → 10.244.2.3
```

예

```
curl 10.244.2.3:8080
```

여기서 port 8080은 애플리케이션(TCP/UDP) 이다.

## 2️⃣ Calico IPIP가 하는 일

IPIP는 Pod 네트워크를 Node 간에 전달하는 터널이다.

즉 Pod IP packet을 Node IP packet 안에 넣어서 전송한다.

구조

```
Outer IP header (node IP)
    ↓
Inner IP header (pod IP)
    ↓
TCP / UDP
```

예

```
Outer IP
src = node1
dst = node2

Inner IP
src = 10.244.1.2
dst = 10.244.2.3

TCP
dst port = 8080
```

## 3️⃣ 실제 패킷 구조

node1 → node2

```
IP (node1 → node2)
  protocol = IPIP
    ↓
IP (10.244.1.2 → 10.244.2.3)
    ↓
TCP
  dst port = 8080
```

여기서 중요한 것

IPIP 자체는 port가 없다.

왜냐하면

```
IP protocol number = 4
```

이다.

즉

```
TCP (6)
UDP (17)
IPIP (4)
```

처럼 IP protocol 필드로 구분한다.

## 4️⃣ 그래서 방화벽에서 중요한 것

```
Node 간 통신에서 필요한 것은 IP protocol 4 허용이다.
```

예

```
iptables

-A INPUT -p 4 -j ACCEPT
```

```
여기서 -p 4는 IPIP protocol이다.

즉, port 없음
```

## 5️⃣ 패킷 흐름 전체

pod1 → pod2

```
pod1
 10.244.1.2
    ↓
veth
    ↓
node1 routing
    ↓
tunl0 (ipip tunnel)
    ↓
encapsulation
    ↓
node1 eth0
    ↓
internet / network
    ↓
node2 eth0
    ↓
tunl0 decap
    ↓
routing
    ↓
veth
    ↓
pod2
```

## 6️⃣ 실제 Linux 인터페이스

Calico IPIP 사용하면 node에 생긴다.

```
tunl0
```

확인

```
➜  ip a
```

예

```
tunl0: <POINTOPOINT,NOARP>
```

라우팅 확인

```
➜  ip route
```

예

```
10.244.2.0/24 via 192.168.10.2 dev tunl0
```

## 7️⃣ 핵심 정리

| 구분                       | port 필요? |
| -------------------------- | ---------- |
| IPIP 터널                  | ❌ 없음    |
| Pod 애플리케이션 (TCP/UDP) | ✅ 필요    |

즉

```
TCP/UDP port
   ↓
Pod network packet
   ↓
IPIP encapsulation
   ↓
Node IP transport
```

## 8️⃣ Calico에는 3가지 모드가 있다.

```
IPIP
VXLAN
BGP (no encapsulation)
```

차이

| 모드  | encapsulation |
| ----- | ------------- |
| IPIP  | L3            |
| VXLAN | L2 over UDP   |
| BGP   | 없음          |

VXLAN이면

```
UDP 4789 port 사용
```

하지만 IPIP 이면

```
port 없음
IP protocol 4
```
