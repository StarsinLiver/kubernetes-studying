<!-- 정상 파드에만 트래픽 라우팅하기 : 레디니스 프로브 -->

# 쿠버네티스는 파드 컨테이너가 실행중인지 파악할 수 있지만, 컨테이너 속에 동작하는 애플리케이션 상태가 정상인지는 알 수 없다.

# 따라서 쿠버네티스에는 컨테이너 프로브(Container probe)를 통해 애플리케이션의 정상 상태 여부를 판단하는 기능이 있다.

# 도커 이미지에도 헬스체크 기능을 설정할 수 있지만, 쿠버네티스는 자신의 프로브를 우선하기 때문에 헬스체크 설정을 무시한다.

# 프로브는 파드 정의에 기술되며, 고정된 스케줄로 실행되어 애플리케이션의 특정한 측면을 시험하고 애플리케이션 상태가 현재 정상이라고 판단할 수 있는 지표를 반환한다.

# 프로브가 컨테이너 상태가 비정상이라고 응답한다면 쿠버네티스가 조치에 나선다. 다만 조치 내용은 프로브 유형에 따라 달라진다.

# 레디니스 프로브(readiness probe)는 네트워크 수준에서 조치되는데, 네트워크 요청을 처리하는 컴포넌트에 대한 라우팅을 관리하는 조치다.

# 파드 컨테이너의 상태가 비정상으로 판정되면 해당 파드는 준비 상태에서 제외되며, 서비스의 활성 파드 목록에서도 제외된다.

# 레디니스 프로브는 일시적인 과부하 문제를 해결하는데 적합하다.

# 일부 파드에 과부하가 걸려 모든 요청에 상태 코드 503을 반환하고 있다면, 레디니스 프로브의 확인 요청에도 정상을 의미하는 상태 코드 200이 아닌 상태 코드 503을 반환할 것이다.

# 따라서 이들 파드는 서비스에서 제외되어 더 이상 요청을 받지 않는다.

# 파드가 서비스에서 제외된 후에도 레디니스 프로브는 과부하 상태의 파드가 복귀할 기회를 주는데, 이때 상태 확인에서 정상으로 돌아오면 파드는 다시 서비스의 엔드포인트 목록에 복귀한다.

<!-- api-with-readiness.yaml -->
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
        version: v2
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
          readinessProbe:                   # 프로브는 컨테이너 수준에서 정의된다.
            httpGet:
              path: /healthz                # 이 URL에 HTTP GET 요청을 보낸다.
              port: 80
            periodSeconds: 5                # 5초에 한번씩 상태를 확인한다.
            
# 쿠버네티스에는 여러 유형의 컨테이너 프로브가 있다. 여기 나온 컨테이너 프로브는 웹 애플리케이션이나 API상태를 확인하기에 적합한 HTTP GET 요청을 사용한다.
# 프로브는 5초마다 엔드포인트 /healthz를 호출하여 그 결과를 쿠버네티스에 보고하는데, 응답의 HTTP 상태 코드가 200에서 399 사이면 정상 상태를 보고하고 그 외의 상태 코드는 실패를 보고한다.

<!-- tcpSocket -->
          readinessProbe:
            tcpSocket:                # 레디니스 프로브는 
              port: 5432              # 데이터베이스가 이 포트를 주시하는지 체크
            periodSeconds: 5
          livenessProbe:              # 리브니스 프로브는 명령행 도구를 실행하여
            exec:                     # 데이터베이스가 동작중인지 체크
              command: ["pg_isready", "-h", "localhost"]
            periodSeconds: 10
            initialDelaySeconds: 10