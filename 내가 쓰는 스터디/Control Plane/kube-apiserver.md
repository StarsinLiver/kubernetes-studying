## 목차

- [1장 kube-apiserver 란?](#1장-kube-apiserver-란)
  - [📦 어디서 실행되냐?](#-어디서-실행되냐)
  - [🔥 핵심 개념 (무조건 기억)](#-핵심-개념-무조건-기억)
  - [🧠 역할](#-역할)
  - [⚙️ 전체 구조](#️-전체-구조)
  - [📡 통신 방식](#-통신-방식)
  - [🔥 중요한 흐름 (Pod 생성 기준)](#-중요한-흐름-pod-생성-기준)
- [2장 Admission Controller (핵심🔥)](#2장-admission-controller-핵심)
  - [흐름](#흐름)
  - [💾 etcd와 관계](#-etcd와-관계)
  - [🔐 인증 \& 인가](#-인증--인가)
  - [🔄 Watch (중요🔥)](#-watch-중요)
- [3장 🧭 kube-apiserver 트래픽 보는 방법 (전체 맵)](#3장--kube-apiserver-트래픽-보는-방법-전체-맵)
  - [1️⃣ kubectl에서 요청 보기 (가장 쉬움)](#1️⃣-kubectl에서-요청-보기-가장-쉬움)
  - [2️⃣ kube-apiserver 로그 보기](#2️⃣-kube-apiserver-로그-보기)
  - [3️⃣ 네트워크 레벨 (tcpdump / Wireshark) ⭐](#3️⃣-네트워크-레벨-tcpdump--wireshark-)
  - [4️⃣ Audit Log (진짜 핵심🔥🔥🔥)](#4️⃣-audit-log-진짜-핵심)

---

# 1장 kube-apiserver 란?

🔹 한 줄 정의

쿠버네티스의 모든 요청을 처리하는 중앙 API 서버 (kubelet이 실행하는 Static Pod)

또는 상태를 저장하고, 모든 컴포넌트가 상태를 공유하는 중앙 허브

### 📦 어디서 실행되냐?

```
👉 kubeadm 기준:
/etc/kubernetes/manifests/kube-apiserver.yaml
```

### 🔥 핵심 개념 (무조건 기억)

```
❗ 쿠버네티스의 모든 컴포넌트는 kube-apiserver를 통해서만 통신한다
```

### 🧠 역할

```
✔️ 1. 요청의 입구
kubectl
kubelet
controller
scheduler
👉 전부 여기로 요청 보냄

✔️ 2. 인증 / 권한 체크
TLS 인증
RBAC 권한 확인

✔️ 3. 데이터 저장 (etcd 연동)
상태를 etcd에 저장

✔️ 4. 상태 제공
클러스터 상태 조회 가능
```

### ⚙️ 전체 구조

```
[kubectl]
[kubelet]
[controller]
[scheduler]
      ↓
kube-apiserver
      ↓
etcd
```

### 📡 통신 방식

```
✔️ REST API
HTTP 기반 및 JSON 데이터
```

### 🔥 중요한 흐름 (Pod 생성 기준)

```
1. kubectl apply
2. kube-apiserver 요청 수신
3. 인증 / 권한 체크
4. etcd에 저장
5. scheduler가 감지
6. kubelet이 감지
7. 실행


1️⃣ 직접 실행 안함
👉 항상 API Server 통해야 함

2️⃣ 이벤트 중심 시스템
👉 polling이 아니라 watch

3️⃣ 상태 저장은 etcd
👉 API Server는 게이트웨이
```

---

# 2장 Admission Controller (핵심🔥)

🔹 한 줄 정의

요청 가로채서 검증 / 수정 (👉 API Server 내부에서 동작)

```
[예시]
Pod 생성 시:
1. 리소스 제한 체크
2. default 값 추가
```

### 흐름

```
요청 → 인증 → 인가 → Admission → etcd 저장
```

### 💾 etcd와 관계

```
kubectl → apiserver → etcd
👉 API Server만 etcd 접근 가능
👉 직접 etcd 접근 ❌
```

### 🔐 인증 & 인가

```
1️⃣ 인증 (Authentication) (연관 : kubeconfig)
👉 “누구냐?”

인증서
토큰

2️⃣ 인가 (Authorization)
👉 “이거 해도 되냐?”

RBAC
Role / ClusterRole
```

### 🔄 Watch (중요🔥)

```
✔️ 개념
상태 변화를 실시간으로 감시

✔️ 예
kubelet: “내 노드에 Pod 생겼냐?”
scheduler: “아직 노드 안 정해진 Pod 있냐?”

👉 전부 API Server watch

[예시]
➜ kubectl get pods -w
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          46m
- watch 기반이므로 ctrl + c 될 때까지 상태 변화를 실시간으로 감시
```

---

# 3장 🧭 kube-apiserver 트래픽 보는 방법 (전체 맵)

| 레벨 | 방법                | 난이도      | 목적           |
| ---- | ------------------- | ----------- | -------------- |
| 1    | kubectl verbose     | 쉬움        | 요청 확인      |
| 2    | apiserver 로그      | 중간        | 서버 처리 확인 |
| 3    | tcpdump / Wireshark | 어려움      | 패킷 수준      |
| 4    | audit log           | 실무 핵심🔥 | 모든 요청 기록 |

### 1️⃣ kubectl에서 요청 보기 (가장 쉬움)

```
✔️ 방법
kubectl get pods -v=8

🔹 결과
GET https://<apiserver>/api/v1/pods

🔥 핵심
HTTP 요청 그대로 보여줌
헤더 / URL / 인증 포함

👉 이건 “클라이언트 → 서버” 확인
```

### 2️⃣ kube-apiserver 로그 보기

```
👉 kubeadm 환경 기준

✔️ 실행
kubectl logs -n kube-system kube-apiserver-<노드이름>

또는 노드에서:

docker logs <apiserver container>
# 또는
crictl logs <container id>

🔹 확인 내용
요청 처리 로그
에러
인증 실패

👉 이건 “서버 내부 처리” 확인
```

### 3️⃣ 네트워크 레벨 (tcpdump / Wireshark) ⭐

```
✔️ apiserver 포트
6443 (HTTPS)

✔️ tcpdump
tcpdump -i any port 6443

🔹 결과
TLS 패킷 보임
HTTP는 암호화됨

👉 그래서:
❗ 내용은 안보이고 “흐름”만 보임

✔️ Wireshark
TLS handshake 확인
연결 흐름 분석
```

### 4️⃣ Audit Log (진짜 핵심🔥🔥🔥)

```
👉 실무에서 “누가 뭘 했는지” 보는 방법

✔️ 개념
API Server로 들어온 모든 요청 기록

🔹 활성화 필요
kube-apiserver 옵션:
--audit-log-path=/var/log/kubernetes/audit.log

🔹 로그 예시
{
"verb": "create",
"user": {
"username": "admin"
},
"objectRef": {
"resource": "pods",
"name": "nginx"
}
}
```
