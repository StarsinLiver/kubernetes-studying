## 목차

- [1장 iptables 란](#1장-iptables-란)
  - [1️⃣ iptables란 무엇인가](#1️⃣-iptables란-무엇인가)
  - [2️⃣ iptables 전체 구조](#2️⃣-iptables-전체-구조)
    - [1️⃣ Table](#1️⃣-table)
      - [filter](#filter)
      - [nat](#nat)
    - [2️⃣ Chain](#2️⃣-chain)
    - [3️⃣ Rule](#3️⃣-rule)
  - [3️⃣ iptables 읽는 방법](#3️⃣-iptables-읽는-방법)
  - [4️⃣ iptables rule 읽는 방법](#4️⃣-iptables-rule-읽는-방법)
  - [5️⃣ Kubernetes iptables 읽는 방법](#5️⃣-kubernetes-iptables-읽는-방법)
  - [6️⃣ iptables rule 쓰는 방법](#6️⃣-iptables-rule-쓰는-방법)
  - [7️⃣ DNAT rule 만들기](#7️⃣-dnat-rule-만들기)
  - [8️⃣ Rule 삭제](#8️⃣-rule-삭제)
  - [9️⃣ 실제 Linux에서 어디 저장되는가](#9️⃣-실제-linux에서-어디-저장되는가)
  - [🔟 Kubernetes iptables 분석 팁](#-kubernetes-iptables-분석-팁)
  - [⭐ 진짜 중요한 iptables 읽는 순서](#-진짜-중요한-iptables-읽는-순서)

# 1장 iptables 란

## 1️⃣ iptables란 무엇인가

🔹 한 줄 정리

Linux 커널 Netfilter를 제어하는 방화벽/패킷 처리 규칙 시스템

구조

```
Network packet
      ↓
Linux kernel
      ↓
Netfilter
      ↓
iptables rules
```

iptables는 패킷을 다음과 같이 한다.

```
허용
차단
변환(NAT)
리다이렉트
```

## 2️⃣ iptables 전체 구조

iptables는 3단계 구조다.

```
Table
  ↓
Chain
  ↓
Rule
```

### 1️⃣ Table

iptables에는 여러 table이 있다.

대표적인 것

```
filter
nat
mangle
raw
```

#### filter

일반 방화벽

```
ACCEPT
DROP
```

#### nat

주소 변환

```
DNAT
SNAT
MASQUERADE
```

Kubernetes Service는 nat table을 사용한다.

### 2️⃣ Chain

Chain은 패킷 흐름 단계이다.

대표적인 chain

```
PREROUTING
INPUT
FORWARD
OUTPUT
POSTROUTING
```

패킷 흐름

```
Network
   ↓
PREROUTING
   ↓
Routing decision
   ↓
INPUT → local process
FORWARD → 다른 host
OUTPUT → local process
   ↓
POSTROUTING
```

### 3️⃣ Rule

Chain 안에는 rule이 있다.

예

```
if packet matches condition
    do action
```

## 3️⃣ iptables 읽는 방법

가장 많이 쓰는 명령어

```
➜  iptables -L
```

하지만 Kubernetes에서는 nat table을 봐야 한다.

```
➜  iptables -t nat -L
```

추천 명령어

```
➜  iptables -t nat -L -n -v
```

옵션

```
-n : DNS 변환 안함
-v : packet count 표시
```

더 정확한 방법

```
➜  iptables-save

왜냐면 iptables -L은 사람이 보기 좋게 바꾼 것이다.
```

실제 rule은 iptables-save가 정확하다.

## 4️⃣ iptables rule 읽는 방법

예

```
-A KUBE-SERVICES \
-d 10.96.0.10/32 \
-p tcp \
--dport 80 \
-j KUBE-SVC-ABCDE
```

이걸 해석해보자.

-A

```
Append

chain에 rule 추가
KUBE-SERVICES chain
```

-d

```
destination IP

10.96.0.10
```

-p

```
protocol

tcp
```

--dport

```
destination port

80
```

-j

```
jump

다음 chain 이동
KUBE-SVC-ABCDE
```

전체 해석

```
목적지 IP가 10.96.0.10
그리고 TCP 80 포트면

KUBE-SVC-ABCDE chain으로 이동
```

## 5️⃣ Kubernetes iptables 읽는 방법

Kubernetes chain

```
KUBE-SERVICES
KUBE-NODEPORTS
KUBE-SVC-xxxxx
KUBE-SEP-xxxxx
```

구조

```
PREROUTING
   ↓
KUBE-SERVICES
   ↓
KUBE-SVC
   ↓
KUBE-SEP
   ↓
DNAT
```

예

```
-A KUBE-SEP-AAAA \
-j DNAT --to-destination 10.244.1.5:8080
```

해석

```
Pod IP로 변환
```

## 6️⃣ iptables rule 쓰는 방법

기본 문법

```
iptables -t TABLE -A CHAIN [조건] -j ACTION
```

예

```
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

의미

```
SSH 허용
```

## 7️⃣ DNAT rule 만들기

예

```
iptables -t nat -A PREROUTING \
-d 1.1.1.1 \
-p tcp --dport 80 \
-j DNAT --to-destination 10.0.0.5:8080
```

의미

```
1.1.1.1:80
→
10.0.0.5:8080
```

## 8️⃣ Rule 삭제

rule 확인

```
iptables -L --line-numbers
```

예

```
1 ACCEPT tcp ...
2 DROP all ...
```

삭제

```
iptables -D INPUT 1
```

## 9️⃣ 실제 Linux에서 어디 저장되는가

중요한 사실

```
iptables rule은 커널 메모리에 있다
파일이 아니다.
```

즉

```
재부팅하면 사라진다
```

저장하려면

Ubuntu

```
iptables-save > /etc/iptables/rules.v4
```

CentOS

```
/etc/sysconfig/iptables
```

확인

```
iptables-save
```

## 🔟 Kubernetes iptables 분석 팁

Service 찾기

```
kubectl get svc
```

예

```
10.96.15.120
```

iptables 찾기

```
iptables-save | grep 10.96
```

chain 찾기

```
iptables-save | grep KUBE-SVC
```

endpoint 찾기

```
iptables-save | grep SEP
```

## ⭐ 진짜 중요한 iptables 읽는 순서

iptables는 이 순서로 읽어야 한다

1️⃣ PREROUTING
<br>2️⃣ KUBE-SERVICES
<br>3️⃣ KUBE-SVC
<br>4️⃣ KUBE-SEP
<br>5️⃣ DNAT

즉

```
packet
↓
PREROUTING
↓
KUBE-SERVICES
↓
KUBE-SVC
↓
KUBE-SEP
↓
DNAT
```
