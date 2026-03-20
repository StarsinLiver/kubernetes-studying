<!-- 크론잡(CronJob) -->
# 데이터베이스를 예로 들면 데이터베이스 백업을 주기적으로 하는경우가 많은데 이때는 크론잡(CronJob)을 사용하면 유용하다.
# 크론잡은 잡을 관리하는 컨트롤러이다.
# 주기적인 일정에 따라 잡을 생성한다.

# 병렬로 진행하는 작업을 포함해서 잡으로 수행할수 있는 작업은 모두 크론잡으로도 수행할 수 있다.
# 잡을 실행할 스케줄은 리눅스의 cron 포맷으로 작성한다.
# 이 포맷을 '1분에 한번' , '매일 한번' 같은 패턴부터 '매주 일요일 오전 4 ~6시' 같은 복잡한 조건까지 쉽게 나타낼 수 있다.

<!-- todo-db-backup-cronjob.yaml -->
apiVersion: batch/v1beta1       # 현재 batch/v1 로 바뀌었음
kind: CronJob
metadata:
  name: todo-db-backup
  labels:
    kiamol: ch08
spec:
  schedule: "*/2 * * * *"        # see https://crontab.guru # 작업을 2분에 한번씩 실행
  concurrencyPolicy: Forbid      # 현재 작업이 끝나기 전까지 새 작업을 실행하지 않음
  jobTemplate:                   # 잡 템플릿
    spec:
      template:
        spec:
          restartPolicy: Never   # 필수 옵션
##########################################################
          containers:            # 파드 정의
          - name: backup
            image: postgres:11.6-alpine
            command: ['sh', '-c', 'pg_dump -h $POSTGRES_SECONDARY_FQDN -U postgres -F tar -f "/backup/$(date +%y%m%d-%H%M).tar" todo']
            envFrom:
            - configMapRef:
                name: todo-db-env
            env:
            - name: PGPASSWORD
              valueFrom:
                secretKeyRef:
                  key: POSTGRES_PASSWORD
                  name: todo-db-secret
            volumeMounts:
              - name: backup
                mountPath: "/backup"
          volumes:
            - name: backup
              persistentVolumeClaim:
                claimName: todo-db-backup-pvc

<!-- 보류 모드 -->
# 크론잡은 작업을 완료한 잡과 파드를 자동으로 삭제하지 않는다 이 일은 TTL(Time-To-Line) 컨트롤러의 몫이지만, 이 기능은 아직 알파 단계에 머무르고 있기 때문에 지원하지 않는 플랫폼이 많다.
# TTL 컨트롤러가 없다면 작업을 마친 후 필요없어진 잡과 파드는 직접 삭제해야한다.
# 크론잡을 다시 활성화 될 때까지 작업이 수행되지 않도록 보류 모드로 전환할 수도 있다

<!-- todo-db-backup-cronjob-suspend.yaml -->
apiVersion: batch/v1
kind: CronJob
metadata:
  name: todo-db-backup
  labels:
    kiamol: ch08
spec:
  schedule: "*/2 * * * *"        # see https://crontab.guru
  concurrencyPolicy: Forbid
  suspend: true                  # 보류 모드로 설정
  ...



<!--! 크론잡의 주의점 -->
# 잡과 크론잡을 활용하다보면, 이들의 단순한 정의가 배치 작업의 복잡도를 가려주고 흥미로운 실패 모드를 만들어 낸것을 알 수 있다.
# 잡은 파드 속 컨테이너를 재시작하거나 다른 노드에서 새 파드를 만드는 방법으로 작업을 완료할 수 있지만, 크론잡 입장에서는 배치 작업 시간이 길어지면 작업이 끝나기도 전에 다음 주기로 돌아와 여러개의 파드가 동작하는 일이 생긱 수 있다.