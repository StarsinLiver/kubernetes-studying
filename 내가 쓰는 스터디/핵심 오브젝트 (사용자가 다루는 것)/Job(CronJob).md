## 목차

- [Job이 무엇인가](#job이-무엇인가)
  - [1️⃣ Job이 무엇인가](#1️⃣-job이-무엇인가)
  - [2️⃣ Deployment와 차이](#2️⃣-deployment와-차이)
  - [3️⃣ Job YAML](#3️⃣-job-yaml)
  - [4️⃣ Job 내부 동작](#4️⃣-job-내부-동작)
  - [5️⃣ Job 완료 조건](#5️⃣-job-완료-조건)
  - [6️⃣ 병렬 Job](#6️⃣-병렬-job)
  - [7️⃣ 실패 처리](#7️⃣-실패-처리)
  - [8️⃣ 실제 Job의 예시](#8️⃣-실제-job의-예시)
  - [9️⃣ CronJob](#9️⃣-cronjob)
  - [🔟 CronJob YAML](#-cronjob-yaml)
  - [11.CronJob 실행 흐름](#11cronjob-실행-흐름)
  - [12. CronJob 실제 실행 구조](#12-cronjob-실제-실행-구조)
  - [13. CronJob 실제 확인](#13-cronjob-실제-확인)

---

# Job이 무엇인가

## 1️⃣ Job이 무엇인가

🔹 한 줄 정의

Job은 "작업을 한번 완료시키는 Pod"을 관리하는 Controller다.
<br> 즉, Pod가 성공(exit code 0)하면 끝

또는 batch workload (작업용 Pod) 이다.

[예시]

```
DB migration
batch data processing
image processing
backup
ML training
report 생성
```

## 2️⃣ Deployment와 차이

Deployment

```
Pod nginx Pod가 종료되면 ReplicaSet → 다시 생성
왜냐하면 서비스는 계속 살아있어야 하니까
```

Job Pod

```
작업 수행
→ 종료
→ 끝
```

재생성 안 한다.

## 3️⃣ Job YAML

```yaml
apiVersion: batch/v1
kind: Job

metadata:
  name: pi-job

spec:
  template:
    spec:
      containers:
        - name: pi
          image: perl
          command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]

      restartPolicy: Never
```

```
중요한 것은 restartPolicy: Never 이다.

이건 작업용 Pod 이라서 필요하다.
```

## 4️⃣ Job 내부 동작

Job Controller 위치

```
kube-controller-manager
```

구조

```
Job Controller
     │
     ├── watch Job
     ├── watch Pod
     │
     └── reconcile
```

핵심 로직

```
Job 생성
 → Pod 생성
 → Pod 성공 여부 확인
```

## 5️⃣ Job 완료 조건

Pod 상태가 Succeeded 되면

```
Job complete
```

확인

```
kubectl get job
```

결과

```
NAME      COMPLETIONS   DURATION
pi-job    1/1           12s
```

## 6️⃣ 병렬 Job

Job은 병렬 작업도 가능하다.

예시로

```
100개 작업을
동시에 10개 Pod씩 실행.
```

YAML

```yaml
spec:
  completions: 100
  parallelism: 10
```

의미

```
총 작업 수 = 100
동시 실행 = 10
```

실행 흐름

```
10 pod 실행
→ 완료
→ 다음 10
→ 완료
```

## 7️⃣ 실패 처리

Job 옵션을 사용할 수 있다.

```
backoffLimit
```

예시

```
backoffLimit: 3
```

의미

```
3번 실패하면 Job 실패
```

## 8️⃣ 실제 Job의 예시

예를 들어

```
database migration
```

컨테이너 실행

```
alembic upgrade
```

끝나면 컨테이너 종료

이럴 때 Deployment 쓰면 안 된다. 왜냐하면 Deployment를 사용하면 migration 계속 실행하기 때문이다. 그래서 Job 사용을 사용한다.

## 9️⃣ CronJob

```
CronJob은 Job을 주기적으로 실행하는 Controller다.
```

Linux의 crontab과 동일하다.

예

```
매일 02:00 백업
```

CronJob

```
→ Job 생성
→ Pod 실행
```

## 🔟 CronJob YAML

```yaml
apiVersion: batch/v1
kind: CronJob

metadata:
  name: backup

spec:
  schedule: "0 2 * * *"

  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: backup
              image: backup-image

          restartPolicy: OnFailure
```

## 11.CronJob 실행 흐름

```
CronJob Controller
      │
      │ 시간 체크
      ↓
시간 도달
      ↓
Job 생성
      ↓
Pod 실행
```

## 12. CronJob 실제 실행 구조

```
CronJob
   ↓
Job
   ↓
Pod
```

즉, CronJob은 Job factory다.

## 13. CronJob 실제 확인

CronJob 확인

```
kubectl get cronjob
```

Job 확인

```
kubectl get jobs
```

Pod 확인

```
kubectl get pods
```
