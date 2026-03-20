<!-- 컨피그맵과 비밀값으로 애플리케이션 설정하기 -->
# 컨테이너에서 애플리케이션을 실행할 때 대표적인 장점 중 하나는 다양한 환경 간 차이를 원천적으로 없앨 수 있다는 점이다. 테스트 환경부터 운영 환경까지 전체 배포 절차가 컨테이너 하나로 이미지로 진행될 수 있기 때문에 모든 환경에서 완전히 동일한 바이너리가 사용된다.
# 물론 환경 간 차이가 아주 없을 수는 없다. 이를 위해 컨테이너 환경별로 설정값을 주입해야 한다.

# 쿠버네티스에서 컨테이너에 설정값을 주입하는데 쓰이는 리소스는 컨피그맵(ConfigMap) 과 비밀값(Secret) 두 가지다.
# 이 두가지 리소스 모두 포맷 제한 없이 데이터를 보유할 수 있다.
# 이 데이터는 클러스터 속에서 다른 리소스와 독립적인 장소에 보관된다.

<!-- 환경변수가 설정된 값 -->
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
          env:                    # 이 아래로 환경변수가 정의
          - name: KIAMOL_CHAPTER  # 새로운 환경 변수의 이름 정의
            value: "04"           # 새로운 환경변수의 값 정의


<!-- 컨피그맵 -->
# 컨피그맵은 파드에서 읽어 들이는 데이터를 저장하는 리소스이다. 
# 데이터의 형태는 한 개 이상의 키-값쌍 , 텍스트 , 바이너리 파일까지 다양하다.

# 키-값 쌍을 저장했다면 파드에서 이를 환경변수 형태로 주입할수 있다.
# 텍스트를 저장했다면 JSON , XML , YAML , TOML , INI 등 설정 파일을 파드에 전달할 수 있다.
# 바이너리 파일 형태로 된 라이선스 키를 전달하는 것도 가능하다.

# 또한 하나에 여러개의 컨피그맵을 전달할 수 있고 , 반대로 하나의 컨피그맵을 여러 파드에 전달할 수도 있다.

# 주의점 
1) 파드가 컨피그맵을 읽어서 온다.
2) 컨피그맵은 특정 애플리케이션 전용으로 사용하거나 여러 파드에서 공유하는 형태로도 사용가능
3) 컨피그맵은 읽기 전용이다.
4) 파드에서 컨피그맵은 수정할 수 없다.

<!-- 두 개 이상의 설정파일을 담은 컨피그맵 -->
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-literal
data:
  config.json: |-         # 기존 설정 파일
    {
      "ConfigController": {
        "Enabled" : true
      }
    }
  logging.json: |-        # 볼륨 마운트로 전달될 두 번째 설정파일
    {
      "Logging": {
        "LogLevel": {
          "ToDoList.Pages" : "Debug"
        }
      }
    }

<!-- 디플로이먼트에서의 컨피그맵 정의 -->
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
          env:
          - name: KIAMOL_SECTION
            valueFrom:
              configMapKeyRef:              # 이 값은 컨피그맵에서 읽어 들이라는 의미            
                name: config-literal        # 컨피그맵 이름
                key: config.json            # 컨피그맵에서 읽어 들일 항목 이름

<!-- 디플로이먼트에서의 컨피그맵 정의 -->
# 한 줄에 하나씩 환경 변수 정의
KIAMOL_CHAPTER=ch04
KIAMOL_SECTION=ch04-4.1
KIAMOL_EXERCISE=try it now

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
          envFrom:                          # envFrom 항목에서 컨피그맵에서 읽어 올
          - configMapRef:                   # 환경 변수를 정의한다.
              name: sleep-config-env-file 
          env:                              # 기존 env 항목
          - name: KIAMOL_SECTION
            valueFrom:
              configMapKeyRef:              
                name: sleep-config-literal
                key: kiamol.section

# 환경 변수이름이 중복되는 경우 env 항목에서 정의된 값이 envFrom 항목에서 정의된 값에 우선한다.
# 이런 우선순위는 컨테이너 이미지나 컨피그맵에서 정의된 환경변수의 값을 파드 정의에서 간단히 수정할 수 있어 편리하다.

<!-- 컨피그맵에 담긴 설정값 데이터 주입하기 -->
# 환경 변수 외에 설정값을 전달하는 또 다른 방법은 컨테이너 파일 시스템 속 파일로 설정값을 주입하는 것이다.
# 컨테이너 파일 시스템은 컨테이너 이미지와 그 외 출처에서 온 파일로 구성되는 가상 구조이다.
# 쿠버네티스는 컨테이너 파일 시스템 구성에 컨피그맵도 추가할 수 있다.

<!-- 볼륨 (Volumn) 과 볼륨 마운트(voulumn mount) -->
# 첫번째는 컨피그맵에 담긴 데이터를 파드로 전달하는 볼륨이다.
# 두번째는 컨피그맵을 읽어 들인 볼륨을 파드 컨테이너 특정 경로에 위치시키는 볼륨 마운트이다.

<!-- todo-list/todo-web-dev.yaml -->
apiVersion: apps/v1
kind: Deployment
metadata:
  name: todo-web
spec:
  selector:
    matchLabels:
      app: todo-web
  template:
    metadata:
      labels:
        app: todo-web
    spec:
      containers:
        - name: web
          image: kiamol/ch04-todo-list 
          volumeMounts:                 # 컨테이너에 볼륨을 마운트한다.
            - name: config              # 마운트할 볼륨 이름
              mountPath: "/app/config"  # 볼륨이 마운트될 경로
              readOnly: true            # 볼륨을 읽기 전용으로
      volumes:                          # 볼륨은 파드 수준에서 정의된다.
        - name: config                  # 이 이름이 볼륨 마운트의 이름과 일치해야한다.
          configMap:                    # 볼륨의 원본은 컨피그맵이다.
            name: config-literal        # 내용을 읽어올 컨피그맵 이름

<!-- todo-web-deb-no-logging.yaml -->
apiVersion: apps/v1
kind: Deployment
metadata:
  name: todo-web
spec:
  selector:
    matchLabels:
      app: todo-web
  template:
    metadata:
      labels:
        app: todo-web
    spec:
      containers:
        - name: web
          image: kiamol/ch04-todo-list
          volumeMounts:
            - name: config              # 컨피그맵 볼륨 마운트
              mountPath: "/app/config"  # 볼륨이 마운트될 경로
              readOnly: true
      volumes:
        - name: config
          configMap:
            name: todo-web-config-dev   # 컨피그맵 지정
            items:                      # 컨피그맵에서 전달할 데이터 항목 지정
            - key: config.json          # config.json 항목 지정
              path: config.json         # config.json 파일로 전달하도록 지정