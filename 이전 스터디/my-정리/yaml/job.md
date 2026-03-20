<!-- 잡(Job)이란? -->
# 잡은 파드 정의를 포함하는데, 이 파드가 수행하는 배치 작업이 완료되는 것을 보장해주는 중요한 역할을 한다.
# 잡은 비단 유상태 애플리케이션에만 필요한 리소스가 아니다.
# 스케줄링이나 재시도 로직을 직접 신경 쓰지않아도 거의 모든 배치 작업을 수행할 수 있는 표준적인 수단이다.
# 잡이 포함하는 파드는 어떤 컨테이너 이미지라도 실행할 수 있다.

# 유일한 조건은 프로세스가 작업을 마치고 종료되어야 한다는 것이다.

# 잡에 포함된 컨테이너가 스스로 종료되지 않는 프로세스를 실행한다면 이 잡은 영원히 완료되지 않을것이다.

<!-- 잡의 여러 입력값 -->
# 잡은 파드 정의에서 배치 작업에 필요한 어떤 컨테이너 이미지라도 사용할 수 있다.
# 또한 동일한 작업에 여러 입력값을 설정할 수도 있다.
# 여러 입력값에 대해 잡을 생성하면 입력값 하나마다 파드가 생성되어 배치 작업이 클러스터에서 병렬로 진행된다.

1) completions :
잡을 실행할 횟수를 지정한다. 기술하려는 잡이 작업 큐를 처리하는 형태라면, 애플리케이션 컨테이너에 다음 처리할 항목을 받아 올 방법을 알려주어야한다.
잡 자체는 지정된 수의 파드가 원하는 만큼 작업을 완료하는 지만 확인한다.

2) parallelism : 
완료 건수가 둘 이상인 잡에서 동시에 실행할 파드 수를 지정한다. 이 필드값으로 잡을 수행하는 속도와 작업에 사용할 클러스터의 연산능력을 조절할 수 있다.


<!-- Todo : job.yaml -->
apiVersion: batch/v1
kind: Job                 # 리소스 유형 : 잡
metadata:
  name: pi-job
  labels:
    kiamol: ch08
spec:                     # 일반적인 파드의 정의이다.
  template:
    spec:
      containers:
        - name: pi
          image: kiamol/ch05-pi
          command: ["dotnet", "Pi.Web.dll", "-m", "console", "-dp", "50"]
      restartPolicy: Never    # 컨테이너가 실패하면 대체 파드가 생성된다. (필수 옵션)



<!-- ch12-3.md 에서 헬름을 이용한 업데이트 -->

<!-- todo-list/v3/todo-db-test-job.yaml -->
# 헬름에서는 잡이 많이 활용된다. 잡 정의에는 실패했을 경우 재시도 횟수와 성공을 판단하는 조건이 포함된다.
# 일반적인 업그레이드는 제대로 진행되며, 업그레이드가 끝난 후 테스트 잡이 실행된다.

apiVersion: batch/v1
kind: Job                                 # 잡의 정의다.
metadata:
  name:  {{ .Release.Name }}-db-test
  labels:
    kiamol: ch12
  annotations:
    "helm.sh/hook": test                  # 테스트 중 실행할 잡임을 명기
spec:
  completions: 1
  backoffLimit: 0                         # 한 번만 실행하며 재시도를 하지 않음
  template:
    spec:                                 # SQL 질의를 실행하는 컨테이너를 정의
      restartPolicy: Never
      containers:
        - name: db
          image: postgres:11.8-alpine          
          env:
          - name: PGHOST 
            value: {{ .Release.Name }}-db
          - name: PGDATABASE 
            value: todo
          - name: PGUSER
            value: postgres
          - name: PGPASSWORD
            valueFrom:
              secretKeyRef:
                key: POSTGRES_PASSWORD
                name: todo-db-secret
          command: ["psql", "-c", "SELECT COUNT(*) FROM \"public\".\"ToDos\""]