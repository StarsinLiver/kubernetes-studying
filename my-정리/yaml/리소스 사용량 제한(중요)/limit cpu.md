<!-- CPU 사용량 제한 -->
# CPU 사용량 역시 컨테이너별 사용량 제한이나 네임스페이스별 총 사용량 제한을 둘 수 있다.
# 하지만 메로리 사용량 제한과는 약간 적용되는 방식이 다르다.
# CPU 사용량 제한이 걸리더라도 컨테이너는 원하는 대로 CPU를 사용할 수 있다. 설사 제한을 초과하더라도 파드가 재시작하거나 하는 일은 없다.
# 어떤 컨테이너의 CPU 사용량을 코어 하나의 절반 정도로 설정했다면 노드의 다른 컨테이너가 사용할 수 있는 코어가 노드에 남아 있는 한 cpu 사용량을 100%까지 쓸 수 있다.
# 원주율 계산 애플리케이션은 CPU를 집중적으로 사용하는 애플리케이션이다.
# 이 애플리케이션에 CPU사용량 제한을 걸어 보자.

<!-- CPU 제한 yaml -->
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pi-web
  labels:
    kiamol: ch12
spec:
  selector:
    matchLabels:
      app: pi-web
  template:
    metadata:
      labels:
        app: pi-web
    spec:
      containers:
        - image: kiamol/ch05-pi
          command: ["dotnet", "Pi.Web.dll", "-m", "web"]
          name: web
          ports:
            - containerPort: 80
              name: http
          resources:          # 리소스 제한
            limits:           
              cpu: 250m       # CPU 250밀리 코어적용 === 0.25코어에 해당한다.


# 쿠버네티스는 cpu 사용량을 고정된 단위로 설정한다.
# 여기에서 1은 코어 하나를 의미한다. 한 컨테이너가 여러개의 코어를 사용하게 할 수도 있고, 하나의 코어를 밀리코어(1/1000 개) 단위로 나뉘어 쓸 수도 있다.
# 애플리케이션 컨테이너 적용된 CPU 사용량 제한이다. 250밀리코어, 즉 코어 1/4개로 제한이 적용되었다.