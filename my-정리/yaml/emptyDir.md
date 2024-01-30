<!-- Todo : 공디렉터리(emptyDir) -->
# 데이터가 특정 노드에 고정된다는것은 대체 파드가 이전 파드와 동일한 노드에만 배치되도록 해야한다는 의미이다.
# 반대로 데이터를 특정 노드에 고정시키지 않는다면 어떤 파드에도 대체 파드를 배치할 수 있다.
# 쿠버네티스에는 이와 관련된 많은 선택지가 있지만, 우선 우리가 원하는 것은 무엇이고 그중에서 클러스터에서 사용 가능한 것이 무엇인지 파악해야한다.
# 그리고 파드의 정의에 이를 명확히 밝혀야한다

# 선택지 중 가장 간단한 것은 노드의 특정 디렉터리를 가리키는 볼륨이다.
# 컨테이너가 볼륨 마운트 경로에 데이터를 기록하면 이 데이터가 실제로 기록되는 위치는 이 노드의 특정 디렉터리가 된다.
# 캐시 목적으로 공디렉터리 볼륨을 사용한느 애플리케이션을 이용하여 이 방법에 어떤 한계가 있는지 알아본 후 이 볼륨을 노드 수준의 스토리지로 업그레이드 해 보겠다.

<!-- emptyDir.yaml -->
<!--! 공디렉터리는 파드 수준에서 생성된다. -->
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
          volumeMounts:
            - name: data        # 이름이 data인 볼륨을 마운트
              mountPath: /data  # 이 볼륨을 경로 /data 에 마운트
      volumes:
        - name: data            # 볼륨 data의 정의
          emptyDir: {}          # 이 볼륨의 유형은 공디렉터리
