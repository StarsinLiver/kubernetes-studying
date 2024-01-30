<!-- 온딜리트 전략 -->
# 온딜리트 전략은 각 파드의 업데이트 시점을 직접 제어해야 할 때 사용하는 전략이다.
# 업데이트를 배치하면 컨트롤러가 기존 파드를 종료하지 않고 그대로 둔 채 파드를 주시한다.
# 그러다 다른 프로세스가 파드를 삭제하면 새로운 정의를 따른 대체 파드를 생성하는 방식이다.

# 제거되기 전에 가지고 있는 데이터를 모두 디스크에 기록해야 하는 파드를 관리하는 스테이트풀셋을 생각해보자 또한 다음 파드가 사용할 수 있도록 종료 전 전용 하드웨어에서 접속을 해제해야 하는 파드를 관리하는 데몬셋은 어떤가.
# 그리 흔한 경우는 아니지만, 이때도 온딜리트 전략을 사용하면 파드의 제거 시점은 사용자가 직접 통제하되 삭제된 파드를 쿠버네티스가 자동으로 대체하게끔 할 수 있다.

# 데몬셋을 롤링 업데이트 전략으로 업데이트한다.
# 단일 노드 클러스터이므로 파드가 교체될때까지 잠시 서비스를 사용할 수 없다.
<!-- daemonset/update -->
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: todo-proxy
  labels:
    kiamol: ch09
spec:
  selector:
    matchLabels:
      app: todo-proxy
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
  minReadySeconds: 90
...

