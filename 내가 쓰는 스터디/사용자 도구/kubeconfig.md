## 목차

- [1장 kubeconfig 란?](#1장-kubeconfig-란)
- [2장 kubectl config 명령어](#2장-kubectl-config-명령어)

---

# 1장 kubeconfig 란?

🔹 한 줄 정의

kubectl이 어느 클러스터에 어떻게 접속할지 알려주는 설정 파일

```
📂 위치
~/.kube/config


⚙️ 전체 구조
---
apiVersion: v1
kind: Config

clusters:
- name: my-cluster
  cluster:
    server: https://1.2.3.4:6443
    certificate-authority-data: ...

users:
- name: admin
  user:
    client-certificate-data: ...
    client-key-data: ...

contexts:
- name: my-context
  context:
    cluster: my-cluster
    user: admin
    namespace: default

current-context: my-context
---

🧠 3가지 핵심 구성요소

1️⃣ clusters
👉 “어디로 갈지”
cluster:
  server: https://API_SERVER:6443

1. API Server 주소
2. CA 인증서

2️⃣ users
👉 “누구로 접속할지”

user:
  client-certificate-data
  client-key-data

1. 인증서 기반 인증 또는 token

3️⃣ contexts ⭐ (핵심)
👉 “cluster + user 묶음”

context:
  cluster: my-cluster
  user: admin

🔥 current-context
👉 현재 사용 중인 설정

current-context: my-context

💥 핵심 구조 한방 정리
context = cluster + user + namespace

🧪 실제 동작 흐름
kubectl 실행
   ↓
kubeconfig 읽음
   ↓
current-context 확인
   ↓
cluster + user 정보 가져옴
   ↓
API Server로 HTTPS 요청

📦 여러 클러스터 관리
👉 kubeconfig 하나에 여러 클러스터 가능

cluster1 / user1 / context1
cluster2 / user2 / context2

👉 전환:
kubectl config use-context cluster2

📦 kubeadm이 자동 생성해준다
/etc/kubernetes/admin.conf
```

---

# 2장 kubectl config 명령어

```
🔹 현재 컨텍스트 보기
kubectl config current-context

🔹 컨텍스트 목록
kubectl config get-contexts

🔹 컨텍스트 변경
kubectl config use-context my-context

🔹 namespace 변경
kubectl config set-context --current --namespace=dev
```
