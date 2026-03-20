## 목차

- [1장 kube-scheduler 란?](#1장-kube-scheduler-란)
  - [🧠 역할](#-역할)
  - [⚙️ 전체 흐름](#️-전체-흐름)
  - [👉 scheduler는 단순히 이거 한 줄만 한다:](#-scheduler는-단순히-이거-한-줄만-한다)
  - [🧠 내부 동작](#-내부-동작)
    - [1️⃣ Filtering (필터링)](#1️⃣-filtering-필터링)
    - [2️⃣ Scoring (점수 매기기)](#2️⃣-scoring-점수-매기기)
  - [중요한 개념들](#중요한-개념들)
  - [scheduler 특징](#scheduler-특징)
  - [중요한 관점](#중요한-관점)

---

# 1장 kube-scheduler 란?

🔹 한 줄 정의

실행되지 않은 Pod를 적절한 노드에 배치하는 결정을 내리는 컴포넌트다
<br>(scheduler는 “실행하지 않는다” → 오직 선택만 한다)

### 🧠 역할

```
✔️ 하는 일
Node 선택
Pod에 nodeName 할당

❌ 안 하는 일
컨테이너 실행 ❌ (kubelet 역할)
네트워크 ❌
이미지 pull ❌
```

### ⚙️ 전체 흐름

```
Pod 생성 (nodeName 없음)
↓
kube-scheduler 감지 (watch)
↓
노드 선택
↓
Pod.spec.nodeName 설정
↓
kubelet이 실행
```

### 👉 scheduler는 단순히 이거 한 줄만 한다:

```
spec:
  nodeName: node1
```

### 🧠 내부 동작

#### 1️⃣ Filtering (필터링)

```
👉 “이 Pod를 실행할 수 있는 노드만 남긴다”

🔹 조건
CPU / Memory 충분한지
nodeSelector
taint / toleration
affinity

🔹 결과
node1 ❌ (리소스 부족)
node2 ⭕
node3 ⭕
```

#### 2️⃣ Scoring (점수 매기기)

```
👉 남은 노드 중 “최적의 노드 선택”

🔹 기준
리소스 여유
Pod 분산
네트워크 거리

🔹 결과
node2: 80점
node3: 90점  ← 선택

🔄 전체 알고리즘
Pod 생성
   ↓
Filtering
   ↓
Scoring
   ↓
최고 점수 노드 선택

🧪 실제 확인
➜  kubectl get pod nginx -o yaml
---
...
spec:
  nodeName: node1 # 노드가 선택되어 있음
  ---
```

### 중요한 개념들

```
1️⃣ nodeSelector
nodeSelector:
  disktype: ssd
👉 특정 노드 강제

2️⃣ affinity (고급)
👉 더 정교한 조건

3️⃣ taint / toleration
👉 특정 노드 차단 / 허용
```

### scheduler 특징

```
✔️ 이벤트 기반
watch 사용

✔️ 비동기 처리
여러 Pod 동시에 처리

✔️ 확장 가능
커스텀 scheduler 가능
```

### 중요한 관점

```
1️⃣ scheduler는 “결정만” 한다
👉 실행은 kubelet

2️⃣ Pod 상태 변화 트리거는 etcd
👉 scheduler는 watch 기반

3️⃣ nodeName이 핵심 결과
👉 이 값 하나로 흐름이 바뀜
```
