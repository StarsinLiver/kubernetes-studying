<!-- deploy.yaml -->
# 디플로이먼트는 API 버전 1 에 속한다.
apiVersion: apps/v1
kind: Deployment

# 디플로이먼트의 이름을 정해야한다.
metadata:
  name: hello-kiamol-4

# 디플로이먼트가 자신의 관리대상을 결정하는 레이블 셀렉터가 정의된다.
# 여기에서는 app 레이블을 사용하는데, 레이블은 임의의 키-값 쌍이다.
spec:
  selector:
    matchLabels:
      app: hello-kiamol-4

# 이 템플릿은 디플로이먼트가 파드를 만들 때 쓰인다.
  template:
# 디플로이먼트 정의 속 파드의 정의에는 이름이 없다.
# 그 대신 레이블 셀렉터와 일치하는 레이블을 지정해야한다.
    metadata:
      labels:
        app: hello-kiamol-4
# 파드의 정의에는 컨테이너 이름과 이미지 이름을 지정한다.
    spec:
      containers:
        - name: web
          image: kiamol/ch02-hello-kiamol


<!-- 컨테이너가 컴퓨팅 공간을 공유하도록 업데이트 -->
<!-- 재시작 조건 -->
# 다음과 같은 재시작 조건을 기억해두자
1) 초기화 컨테이너를 가진 파드가 대체될 때 새 파드는 초기화 컨테이너를 모두 실행한다. 따라서 초기화 로직을 반복적으로 실행할 수 있는 것이여야 한다.
2) 초기화 컨테이너의 이미지를 변경하면 파드 자체를 재시작한다. 초기화 컨테이너는 다시 한번 실행되며, 애플리케이션 컨테이너도 모두 교체된다.
3) 파드 정의에서 애플리케이션 컨테이너의 이미지를 변경하면 애플리케이션 컨테이너가 대체 된다. 초기화 컨테이너는 재시작하지 않는다.
4) 애플리케이션 컨테이너가 종료되면 파드가 애플리케이션 컨테이너를 재생성한다. 대체 컨테이너가 준비될 때까지 파드는 완전한 동작 상태가 아니게 되며, 서비스에서 트래픽을 전달 받지 못한다.

# 파드는 단일한 컴퓨팅 환경이다. 하지만 이 환경을 구성하는 부품이 여러 개일 때는 각각의 실패 시나리오를 검토하고 애플리케이션이 의도대로 동작하는지 확인해야한다.

# 아직 다루지 않은 파드 내 환경의 마지막 부분이 있다. 바로 컴퓨팅 계층이다.
# 파드 속 컨테이너는 네트워크 주소와 파일 시스템의 일부를 공유할 수 있다.
# 하지만 컨테이너 경계를 넘어 서로의 프로세스에는 접근할 수 없다.

# 파드 정의에 shareProcessNamespace:true 설정을 추가하면 이런 접근이 가능하다. 파드의 모든 컨테이너가 컴퓨팅 공간을 공유하며 서로의 프로세스를 볼 수 있게된다.

<!-- Todo : sleep-with-server-shared -->
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sleep
  labels:
    kiamol: ch07
spec:
  selector:
    matchLabels:
      app: sleep
  template:
    metadata:
      labels:
        app: sleep
        version: shared
    spec:
      shareProcessNamespace: true       # 컨테이너가 컴퓨팅 공간을 공유하도록 업데이트
      containers:
        - name: sleep
          image: kiamol/ch03-sleep        
        - name: server
          image: kiamol/ch03-sleep  
          command: ['sh', '-c', "while true; do echo -e 'HTTP/1.1 200 OK\nContent-Type: text/plain\nContent-Length: 7\n\nkiamol' | nc -l -p 8080; done"]
          ports:
            - containerPort: 8080
              name: http