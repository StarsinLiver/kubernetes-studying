<!-- 비밀값을 이용하여 민감한 정보가 담긴 설정값 다루기 -->
# 컨피그맵에는 사용하지 말아야 하는 경우는 민감한 데이터를 다룰때 뿐이다.
# 컨피그맵은 텍스트파일을 잘 추상화한 객체일 뿐 그 내용을 보호할 수 있는 보안적 수단이 전혀 없다.
# 쿠버네티스에는 외부에 유출하기 곤란한 민감한 설정값을 위해 비밀값이 있다.

<!-- 시크릿 정의 -->
PS C:\ALL_WORKSPACE\0_GIT\kiamol\ch04> kubectl create secret generic sleep-secret-literal --from-literal=secret=shh..

<!-- 시크릿 정의 -->
apiVersion: v1
kind: Secret
metadata:
  name: todo-db-secret-test
type: Opaque
stringData:
  POSTGRES_PASSWORD: "kiamol-2*2*"

<!-- 디플로이먼트에서의 시크릿 가져오기 -->
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sleep
spec:
  selector:
    matchLabels:
      app: sleep
  template:
    metadata:
      labels:
        app: sleep
    spec:
      containers:
        - name: sleep
          image: kiamol/ch03-sleep
          env:                    # 환경변수 정의
          - name: KIAMOL_SECRET   # 컨테이너에 전달될 환경 변수 이름
            valueFrom:            # 환경변수의 값은 외부에서 도입
              secretKeyRef:       # 비밀값에서 도입              
                name: sleep-secret-literal  # 비밀값 이름
                key: secret                 # 비밀값의 항목 이름


<!-- 디플로이먼트에서의 시크릿 마운트 -->
apiVersion: v1
kind: Service
metadata:
  name: todo-db
spec:
  ports:
    - port: 5432
      targetPort: 5432
  selector:
    app: todo-db
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: todo-db
spec:
  selector:
    matchLabels:
      app: todo-db
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
            value: /secrets/postgres_password   # 설정파일이 마운트될 경로
          volumeMounts:                         # 볼륨 마운트 설정
            - name: secret                      # 마운트할 볼륨 이름
              mountPath: "/secrets"
      volumes:
        - name: secret                          
          secret:                               # 비밀값에서 볼륨 생성
            secretName: todo-db-secret-test     # 볼륨을 만들 비밀값 이름
            defaultMode: 0400                   # 파일의 권한 설정
            items:                              # 비밀값의 특정 데이터 항목 지정가능
            - key: POSTGRES_PASSWORD            # POSTGRES_PASSWORD: "kiamol-2*2*"
              path: postgres_password           

