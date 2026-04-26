## 목차

- [1장 Anffinity / Anti-affinity](#1장-anffinity--anti-affinity)
  - [✅ 1. Anffinity / Anti-affinity 이란?](#-1-anffinity--anti-affinity-이란)
  - [✅ 2. 어디서 실행되냐 (아키텍처🔥)](#-2-어디서-실행되냐-아키텍처)
  - [✅ 3. Linux 어디에 저장되냐](#-3-linux-어디에-저장되냐)
  - [✅ 4. 종류 (핵심🔥)](#-4-종류-핵심)
    - [🔥 1) Node Affinity (노드 기준)](#-1-node-affinity-노드-기준)
    - [🔥 2) Pod Affinity (Pod 기준)](#-2-pod-affinity-pod-기준)
    - [🔥 3) Pod Anti-affinity](#-3-pod-anti-affinity)
  - [✅ 5. 강제 vs 선호 (매우 중요🔥)](#-5-강제-vs-선호-매우-중요)
  - [✅ 6. 전체 구조](#-6-전체-구조)
  - [✅ 7. Node Affinity (상세🔥)](#-7-node-affinity-상세)
- [2장 멀티 AZ / 멀티 리전 배치 전략](#2장-멀티-az--멀티-리전-배치-전략)
  - [✅ 1. 멀티 AZ / 멀티 리전 배치 전략이란?](#-1-멀티-az--멀티-리전-배치-전략이란)
  - [✅ 2. AZ / Region 개념](#-2-az--region-개념)
    - [✔ AZ (Availability Zone)](#-az-availability-zone)
    - [✔ Region](#-region)
  - [✅ 3. 어디서 실행되냐 (아키텍처🔥)](#-3-어디서-실행되냐-아키텍처)
  - [✅ 4. Linux / 저장 위치](#-4-linux--저장-위치)
  - [✅ 5. 핵심 개념 (중요🔥)](#-5-핵심-개념-중요)
    - [🔥 1) “같이 두지 말라” (Anti-affinity)](#-1-같이-두지-말라-anti-affinity)
    - [🔥 2) “골고루 퍼뜨려라” (Topology Spread)](#-2-골고루-퍼뜨려라-topology-spread)
    - [🔥 3) “트래픽도 같이 분산해야 함”](#-3-트래픽도-같이-분산해야-함)
  - [✅ 6. 전체 구조](#-6-전체-구조-1)
  - [✅ 7. 핵심 전략 1: Pod Anti-affinity](#-7-핵심-전략-1-pod-anti-affinity)
  - [✅ 8. 핵심 전략 2: Topology Spread (강력🔥)](#-8-핵심-전략-2-topology-spread-강력)
  - [✅ 9. 핵심 전략 3: Node Affinity](#-9-핵심-전략-3-node-affinity)
  - [✅ 10. 핵심 전략 4: Service / Load Balancing](#-10-핵심-전략-4-service--load-balancing)
  - [✅ 11. 멀티 AZ 전체 흐름 (중요🔥)](#-11-멀티-az-전체-흐름-중요)
  - [✅ 12. 멀티 Region 전략 (더 중요🔥)](#-12-멀티-region-전략-더-중요)
  - [✅ 13. 실전 설계 패턴🔥](#-13-실전-설계-패턴)
  - [✅ 14. 실무에서 터지는 문제🔥](#-14-실무에서-터지는-문제)

---

# 1장 Anffinity / Anti-affinity

## ✅ 1. Anffinity / Anti-affinity 이란?

한줄 정리

```
Affinity 는 “같이 두고 싶은 조건”
Anti-affinity 는 “같이 두기 싫은 조건”을 정의하는 스케줄링 규칙
```

어떤 역할을 하는가

```
특정 노드에만 배치 (region, disk, GPU)
특정 Pod와 같이 배치 (캐시-앱)
특정 Pod와 분리 (HA, 장애 격리)
같은 AZ/노드 분산 배치
```

👉 핵심

```
“배치 위치를 ‘선호/강제’로 제어”
```

## ✅ 2. 어디서 실행되냐 (아키텍처🔥)

```
Scheduler에서만 동작
Pod 생성 시 filter / score 단계에서 판단
```

```
Pod 생성
   ↓
Scheduler
   ├─ Filter (조건 만족 노드만 남김)
   └─ Score  (우선순위 계산)
   ↓
최종 Node 선택
```

```
Kubelet은 Taint(NoExecute)처럼 “퇴출”은 관여하지만, Affinity 자체 판단은 스케줄러 영역이다.
```

## ✅ 3. Linux 어디에 저장되냐

👉 etcd에 저장됨 (Pod spec 안)

```
 /registry/pods/<namespace>/<pod>
```

✔ 확인 방법

```
kubectl get pod <name> -o yaml
kubectl describe pod <name>
```

## ✅ 4. 종류 (핵심🔥)

### 🔥 1) Node Affinity (노드 기준)

```
👉 “이런 노드에 배치해라”
```

### 🔥 2) Pod Affinity (Pod 기준)

```
👉 “이 Pod랑 같은 곳에 둬라”
```

### 🔥 3) Pod Anti-affinity

```
👉 “이 Pod랑 떨어뜨려라”
```

## ✅ 5. 강제 vs 선호 (매우 중요🔥)

| 타입                                            | 의미                    |
| ----------------------------------------------- | ----------------------- |
| requiredDuringSchedulingIgnoredDuringExecution  | 반드시 만족 (필터 단계) |
| preferredDuringSchedulingIgnoredDuringExecution | 가능하면 (점수 단계)    |

```
👉 핵심:

required = 못 맞추면 스케줄 실패
preferred = 안 맞아도 배치됨
```

## ✅ 6. 전체 구조

```
Pod
 └── spec
      └── affinity
           ├── nodeAffinity
           ├── podAffinity
           └── podAntiAffinity
```

## ✅ 7. Node Affinity (상세🔥)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: node-affinity-pod

spec:
  affinity:
    # nodeAffinity : “이런 노드에 배치해라”
    nodeAffinity:
      # 반드시 만족 (필터 단계)
      requiredDuringSchedulingIgnoredDuringExecution:
        # nodeSelectorTerms : OR 조건
        # matchExpressions : AND 조건
        nodeSelectorTerms:
          - matchExpressions:
              - key: disktype
                # operator
                # 값            의미
                # In	        포함
                # NotIn	        제외
                # Exists	    존재
                # DoesNotExist	없음
                # Gt	        크다
                # Lt	        작다
                operator: In
                values:
                  - ssd

      # 가능하면 (점수 단계)
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 1
          preference:
            matchExpressions:
              - key: zone
                operator: In
                values:
                  - seoul

    # podAntiAffinity : “이 Pod랑 같은 곳에 둬라”
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        # labelSelector : 대상 Pod 선택
        - labelSelector:
            matchLabels:
              app: redis
          # topologyKey (중요🔥)
          # : “같은 topologyKey 영역에 배치”
          # 값                             의미
          # kubernetes.io/hostname         같은 노드
          # topology.kubernetes.io/zone    같은 AZ
          # topology.kubernetes.io/region  같은 리전
          topologyKey: kubernetes.io/hostname

    # podAntiAffinity : “이 Pod랑 떨어뜨려라”
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        # labelSelector : 대상 Pod 선택
        - labelSelector:
            matchLabels:
              app: web
          # topologyKey (중요🔥)
          # : “같은 topologyKey 영역에 배치”
          # 값                             의미
          # kubernetes.io/hostname         같은 노드
          # topology.kubernetes.io/zone    같은 AZ
          # topology.kubernetes.io/region  같은 리전
          topologyKey: kubernetes.io/hostname
```

---

# 2장 멀티 AZ / 멀티 리전 배치 전략

## ✅ 1. 멀티 AZ / 멀티 리전 배치 전략이란?

한줄 정리

```
멀티 AZ/리전 배치는 장애가 나도 서비스가 끊기지 않도록
워크로드를 물리적으로 분산하는 전략
```

## ✅ 2. AZ / Region 개념

### ✔ AZ (Availability Zone)

```
같은 리전 내 서로 다른 데이터센터
네트워크 빠름
```

### ✔ Region

```
완전히 다른 지역 (서울 / 도쿄 등)
네트워크 느림
```

구조

```
Region (Seoul)
 ├── AZ-a
 ├── AZ-b
 └── AZ-c
```

## ✅ 3. 어디서 실행되냐 (아키텍처🔥)

```
1. Scheduler → Pod 분산
2. Controller (Deployment) → replica 유지
3. Service / LB → 트래픽 분산
4. CNI / 네트워크 → 통신
```

## ✅ 4. Linux / 저장 위치

👉 etcd에 저장됨

```
Pod spec (affinity)
Node label (zone, region)
```

✔ 확인

```
kubectl get nodes --show-labels

👉 핵심 label:
topology.kubernetes.io/zone
topology.kubernetes.io/region
```

## ✅ 5. 핵심 개념 (중요🔥)

### 🔥 1) “같이 두지 말라” (Anti-affinity)

```
👉 같은 AZ에 몰리면 의미 없음
```

### 🔥 2) “골고루 퍼뜨려라” (Topology Spread)

```
👉 최신 방식
```

### 🔥 3) “트래픽도 같이 분산해야 함”

```
👉 Service / LB 필요
```

## ✅ 6. 전체 구조

```
Region
 ├── AZ-a (Node1, Node2)
 │     └── Pod
 ├── AZ-b (Node3, Node4)
 │     └── Pod
 └── AZ-c (Node5, Node6)
       └── Pod
```

## ✅ 7. 핵심 전략 1: Pod Anti-affinity

✔ YAML

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: web
        topologyKey: topology.kubernetes.io/zone
```

✔ 의미

```
👉 같은 AZ에 같은 app 못 배치
```

## ✅ 8. 핵심 전략 2: Topology Spread (강력🔥)

```
👉 Anti-affinity보다 더 정교함
```

✔ YAML

```yaml
topologySpreadConstraints:
  # maxSkew : AZ 간 최대 Pod 차이
  - maxSkew: 1
    # topologyKey : 기준 (zone / hostname)
    topologyKey: topology.kubernetes.io/zone
    # whenUnsatisfiable
    # 값              의미
    # DoNotSchedule   못 맞추면 배치 안함
    # ScheduleAnyway  그냥 배치
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        app: web
```

## ✅ 9. 핵심 전략 3: Node Affinity

✔ 특정 AZ 지정

```yaml
nodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
      - matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In
            values:
              - az-a
              - az-b
```

```
👉 특정 AZ만 사용 가능
```

## ✅ 10. 핵심 전략 4: Service / Load Balancing

✔ Cluster 내부

```
kube-proxy
iptables
```

✔ 외부

```
LoadBalancer
Ingress
```

```
👉 핵심:
Pod 분산만 하면 끝이 아니라 트래픽도 분산해야 함
```

## ✅ 11. 멀티 AZ 전체 흐름 (중요🔥)

```
사용자 요청
   ↓
LoadBalancer
   ↓
각 AZ로 분산
   ↓
Service
   ↓
Pod (각 AZ에 분산)
```

## ✅ 12. 멀티 Region 전략 (더 중요🔥)

✔ 특징

```
완전 독립 클러스터
데이터 복제 필요
DNS 기반 라우팅
```

구조

```
Region A (Seoul)
 └── Cluster A

Region B (Tokyo)
 └── Cluster B
```

✔ 트래픽 분산

```
Geo DNS
Anycast
Global LB
```

✔ 데이터 전략

| 방식           | 설명           |
| -------------- | -------------- |
| Active-Active  | 양쪽 다 서비스 |
| Active-Passive | 하나는 대기    |

## ✅ 13. 실전 설계 패턴🔥

✔ HA 웹 서비스

```
replicas ≥ AZ 수
Anti-affinity or spread
```

✔ DB

```
AZ 분산 필수
replication 필요
```

✔ 캐시

```
locality 중요 → affinity 사용
```

## ✅ 14. 실무에서 터지는 문제🔥

❗ 1) Pod 다 한 AZ에 몰림

```
👉 affinity 없음
```

❗ 2) Anti-affinity 때문에 Pending

```
👉 노드 부족
```

❗ 3) topologyKey 오타

```
👉 분산 안됨
```

❗ 4) Service가 특정 AZ만 때림

```
👉 LB 설정 문제
```

❗ 5) 멀티 리전에서 데이터 깨짐

```
👉 replication 문제
```
