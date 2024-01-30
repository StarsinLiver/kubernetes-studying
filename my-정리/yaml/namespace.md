# 네임스페이스에 속한 객체는 다른 네임스페이스와 격리되므로 네임스페이스만 바꾸면 동일한 애플리케이션을 동일한 객체이름으로 여러 벌 배치할 수 있다.
# 리소스는 다른 네임스페이스에 속한 리소스 존재를 알 수 없다.
# 쿠버네티스 네트워크는 계층이 없으므로 서비스를 통하면 네임스페이스를 가로질러 통신이 가능하지만, 컨트롤러는 자시느이 네임스페이스 안에서만 관리 대상 파드를 찾는다.


# 네임스페이스를 삭제하면 해당 네임스페이스에 속한 모든 리소스가 함께 삭제된다.
<!-- sleep-uat.yaml : 네임스페이스를 생성하고 사용하는 메니페스트 -->
apiVersion: v1
kind: Namespace
metadata:
  name: kiamol-ch11-uat     # 네임스페이스 정의는 이름만 있으면 된다.
---
apiVersion: apps/v1
kind: Deployment
metadata:                     # 소속 네임스페이스도 객체 메타데이터의 일부다.
  name: sleep                 # 네임스페이스가 이미 정의되어 있어야하며
  namespace: kiamol-ch11-uat  # 정의되어 있지 않다면 리소스 배치가 실패한다.
  labels:
    app: sleep
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
