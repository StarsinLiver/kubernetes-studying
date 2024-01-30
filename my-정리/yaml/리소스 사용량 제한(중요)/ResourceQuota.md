<!-- 네임스페이스 단위로 총 사용량을 지정하는 방식 -->
# 네임스페이스 단위로 총 사용량을 지정하는 방식으로도 리소스 사용량을 제한할 수 있다.
# 이 방법은 같은 클러스터를 여러 팀이나 환경이 공유할 때 특히 유용하다.

<!-- 02-memory-quota.yaml : 네임스페이스의 총 메모리 사용량 제한 -->
apiVersion: v1
kind: ResourceQuota               # 리소스쿼터 객체
metadata:                         # 지정된 네임스페이스에 적용된다.
  name: memory-quota
  namespace: kiamol-ch12-memory
spec:
  hard:                           # CPU 사용량 또는 메모리 사용량을 제한
    limits.memory: 150Mi          # 150MB 제한

# 컨테이너의 리소스 사용량 제한은 사후 적용 방식이어서 사용량을 초과한 파드가 재시작되는 형태로 적용되지만, 리소스쿼터 객체에 적용된 사용량 제한은 사전 적용 방식이어서 네임스페이스에 할당된 리소스 사용량을 초과하면 더 이상 파드가 생성되지 않는다.
# 리소스쿼터 객체가 있는 네임스페이스에서는 현재 리소스의 잔량과 파드를 생성하면서 사용하게 될 리소스양을 비교할 수 있도록 모든 파드의 정의에 resource 항목이 포함되어야한다.

<!-- cpu-quota.yaml : 네임의스페이스의 총 CPU 사용량 제한 -->
apiVersion: v1
kind: ResourceQuota
metadata:
  name: cpu-quota
  namespace: kiamol-ch12-cpu
spec:
  hard:
    limits.cpu: 500m            # 500밀리코어 === 0.5코어