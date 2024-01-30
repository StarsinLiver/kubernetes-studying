<!-- 쿠버네티스는 어떻게 애플리케이션을 스케일링하는가 -->
# 파드는 쿠버네티스에서 컴퓨팅의 단위이다.
# 앞서 2장에서 사람이 직접 파드를 실행하는 경우는 드물고, 주로 파드를 관리하는 다른 리소스를 정의하여 사용한다고 설명했다
# 이런 리소스를 컨트롤러라고 하낟.
# 그 후로 컨트롤러 중에서도 디플로이먼트를 주로 사용했다.

# 컨트롤러 리소스 정의는 파드의 템플릿을 포함한다.
# 컨트롤러 리소스는 파드를 생성하고 대체하는데 이 템플릿을 사용하며, 이 템플릿으로 똑같은 파드의 레플리카를 여러개 만들 수도 있다.

# 쿠버네티스에서 가장 많이 사용하게 될 리소스는 디플로이먼트일 것이다. 다만, 디플로이먼트는 사실 직접 파드를 관리하지 않는다.
# 파드를 직접 관리하는 역할은 레플리카셋(ReplicaSet)의 몫이다.

# 디플로이먼트 정의와 차이점은 리소스 유형과 파드 수를 기재한 replicas 필드가 있다는 것 뿐이다.
# replicas 필드값이 1이므로 이 레플리카셋은 파드를 한 개만 실행한다.

<!-- Todo : sleep.yaml -->
apiVersion: apps/v1
kind: ReplicaSet              # 디플로이먼트와 정의 내용이 거의 같다.
metadata:
  name: whoami-web
  labels:
    kiamol: ch06
spec:
  replicas: 1               
  selector:                   # 관리 대상 파드를 찾기 위한 셀렉터
    matchLabels:            
      app: whoami-web
  <!-- template:
    metadata:
      labels:
        app: whoami-web
    spec:
      containers:
        - image: kiamol/ch02-whoami
          name: web
          ports:
            - containerPort: 80
              name: http -->