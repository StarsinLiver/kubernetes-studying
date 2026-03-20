## 시작

🧭 0. 전체 구조 한눈에

쿠버네티스는 크게 3계층이다:

```
[사용자 도구]
↓
[Control Plane]
↓
[Node (Worker)]
```

### 🧱 1. 사용자 도구 (Client Layer)

🔹 kubectl

역할: API 호출 클라이언트

내부: REST API 요청

예:

kubectl get pod

👉 실제로는:
→ API Server로 HTTP 요청 날림

🔹 kubeadm

역할: 클러스터 설치 도구

하는 일:

Control Plane 구성

인증서 생성

etcd 구성

👉 “쿠버네티스 설치기”

🔹 kubeconfig

위치: ~/.kube/config

역할:

어떤 클러스터 접속할지

인증 정보

### 🧠 2. Control Plane (두뇌)

🔹 kube-apiserver

모든 요청의 입구

REST API 제공

인증 / 권한 체크

👉 모든 컴포넌트는 이것만 본다

🔹 etcd

분산 Key-Value 저장소

클러스터 상태 저장

👉 “쿠버네티스의 DB”

🔹 kube-scheduler

역할: Pod를 어느 Node에 배치할지 결정

👉 기준:

리소스 (CPU, Memory)

affinity / taint

🔹 kube-controller-manager

여러 컨트롤러 모음

대표 컨트롤러

Deployment Controller

ReplicaSet Controller

Node Controller

Job Controller

👉 역할:
“현재 상태 → 원하는 상태로 맞추기”

🔹 cloud-controller-manager (옵션)

클라우드 연동 (AWS, GCP)

LoadBalancer 생성 등

### ⚙️ 3. Node (Worker)

🔹 kubelet ⭐ (중요)

노드 에이전트

API Server와 통신

👉 하는 일:

Pod spec 받아서 실행

상태 보고

🔹 Container Runtime (CRI)

실제 컨테이너 실행

예:

containerd

CRI-O

👉 kubelet이 얘한테 명령함

🔹 kube-proxy

Service 트래픽 처리

👉 방식:

iptables / IPVS

### 📦 4. 핵심 오브젝트 (사용자가 다루는 것)

🔹 Pod

최소 배포 단위

컨테이너 묶음

하나의 IP

🔹 ReplicaSet

Pod 개수 유지

🔹 Deployment

Pod 업데이트 관리

롤링 업데이트

🔹 StatefulSet

상태 있는 앱

고정된 이름 / 스토리지

🔹 DaemonSet

모든 노드에 하나씩 실행

🔹 Job / CronJob

일회성 / 스케줄 작업

🌐 5. 네트워크 관련

### 🔹 CNI (Container Network Interface)

Pod 네트워크 담당

예:

Calico

Flannel

🔹 Service

Pod 접근 abstraction

타입

ClusterIP

NodePort

LoadBalancer

🔹 kube-proxy

Service → Pod 연결

🔹 CoreDNS

클러스터 DNS

👉 예:

my-service.default.svc.cluster.local
🔹 Ingress

외부 HTTP 라우팅

💾 6. 스토리지
🔹 Volume

Pod 내부 저장소

🔹 PersistentVolume (PV)

실제 스토리지

🔹 PersistentVolumeClaim (PVC)

스토리지 요청

🔹 StorageClass

동적 프로비저닝

🔐 7. 설정 / 보안
🔹 ConfigMap

설정값 저장

🔹 Secret

민감 정보 (비밀번호 등)

🔹 RBAC

권한 관리

🔹 ServiceAccount

Pod용 계정

📊 8. 기타 중요한 것들
🔹 Namespace

리소스 그룹

🔹 Label / Selector

리소스 매칭

🔹 Annotation

메타데이터

🔹 Taint / Toleration

특정 노드 배치 제어

🔹 Affinity / Anti-affinity

Pod 배치 전략

🔄 9. 전체 흐름 (진짜 핵심🔥)

Pod 생성 시:

kubectl apply
↓
kube-apiserver
↓
etcd 저장
↓
scheduler → 노드 선택
↓
kubelet → container runtime 호출
↓
컨테이너 생성
↓
CNI → 네트워크 붙임
