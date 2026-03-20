## 목차

- [1장 한 줄 정의](#1장-한-줄-정의)
- [2장 kubectl 전체 옵션 구조](#2장-kubectl-전체-옵션-구조)
  - [2-1장 실무에서 자주 쓰는 용어](#2-1장-실무에서-자주-쓰는-용어)

---

# 1장 한 줄 정의

쿠버네티스 클러스터의 API Server에 요청을 보내는 CLI 도구

```
🧠 핵심 개념 먼저 잡자

👉 kubectl은 절대 직접:
컨테이너 실행 ❌
Pod 생성 ❌
노드 접근 ❌

👉 오직 하나만 한다:
❗ API Server에 HTTP 요청을 보낸다

⚙️ 내부 동작 구조
kubectl → kube-apiserver → (etcd / scheduler / kubelet)

🔥 실제 흐름 (예시)
kubectl run nginx --image=nginx

이걸 내부적으로 보면:
1. kubectl이 요청 생성 (JSON)
2. API Server로 HTTP POST
3. API Server가 etcd에 저장
4. 이후 scheduler/kubelet이 처리

🧪 요청 확인(중요🔥)
kubectl get pods -v=8

👉 보면:
실제 HTTP 요청 로그 뜸

➜  .kube kubectl get pods -v=8
I0320 12:28:02.731756    8616 loader.go:395] Config loaded from file:  /root/.kube/config
I0320 12:28:02.742495    8616 round_trippers.go:463] GET https://192.168.0.13:6443/api/v1/namespaces/default/pods?limit=500
I0320 12:28:02.742593    8616 round_trippers.go:469] Request Headers:
I0320 12:28:02.742750    8616 round_trippers.go:473]     Accept: application/json;as=Table;v=v1;g=meta.k8s.io,application/json;as=Table;v=v1beta1;g=meta.k8s.io,application/json
I0320 12:28:02.742780    8616 round_trippers.go:473]     User-Agent: kubectl/v1.29.15 (linux/amd64) kubernetes/0d0f172
I0320 12:28:02.762534    8616 round_trippers.go:574] Response Status: 200 OK in 19 milliseconds
I0320 12:28:02.762591    8616 round_trippers.go:577] Response Headers:
I0320 12:28:02.762711    8616 round_trippers.go:580]     Cache-Control: no-cache, private
I0320 12:28:02.762796    8616 round_trippers.go:580]     Content-Type: application/json
I0320 12:28:02.762818    8616 round_trippers.go:580]     X-Kubernetes-Pf-Flowschema-Uid: 687f51bb-08f7-48ec-bf20-25259fce9fe7
I0320 12:28:02.762907    8616 round_trippers.go:580]     X-Kubernetes-Pf-Prioritylevel-Uid: 67634caf-2e25-4a13-a3af-72f25f15d2c8
I0320 12:28:02.762994    8616 round_trippers.go:580]     Content-Length: 2959
I0320 12:28:02.763026    8616 round_trippers.go:580]     Date: Fri, 20 Mar 2026 16:28:02 GMT
I0320 12:28:02.763101    8616 round_trippers.go:580]     Audit-Id: ca207a4b-9c19-4d62-9e10-de82a749f436
I0320 12:28:02.763888    8616 request.go:1212] Response Body: {"kind":"Table","apiVersion":"meta.k8s.io/v1","metadata":{"resourceVersion":"1343"},"columnDefinitions":[{"name":"Name","type":"string","format":"name","description":"Name must be unique within a namespace. [truncated 1935 chars]
```

---

# 2장 kubectl 전체 옵션 구조

```
kubectl [global flags] <command> [type] [name] [flags]
```

```
1️⃣ 전역 옵션 (Global Flags)
👉 모든 kubectl 명령에 붙일 수 있는 것들

🔹 연결 관련
--server              # API Server 주소
--kubeconfig          # kubeconfig 경로
--context             # 사용할 context
--cluster             # 클러스터 지정
--user                # 사용자 지정

🔹 인증 / 보안
--certificate-authority
--client-certificate
--client-key
--token
--insecure-skip-tls-verify

🔹 출력 / 디버깅
--v=8                 # 로그 레벨 (디버깅 핵심🔥)
--request-timeout

🔹 네임스페이스
-n, --namespace
--all-namespaces

2️⃣ 핵심 명령어 그룹 (중요🔥)
📦 리소스 조회 / 확인
kubectl get
kubectl describe
kubectl api-resources
kubectl api-versions

🏗️ 리소스 생성 / 수정
kubectl apply
kubectl create
kubectl edit
kubectl patch
kubectl replace

❌ 삭제
kubectl delete

📂 파일 / 리소스 관리
kubectl kustomize
kubectl diff

🧪 디버깅 / 실행
kubectl exec
kubectl logs
kubectl attach
kubectl port-forward

📡 클러스터 정보
kubectl cluster-info
kubectl top
kubectl get componentstatuses

🔐 인증 / 컨텍스트
kubectl config
kubectl auth

🚀 고급
kubectl rollout
kubectl scale
kubectl autoscale
kubectl drain
kubectl cordon
kubectl taint

3️⃣ “진짜 자주 쓰는” 핵심 옵션들

🔹 출력 형식
-o yaml
-o json
-o wide
-o custom-columns

🔹 필터링
-l key=value        # label selector
--field-selector

🔹 파일 입력
-f file.yaml

🔹 강제 옵션
--force
--grace-period=0

🔹 watch (실시간)
-w

🔹 컨테이너 지정
-c container-name
```

## 2-1장 실무에서 자주 쓰는 용어

```
1️⃣ -o yaml

✔️ 특징
리소스 전체 구조 그대로 출력
사람이 보기 쉽게 JSON → YAML 변환

🔹 예시
➜  kubectl get pod nginx -o yaml
spec:
  containers:
  - name: nginx
status:
  phase: Running

2️⃣ -o wide

✔️ 특징
기본 테이블 + 추가 정보

🔹 예시
➜ kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP            NODE   NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          2m34s   10.96.0.200   tiny   <none>           <none>

3️⃣ -o custom-columns

✔️ 개념
원하는 필드만 골라서 테이블로 출력

🔹 기본 문법
kubectl get pods -o custom-columns=<컬럼명>:<JSON 경로>

➜ kubectl get pods -o custom-columns=NAME:.metadata.name,IP:.status.podIP
NAME    IP
nginx   10.96.0.200

4️⃣ -o json

✔️ 특징
API Server가 반환한 원본 데이터를 JSON 그대로 출력

kubectl은 내부적으로 항상 이렇게 동작한다:
API Server → JSON 응답 → kubectl → (yaml / table / json 변환)

kubectl -o jsonpath란?

🔹 한 줄 정의
JSON 데이터에서 원하는 값만 뽑아내는 쿼리 문법

🔹 핵심 포인트
. → 루트 (현재 객체)
items → 리스트
[*] → 전체 반복

💥 jsonpath 핵심 문법 정리
{} → 표현식
[] → 배열 접근
range → 반복문
@ → 현재 요소
{"\n"} → 줄바꿈

🔹 기본 문법
✔️ 단일 값
-o jsonpath='{.metadata.name}'
✔️ 여러 개 (배열)
-o jsonpath='{.items[*].metadata.name}'
✔️ 줄바꿈 추가 (근데 이건 마지막에만 줄바꿈됨)
-o jsonpath='{.items[*].metadata.name}{"\n"}'
✔️ 제대로 줄바꿈 반복 출력 (range 사용)
-o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}'
✔️ 여러 값 출력
-o jsonpath='{range .items[*]}{.metadata.name} {.status.podIP}{"\n"}{end}'
✔️ 특정 인덱스 접근 (첫 번째 컨테이너 이미지)
kubectl get pod nginx -o jsonpath='{.spec.containers[0].image}'
✔️ 특정 값 선택 (Running 상태만 출력)
kubectl get pods -o jsonpath='{.items[?(@.status.phase=="Running")].metadata.name}'
```
