<!-- 스테이트풀셋 이용한 안정성 모델링 -->
# 스테이트풀셋(StatefulSet)은 그 이름에서 기능을 쉽게 짐작할 수 있다.
# 안정된 프레임워크에서 시작하는 애플리케이션에 스케일링 기능을 제공하는 파드 컨트롤러이다.
# 우리가 레플리카셋을 배치하면 무작위 이름이 부여된 파드가 만들어 졌다.
# 이들 파드는 도메인 네임으로 일일이 구분되며 레플리카셋이 이들을 병렬로 함께 실행했다.

# 반면 스테이트풀셋은 도메인 네임으로 각각 식별되는 규칙적인 이름이 부여된 파드를 생성한다.
# 또한 이들 파드는 첫번째 파드가 Running 되면 그 다음 파드가 생성되는 식으로 순서대로 생성된다.

# 클스터 애플리케이션이 스테이트풀셋으로 모델링하기 적합한 대상이다.
# 이들 애플리케이션은 대개 미리 정해진 주 인스턴스와 부 인스턴스가 함께 동작하며 고가용성을 확보한다.
# 부 인스턴스의 수를 늘리는 방식으로 스케일링 되는데, 부 인스턴스는 데이터 동기화를 위해 주 인스턴스와 통신이 가능해야한다.
# 디플로이먼트로는 이런 애플리케이션을 모델링할 수 없는데, 그 이유는 레플리카셋에서는 특정 파드를 주 인스턴스로 지정할 수 없기 때문이다.
# 억지로 모델링을 하면 모든 파드가 주 인스턴스가 되거나 주 인스턴스가 없어 엉망이 되는 결과를 보기 쉽다.

# 이 패턴(p.247)은 데이터베이스 외에도 유용한 곳이 많은데 레거시 애플리케이션 중 많은 수가 정적인 실행시점 환경과 쿠버네티스와 맞지 않는 안정성을 가정하고 만들어 졌다.

# 스테이트풀셋을 이런 안정성을 쿠버네티스 안으로 포용하기 위한 컨트롤러이다.

<!-- 기본적인 스테이트풀셋 -->
<!-- Todo : todo-db.yaml -->
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: todo-db
  labels:
    kiamol: ch08
spec:
  selector:               # 스테이트풀셋에도 셀렉터가 쓰인다.
    matchLabels:
      app: todo-db
  serviceName: todo-db    # 스테이트풀셋은 연결된 서비스가 필요하다.
  replicas: 2
  template:
    metadata:
      labels:
        app: todo-db
    spec:
      containers:
        - name: db
          image: postgres:11.6-alpine
          env:
          - name: POSTGRES_PASSWORD_FILE              
            value: /secrets/postgres_password     # 해당 경로 값("kiamol-2*2*")을 환경변수로 지정했다.
          volumeMounts:
            - name: secret
              mountPath: "/secrets"               # /secrets 의 경로를 마운트 했다.
      volumes:
        - name: secret                            # 볼륨 이름 : secret
          secret:
            secretName: todo-db-secret            
            defaultMode: 0400
            items:
            - key: POSTGRES_PASSWORD              
              path: postgres_password             # /postgres_password 경로로 시크릿 키를 설정했다.

<!-- + 볼륨클레임템플릿 -->
# 실행결과 - 각 파드마다 동적으로 영구볼륨클레임이 생성된다.
<!-- Todo : with-pvc.yaml -->
apiVersion: v1
kind: Service                  # 헤드리스 서비스
metadata:
  name: sleep-with-pvc
  labels:
    kiamol: ch08
spec:
  selector:
    app: sleep-with-pvc
  clusterIP: None
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sleep-with-pvc
  labels:
    kiamol: ch08
spec:
  selector:
    matchLabels:
      app: sleep-with-pvc
  serviceName: sleep-with-pvc     # 헤드리스 서비스 이름
  replicas: 2
  template:
    metadata:
      labels:
        app: sleep-with-pvc
    spec:
      containers:
        - name: sleep
          image: kiamol/ch03-sleep
          volumeMounts:
            - name: data
              mountPath: /data
  volumeClaimTemplates:            # 별도의 스토리지가 마운트되도록 volumeClaimTemplates 를 기술
  - metadata:
      name: data                   # 파드 볼륨 마운트 이름
      labels:               
        kiamol: ch08
    spec:                          # 일반적인 영구볼륨클레임의 정의
      accessModes: 
       - ReadWriteOnce
      resources:
        requests:
          storage: 5Mi