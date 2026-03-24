## 목차

- [1장 DaemonSet이 무엇인가](#1장-daemonset이-무엇인가)
  - [1️⃣ DaemonSet이 무엇인가](#1️⃣-daemonset이-무엇인가)
  - [2️⃣ 이름이 왜 DaemonSet인가](#2️⃣-이름이-왜-daemonset인가)
  - [3️⃣ 실제 Kubernetes에서 쓰는 곳](#3️⃣-실제-kubernetes에서-쓰는-곳)
    - [1️⃣ CNI plugin](#1️⃣-cni-plugin)
    - [2️⃣ 로그 수집](#2️⃣-로그-수집)
    - [3️⃣ 모니터링](#3️⃣-모니터링)
  - [4️⃣ Deployment와 차이](#4️⃣-deployment와-차이)
  - [5️⃣ DaemonSet YAML](#5️⃣-daemonset-yaml)
  - [6️⃣ DaemonSet 내부 동작](#6️⃣-daemonset-내부-동작)
  - [7️⃣ Scheduler를 안쓴다](#7️⃣-scheduler를-안쓴다)
  - [8️⃣ 실제 생성 흐름](#8️⃣-실제-생성-흐름)
  - [9️⃣ Node 추가 시](#9️⃣-node-추가-시)
  - [10. Node 삭제 시](#10-node-삭제-시)
  - [11. Linux 관점](#11-linux-관점)
  - [12. Host 접근](#12-host-접근)
  - [13. Rolling Update](#13-rolling-update)
  - [14. 실제 예시 (calico)](#14-실제-예시-calico)

---

# 1장 DaemonSet이 무엇인가

## 1️⃣ DaemonSet이 무엇인가

🔹 한 줄 정의

DaemonSet은 모든 Node에 Pod 하나씩 실행 시키는 Controller다.

Node 3개

```
node1
node2
node3
```

DaemonSet 생성하면

```
node1 → pod
node2 → pod
node3 → pod
```

즉 Node 하나당 1 Pod 이다.

## 2️⃣ 이름이 왜 DaemonSet인가

Linux의 daemon 프로세스에서 온 이름이다.

daemon 프로세스 예시

```
sshd
systemd
docker
```

이런 것들은 모든 서버에서 항상 실행된다.

Kubernetes에서도 모든 Node에서 실행해야 하는 Pod들이 있다.

## 3️⃣ 실제 Kubernetes에서 쓰는 곳

DaemonSet은 거의 전부 인프라용이다.

대표적으로

### 1️⃣ CNI plugin

```
calico-node
flannel
weave
```

지금 쓰는 Calico도 DaemonSet이다.

확인해보면

```
kubectl get daemonset -n kube-system

아마 이런게 있을 것이다. calico-node
```

### 2️⃣ 로그 수집

```
fluentd
filebeat
vector
```

```
각 node의 /var/log을 읽어야 하기 때문에 node마다 pod 필요하다.
```

### 3️⃣ 모니터링

```
node-exporter
datadog-agent
newrelic-agent
```

Node CPU / Memory / Disk를 수집한다.

## 4️⃣ Deployment와 차이

```
Deployment replicas = 3이면 Scheduler가

node1
node2
node3

아무 곳에 배치한다.
```

```
DaemonSet은 replicas 없다. 대신 node 수 = pod 수 이다.
```

## 5️⃣ DaemonSet YAML

```yaml
apiVersion: apps/v1
kind: DaemonSet

metadata:
  name: node-exporter

spec:
  selector:
    matchLabels:
      app: node-exporter

  template:
    metadata:
      labels:
        app: node-exporter

    spec:
      containers:
        - name: node-exporter
          image: prom/node-exporter
```

```
특징으로는 replicas 가 없다
```

## 6️⃣ DaemonSet 내부 동작

```
DaemonSet Controller는 kube-controller-manager 에 있다.
```

구조

````
DaemonSet Controller
      │
      ├── watch Node
      ├── watch Pod
      │
      └── reconcile
      ```
````

핵심 로직

```
모든 node 확인
→ 해당 node에 pod 있는지 확인
→ 없으면 생성
```

간단한 알고리즘

```
nodes = getNodes()

for node in nodes:

  if pod not exist on node:
        create pod on node
```

## 7️⃣ Scheduler를 안쓴다

Deployment

```
Pod 생성
→ Scheduler
→ Node 선택
```

DaemonSet

```
Controller가 Node 지정한다.
즉 Pod 생성 시 nodeName 지정된다.
```

예

```
spec:
  nodeName: node1
```

```
그래서 scheduler bypass한다.
```

## 8️⃣ 실제 생성 흐름

```
DaemonSet 생성
      ↓
Controller node 목록 조회
      ↓
node1 pod 생성
node2 pod 생성
node3 pod 생성
```

## 9️⃣ Node 추가 시

```
node4라는 Node가 추가되면 Controller가 감지한다. (watch node event)
그리고 node4 → pod 생성한다.
```

## 10. Node 삭제 시

```
node2라는 Node 삭제시 자동으로 node2의 pod 삭제된다.
```

## 11. Linux 관점

각 node에서 container runtime이 pod를 실행한다.

예

```
[node1]
/var/lib/containerd

안에 daemonset container 생긴다.
```

## 12. Host 접근

DaemonSet은 보통

```
hostNetwork
hostPID
hostPath
```

을 많이 쓴다.

예

```
hostNetwork: true
또는
hostPath:
  path: /var/log
```

> 이유는 node 리소스 접근을 위해서이다.

## 13. Rolling Update

Deployment

```
parallel update
```

DaemonSet

```
node별 순차 update
```

예

```
node1 update
node2 update
node3 update
```

## 14. 실제 예시 (calico)

확인

```
kubectl get pods -n kube-system -o wide
```

아마 이런게 보일 것이다.

```
calico-node-abcde   node1
calico-node-fghij   node2
calico-node-klmno   node3
```

각 node마다 있다.
