<!-- 스테이트풀셋에서의 업데이트 -->
# 스테이트풀셋의 업데이트에서는 maxSurge나 maxUnavailable 등 설정은 사용할 수 없다.
# 동시에 업데이트되는 파드 수는 항상 하나다.
# 다만 partition 값을 이용하여 전체 파드 중 업데이트해야하는 파드의 비율은 설정할 수 있다.
# 지정된 비율의 파드에 업데이트되면 롤아웃이 중단한다.

# 유상태 애플리케이션에서 단계별 롤아웃을 수행할 때 유용하다.

# 스테이트풀셋에 partition 설정이 적용된 업데이트를 배치하라 업데이트 결과는 파드 1만 업데이트되며 파드 0은 업데이트 되지 않는다.

<!-- statefulSet/partition -->
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: todo-db
  labels:
    kiamol: ch09
spec:
  replicas: 2
  selector:
    matchLabels:
      app: todo-db
  serviceName: todo-db  
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      partition: 1  # only updates Pod 1
  template:
    metadata:
      labels:
        app: todo-db
...


# 롤아웃이 끝났는데도 서로 다른 정의의 파드가 함께 동작중이다.
# 스테이트풀셋에서 동작하는 데이터 위주의 애플리케이션에는 롤아웃 중 파드를 업데이트할 때마다 실행하는 일련의 검증 작업이 갖추어질때가 많은데, 여기에서도 이 검증 작업을 이용할 수 있다.
# partition 값을 차츰 줄여가며 릴리스의 페이스를 조절하다 업데이트가 안정적임을 확인한 후 partition 값을 0으로 설정해서 전체 스테이트풀셋을 업데이트 하면 된다.

<!-- 실습 4 -->
# 데이터베이스의 주 인스턴스를 업데이트 해보자
# partition 설정값만 제거하면 된다.

