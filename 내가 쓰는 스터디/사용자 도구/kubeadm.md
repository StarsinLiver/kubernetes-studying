## 목차

- [1장 한 줄 정의](#1장-한-줄-정의)
- [2장 kubeadm이 만드는 파일들](#2장-kubeadm이-만드는-파일들)

---

# 1장 한 줄 정의

쿠버네티스 클러스터(Control Plane + Node)를 설치하고 초기화하는 도구

```
🔥 핵심 개념 먼저
❗ kubeadm은 “운영 도구”가 아니라 설치 도구다

👉 즉:
kubectl → 계속 사용
kubeadm → 처음 설치할 때만 사용

🧠 kubeadm이 하는 일 (본질)
👉 한 문장으로:
❗ “Control Plane을 구성하고 노드를 클러스터에 붙인다”

⚙️ 내부에서 실제로 하는 일

kubeadm init 실행하면 내부에서 이런 일이 벌어진다 👇
1. 인증서 생성 (TLS)
2. kube-apiserver 생성
3. etcd 생성
4. controller-manager 생성
5. scheduler 생성
6. kubelet 설정
7. kubeconfig 생성

1️⃣ 클러스터 생성
kubeadm init

2️⃣ 워커 노드 추가
kubeadm join <ip>:<port> --token xxx --discovery-token-ca-cert-hash xxx

3️⃣ 초기화 (삭제)
kubeadm reset

🔐 인증 구조 (중요🔥)

kubeadm이 자동으로 생성하는 것:
CA (Certificate Authority)
API Server 인증서
kubelet 인증서

👉 결과:
모든 통신 TLS 기반
```

---

# 2장 kubeadm이 만드는 파일들

```
📂 kubeadm이 만드는 파일들
🔹 kubeconfig
~/.kube/config

👉 kubectl이 쓰는 설정
🔹 주요 경로
/etc/kubernetes/
  ├─ admin.conf
  ├─ kubelet.conf
  ├─ manifests/

🔹 manifests 폴더 (핵심🔥)
/etc/kubernetes/manifests/
  ├─ kube-apiserver.yaml
  ├─ etcd.yaml
  ├─ kube-scheduler.yaml
  ├─ kube-controller-manager.yaml

💥 진짜 중요한 포인트

1️⃣ Control Plane도 Pod다
👉 kubeadm은 결국 이걸 만들어주는 것

2️⃣ kubelet이 Control Plane도 띄운다 (이걸 Static Pod라고 한다)
kubelet → manifests 읽음 → control plane pod 실행

🚀 kubeadm이 안 하는 것
❌ Pod 관리
❌ Service 관리
❌ 애플리케이션 배포

👉 이런 건 전부 kubectl 역할
```
