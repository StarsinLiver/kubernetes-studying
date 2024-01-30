<!-- 고장을 일으킨 파드 재시작하기 : 리브니스 프로브 -->
# 리브니스 프로브의 상태 체크 매커니즘은 레디니스 프로브와 동일하다.
# 하지만 고장을 일으킨 파드에 대한 조치는 레디니스 프로브와 다르다.

# 리브니스 프로브의 조치는 컴퓨팅 수준이라고 할 수 있는 파드 재시작이다.
# 여기에서 말하는 재시작은 파드 컨테이너의 재시작이다. 파드 자체는 원래 있던 노드에 그대로 남는다.

# 밑의 무작위 숫자 API정의에 포함된 리브니스 프로브의 설정이다.
# 여기에서도 HTTP GET 요청을 통해 상태를 체크하지만, 몇가지 추가된 설정잉 있다.
# 파드 재시작은 엔드포인트 목록에서 제외하는 것 보다는 훨씬 적극적인 조치다. 그런만큼 이 조치를 취할 시점을 더 명확히 하는 추가적인 설정이 잇다.

<!-- # api.with0readiness-and-liveness.yaml -->
apiVersion: apps/v1
kind: Deployment
metadata:
  name: numbers-api
  labels:
    kiamol: ch12
spec:
  replicas: 2
  selector:
    matchLabels:
      app: numbers-api
  template:
    metadata:
      labels:
        app: numbers-api
        version: v3
    spec:
      restartPolicy: Always
      containers:
        - name: api
          image: kiamol/ch03-numbers-api
          ports:
            - containerPort: 80
              name: api
          env:
            - name: FailAfterCallCount
              value: "1"
          readinessProbe:                 # < 레디니스 프로브 >
            httpGet:                      # HTTP GET 요청
              path: /healthz              # .../healthz 80번 포트로 
              port: 80
            periodSeconds: 5              # 5초간격으로 전송
          livenessProbe:                  # < 리브니스 프로브 >
            httpGet:                      # HTTP GET 요청을 통해 상태 체크
              path: /healthz              # .../healthz 80번 포트로 
              port: 80
            periodSeconds: 10             # 10초 간격으로 전송
            initialDelaySeconds: 10       # 첫번째 상태 체크 전 10초간 대기
            failureThreshold: 2           # 상태 체크 실패 두 번까지는 용인


# 리브니스 프로브 설정은 파드 정의이므로 이번 업데이트를 반영하면 새로운 대체 파드가 생성된다.
# 이번에도 파드가 고장을 일으키면 레디니스 프로브가 서비스에서 파드를 제외하여 그 다음에는 리브니스 프로브가 파드를 재시작해서 파드가 서비스에 복귀한다.

<!-- exec command -->
          readinessProbe:
            tcpSocket:                # 레디니스 프로브는 
              port: 5432              # 데이터베이스가 이 포트를 주시하는지 체크
            periodSeconds: 5
          livenessProbe:              # 리브니스 프로브는 명령행 도구를 실행하여
            exec:                     # 데이터베이스가 동작중인지 체크
              command: ["pg_isready", "-h", "localhost"]
            periodSeconds: 10
            initialDelaySeconds: 10