## 목차

- [1장 kubelet 이란?](#1장-kubelet-이란)
  - [역할](#역할)
  - [⚙️ 전체 구조](#️-전체-구조)
  - [🔥 kubelet 전체 동작 흐름](#-kubelet-전체-동작-흐름)
  - [💥 kubelet 특징](#-kubelet-특징)
  - [🧠 kubelet이 Pod를 받는 방법](#-kubelet이-pod를-받는-방법)
  - [📦 kubelet 내부 처리 단계](#-kubelet-내부-처리-단계)
    - [1️⃣ Pod Sandbox 생성 (매우 중요🔥)](#1️⃣-pod-sandbox-생성-매우-중요)
    - [2️⃣ 네트워크 생성 (CNI)](#2️⃣-네트워크-생성-cni)
    - [3️⃣ container runtime 호출](#3️⃣-container-runtime-호출)
    - [4️⃣ 컨테이너 실행](#4️⃣-컨테이너-실행)
    - [5️⃣ 상태 보고](#5️⃣-상태-보고)
  - [📡 kubelet ↔ API Server 통신](#-kubelet--api-server-통신)
  - [kubelet이 관리하는 것](#kubelet이-관리하는-것)
- [2장 Linux에서 실제 kubelet 동작](#2장-linux에서-실제-kubelet-동작)
  - [프로세스 확인](#프로세스-확인)
  - [실행 파일 위치](#실행-파일-위치)
  - [설정 파일 위치](#설정-파일-위치)
- [3장 Pod 생성 시 리눅스 커널 레벨에서 실제로 일어나는 일](#3장-pod-생성-시-리눅스-커널-레벨에서-실제로-일어나는-일)
  - [📌흐름 및 핵심 정리](#흐름-및-핵심-정리)
  - [1️⃣ Pod 생성 요청](#1️⃣-pod-생성-요청)
  - [2️⃣ Pod Sandbox 생성 (pause container)](#2️⃣-pod-sandbox-생성-pause-container)
  - [3️⃣ Network Namespace 생성](#3️⃣-network-namespace-생성)
  - [4️⃣ veth pair 생성](#4️⃣-veth-pair-생성)
  - [5️⃣ veth pair 연결 구조](#5️⃣-veth-pair-연결-구조)
  - [6️⃣ bridge 연결](#6️⃣-bridge-연결)
  - [7️⃣ Pod IP 할당](#7️⃣-pod-ip-할당)
  - [8️⃣ routing 설정](#8️⃣-routing-설정)
  - [9️⃣ 실제 패킷 흐름](#9️⃣-실제-패킷-흐름)
  - [🔟 Pod ↔ Pod 통신](#-pod--pod-통신)
  - [중요한 리눅스 기술](#중요한-리눅스-기술)

---

# 1장 kubelet 이란?

🔹 한 줄 정의

각 노드에서 실행되며 Pod를 실제로 실행하고 관리하는 에이전트

kubelet은 노드에서 PodSpec을 받아 container runtime과 CNI를 호출하여 실제 Pod를 실행하고 상태를 관리하는 에이전트다

### 역할

| 역할          | 설명                 |
| ------------- | -------------------- |
| Pod 실행      | 컨테이너 실행        |
| 상태 보고     | Node 상태 / Pod 상태 |
| Runtime 호출  | containerd / CRI     |
| 볼륨 관리     | mount                |
| 네트워크 준비 | CNI 호출             |

### ⚙️ 전체 구조

```
Control Plane
     ↓
kube-apiserver
     ↓
kubelet (node)
     ↓
container runtime
     ↓
containers
```

### 🔥 kubelet 전체 동작 흐름

```
Pod 생성부터 실행까지 실제 흐름이다.

1 Pod 생성 (kubectl)
2 API Server 저장 (etcd)
3 scheduler node 선택
4 kubelet watch
5 Pod spec 수신
6 container runtime 호출
7 CNI 네트워크 생성
8 container 실행
9 상태 보고
```

### 💥 kubelet 특징

```
✔️ 노드마다 하나
node1 → kubelet
node2 → kubelet

✔️ 직접 etcd 접근 없음
kubelet → apiserver → etcd

✔️ self healing
container 죽으면
restart
```

### 🧠 kubelet이 Pod를 받는 방법

```
📦 watch
kubelet은 API Server를 계속 감시한다.

조건:
spec.nodeName == 자기 노드

예:
spec:
  nodeName: node1

node1의 kubelet이 실행한다.
```

### 📦 kubelet 내부 처리 단계

```
Pod를 받으면 다음 순서로 처리한다.

1 PodSpec 확인
2 볼륨 준비
3 네트워크 준비
4 container runtime 호출
5 container 실행
```

#### 1️⃣ Pod Sandbox 생성 (매우 중요🔥)

첫 번째로 생성되는 것은 pause container

```
[pause container 역할]
Pod의 network namespace 유지

구조:

Pod
├ pause
├ container1
└ container2

모든 container는
pause network namespace 공유
```

#### 2️⃣ 네트워크 생성 (CNI)

```
kubelet은 CNI plugin 호출

kubelet
   ↓
CNI plugin
   ↓
veth 생성
   ↓
Pod IP 할당

결과:
Pod IP 생성
```

#### 3️⃣ container runtime 호출

```
kubelet은 직접 container 실행 안 한다.

사용하는 인터페이스:
CRI (Container Runtime Interface)

예:
kubelet
   ↓
CRI
   ↓
containerd
   ↓
runc
```

#### 4️⃣ 컨테이너 실행

```
container runtime이 실제 실행한다.

image pull
container create
container start
```

#### 5️⃣ 상태 보고

```
kubelet은 API Server에 상태 전달

Running
CrashLoopBackOff
Completed
```

### 📡 kubelet ↔ API Server 통신

```
양방향이다.

[kubelet → apiserver]
Pod 상태
Node 상태

[apiserver → kubelet]
Pod spec 전달
```

### kubelet이 관리하는 것

| 대상        | 설명      |
| ----------- | --------- |
| Pod         | 실행      |
| Container   | lifecycle |
| Volume      | mount     |
| Network     | CNI       |
| Node status | heartbeat |

---

# 2장 Linux에서 실제 kubelet 동작

### 프로세스 확인

```
ps -ef | grep kubelet

또는

systemctl status kubelet
```

### 실행 파일 위치

```
/usr/bin/kubelet
```

### 설정 파일 위치

```
/var/lib/kubelet/config.yaml

또는

/etc/kubernetes/kubelet.conf
```

---

# 3장 Pod 생성 시 리눅스 커널 레벨에서 실제로 일어나는 일

### 📌흐름 및 핵심 정리

```
Pod 생성 시 리눅스 내부에서는 다음이 발생한다:
1 pause container 생성
2 network namespace 생성
3 veth pair 생성
4 veth host ↔ bridge 연결
5 veth pod ↔ pod namespace 연결
6 Pod IP 할당
7 routing 설정
```

### 1️⃣ Pod 생성 요청

```
예:
kubectl run nginx --image=nginx

흐름:
kubectl
→ kube-apiserver
→ etcd 저장
→ scheduler node 선택
→ kubelet 감지

이제 kubelet이 Pod 실행 시작한다.
```

### 2️⃣ Pod Sandbox 생성 (pause container)

```
먼저 pause container가 만들어진다.

구조:
Pod
├ pause
├ nginx
└ sidecar

pause container 역할:
Pod의 namespace 유지
특히 network namespace를 담당한다.
```

### 3️⃣ Network Namespace 생성

```
컨테이너는 리눅스 namespace로 격리된다.

Pod 생성 시:
새 network namespace 생성

리눅스에서 보면:
/proc/<pid>/ns/net

예시:
host network namespace
Pod network namespace

각 namespace는

IP
routing table
iptables
network device

전부 독립적이다.
```

### 4️⃣ veth pair 생성

```
이제 Pod를 host 네트워크에 연결해야 한다.
그래서 사용하는 것이 veth pair.

개념:
가상 이더넷 케이블

구조:
veth0 <----> veth1

패킷 흐름:
veth0 → veth1
veth1 → veth0
```

### 5️⃣ veth pair 연결 구조

Pod 생성 시 이런 구조가 만들어진다.

```
Host Network Namespace
────────────────────────────

      linux bridge
          |
        veth-host
          |

────────────────────────────
Pod Network Namespace

        veth-pod
          |
        Pod

veth pair 중

veth-host → host namespace
veth-pod → pod namespace
```

### 6️⃣ bridge 연결

```
host에는 보통 이런 bridge가 있다.

예 (CNI):
cni0

구조:
Pod
|
veth
|
cni0 bridge
|
node network

bridge 역할:
Pod 간 L2 통신
```

### 7️⃣ Pod IP 할당

```
이제 CNI plugin이 IP를 할당한다.

예:
Pod IP = 10.244.1.5

이 IP는
Pod network namespace 내부 인터페이스에 설정된다.

리눅스 내부:
ip addr

보면:
eth0@if123
10.244.1.5/24
```

### 8️⃣ routing 설정

```
Pod 내부 routing table:
default via 10.244.1.1

10.244.1.1은 보통 node bridge gateway 이다.
```

### 9️⃣ 실제 패킷 흐름

```
[Pod → 외부]

Pod container
↓
eth0
↓
veth-pod
↓
veth-host
↓
cni0 bridge
↓
node network
↓
외부
```

### 🔟 Pod ↔ Pod 통신

```
[같은 노드인 경우]
Pod1
↓
veth
↓
bridge
↓
veth
↓
Pod2

라우터 필요 없음.
```

### 중요한 리눅스 기술

Pod 네트워크는 다음 기술로 구성된다.
| 기술 | 역할 |
| ----------------- | -------- |
| Network Namespace | 네트워크 격리 |
| veth pair | 네트워크 연결 |
| Linux Bridge | L2 스위치 |
| iptables | NAT / 정책 |
| routing | 패킷 전달 |
