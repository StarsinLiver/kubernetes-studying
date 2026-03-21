## 목차

- [1장 CRI (Container Runtime Interface)](#1장-cri-container-runtime-interface)
  - [🔥 핵심 개념](#-핵심-개념)
  - [전체 구조](#전체-구조)
  - [왜 CRI가 필요했나](#왜-cri가-필요했나)
  - [CRI가 하는 일](#cri가-하는-일)
  - [실제 통신 방식](#실제-통신-방식)
  - [실제 runtime 종류](#실제-runtime-종류)
  - [실제 Pod 생성 시 CRI 흐름](#실제-pod-생성-시-cri-흐름)
  - [CRI 위치 (전체 아키텍처)](#cri-위치-전체-아키텍처)
  - [Dockershim 제거](#dockershim-제거)
- [2장 CRI 내부 서비스](#2장-cri-내부-서비스)
  - [1️⃣ Runtime Service](#1️⃣-runtime-service)
  - [2️⃣ Image Service](#2️⃣-image-service)
- [3장 런타임 내부구조](#3장-런타임-내부구조)
  - [흐름](#흐름)
  - [1️⃣ container runtime: containerd](#1️⃣-container-runtime-containerd)
  - [2️⃣ OCI Runtime](#2️⃣-oci-runtime)
  - [3️⃣ 왜 containerd-shim이 필요한가](#3️⃣-왜-containerd-shim이-필요한가)
  - [4️⃣ 실제 Pod 실행 흐름](#4️⃣-실제-pod-실행-흐름)
  - [5️⃣ 실제 프로세스 구조](#5️⃣-실제-프로세스-구조)
  - [6️⃣ 컨테이너 내부 기술](#6️⃣-컨테이너-내부-기술)
  - [7️⃣ namespace 종류](#7️⃣-namespace-종류)
  - [8️⃣ cgroup (리소스 제한)](#8️⃣-cgroup-리소스-제한)
  - [9️⃣ 전체 실행 흐름 (정리)](#9️⃣-전체-실행-흐름-정리)

---

# 1장 CRI (Container Runtime Interface)

🔹 한 줄 정의

kubelet이 컨테이너 런타임과 통신하기 위한 표준 인터페이스

### 🔥 핵심 개념

```
❗ kubelet은 컨테이너를 직접 실행하지 않는다
→ CRI를 통해 runtime에게 실행을 요청한다
```

### 전체 구조

```
kubelet
│
│ CRI (gRPC API)
▼
container runtime
│
▼
OCI runtime
│
▼
Linux kernel
```

```
[예시]
kubelet
  ↓
CRI
  ↓
containerd
  ↓
runc
  ↓
namespaces / cgroups
```

### 왜 CRI가 필요했나

```
초기 쿠버네티스 구조는 이랬다.

kubelet
↓
Docker API
↓
Docker

문제:
Docker에 강하게 의존
다른 runtime 사용 어려움
Kubernetes 확장성 제한

그래서 만든 것이 CRI 즉, runtime 교체 가능
```

### CRI가 하는 일

CRI는 kubelet과 runtime 사이의 통신 규칙(API) 이다.
| 기능 | 설명 |
| ----------- | ------------ |
| Pod Sandbox | Pod 환경 생성 |
| Container | container 생성 |
| Image | 이미지 관리 |
| Status | 상태 조회 |

### 실제 통신 방식

```
CRI는 gRPC 로 통신한다.

구조:
kubelet
↓
unix socket
↓
container runtime

예:
/run/containerd/containerd.sock
```

### 실제 runtime 종류

현재 쿠버네티스에서 많이 사용하는 것들

| Runtime        | 특징            |
| -------------- | --------------- |
| **containerd** | 가장 많이 사용  |
| **CRI-O**      | Kubernetes 전용 |
| **Docker**     | 과거 방식       |

### 실제 Pod 생성 시 CRI 흐름

```
Pod 생성하면 runtime에서 이런 호출이 일어난다 :
RunPodSandbox

pause container 생성 :
CreateContainer

application container 생성 :
StartContainer

container 실행
```

```
[kubelet ↔ CRI 통신 예]
kubelet
 ↓
RunPodSandbox
 ↓
containerd
 ↓
pause container 생성

그 다음
CreateContainer
StartContainer
```

### CRI 위치 (전체 아키텍처)

```
Control Plane
  │
  │
kubelet
  │
  │ CRI
  ▼
container runtime
  │
  ▼
OCI runtime
  │
  ▼
Linux kernel
```

### Dockershim 제거

```
예전에는 kubelet 내부에 dockershim 이 있었다.

구조:
kubelet
↓
dockershim
↓
Docker

하지만 Kubernetes 1.24 이후 dockershim 제거

그래서 현재 구조는
kubelet
↓
CRI
↓
containerd
```

---

# 2장 CRI 내부 서비스

CRI는 크게 2개의 서비스로 구성된다.

### 1️⃣ Runtime Service

컨테이너 실행 관련 API

```
대표 함수:
RunPodSandbox
StopPodSandbox
CreateContainer
StartContainer
StopContainer
RemoveContainer
```

```
Pod 생성 흐름:

RunPodSandbox
↓
CreateContainer
↓
StartContainer
```

### 2️⃣ Image Service

이미지 관리

```
대표 함수:
PullImage
ListImages
RemoveImage
ImageStatus

예:
nginx image pull
```

---

# 3장 런타임 내부구조

실제 컨테이너가 어떻게 실행되는지 런타임 내부 고조를 살펴보자

### 흐름

```
kubelet
 ↓
CRI
 ↓
containerd
 ↓
containerd-shim
 ↓
runc
 ↓
container (Linux namespace + cgroup)
```

### 1️⃣ container runtime: containerd

먼저 가장 많이 쓰는 런타임이 **containerd**다.

| 역할                | 설명                  |
| ------------------- | --------------------- |
| Image 관리          | pull / push           |
| Container lifecycle | create / start / stop |
| Snapshot            | filesystem layer 관리 |
| CRI 연결            | kubelet과 통신        |

```
즉 containerd는 컨테이너 관리 엔진이다.

하지만 중요한 점:
❗ containerd도 실제 컨테이너를 직접 실행하지 않는다.
```

### 2️⃣ OCI Runtime

실제 컨테이너를 실행하는 것은 OCI runtime이다. 대표적인 것이 **runc**이다.

```
역할:

Linux namespace 생성
cgroup 설정
filesystem mount
container process 실행

즉 runc는 컨테이너를 실제로 만드는 프로그램이다.
```

### 3️⃣ 왜 containerd-shim이 필요한가

```
중간에 있는 것이 containerd-shim이다.

구조:
containerd
 ↓
containerd-shim
 ↓
runc
 ↓
container

이 shim이 중요한 이유가 있다.

[문제 (shim 없으면)]
만약 containerd가 직접 runc를 실행하면:

containerd 죽음
 ↓
모든 container 종료

이건 매우 위험하다.

[해결: shim]
containerd-shim이 중간에 있으면:

containerd
 ↓
shim
 ↓
container

containerd가 죽어도 shim이 container 유지
```

### 4️⃣ 실제 Pod 실행 흐름

```
Pod 생성 시 실제로 이런 일이 일어난다.

[Step 1] (RunPodSandbox)
kubelet → CRI
pause container 생성

[Step 2] (CreateContainer)
application container 생성

[Step 3] (containerd 내부)
containerd
↓
containerd-shim 생성

[Step 4] (shim이 runc 실행)
runc create
runc start

[Step 5] (runc가 Linux container 생성)
namespace
cgroup
mount
process
```

### 5️⃣ 실제 프로세스 구조

리눅스에서 보면 이런 트리다.

```
containerd
 └─ containerd-shim
      └─ nginx
```

### 6️⃣ 컨테이너 내부 기술

runc가 실행할 때 사용하는 Linux 기술

| 기술         | 역할            |
| ------------ | --------------- |
| namespace    | 프로세스 격리   |
| cgroup       | 리소스 제한     |
| mount        | filesystem 격리 |
| capabilities | 권한 제한       |

### 7️⃣ namespace 종류

컨테이너는 여러 namespace를 사용한다.

| namespace | 설명          |
| --------- | ------------- |
| pid       | 프로세스 격리 |
| net       | 네트워크      |
| mnt       | 파일시스템    |
| ipc       | 공유 메모리   |
| uts       | hostname      |
| user      | 사용자        |

### 8️⃣ cgroup (리소스 제한)

컨테이너 CPU / 메모리 제한은 cgroup으로 한다.

```
cpu.shares
memory.limit_in_bytes
```

```
쿠버네티스에서
---
resources:
  limits:
    cpu: 1
    memory: 512Mi
---

이 설정이 결국 cgroup 설정 으로 변환된다.
```

### 9️⃣ 전체 실행 흐름 (정리)

Pod 생성 → 실제 Linux container 생성까지 흐름

```
kubectl
↓
kube-apiserver
↓
scheduler
↓
kubelet
↓
CRI
↓
containerd
↓
containerd-shim
↓
runc
↓
namespace + cgroup 생성
↓
container process 실행
```

```
[전체 구조를 다시 그리면]
kubelet
  │
  │ CRI
  ▼
containerd
  │
  ▼
containerd-shim
  │
  ▼
runc
  │
  ▼
Linux kernel
  │
  ├─ namespace
  ├─ cgroup
  └─ process
```
