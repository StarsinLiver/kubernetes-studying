<!-- Todo : 영구볼륨 -->
# 영구볼륨은 사용가능한 스토리지의 조각을 정의한 쿠버네티스의 리소스이다.
# 영구볼륨은 클러스터 관리자가 만드는데, 각각의 영구볼륨에는 이를 구현하는 스토리지 시스템에 대한 볼륨 정의가 들어가있다.
# 예제는 NFS스토리지를 사용하는 영구볼륨의 정의다.

<!-- persistentVolumn.yaml -->
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv01                     # 볼륨 이름
spec:
  capacity:
    storage: 50Mi                # 볼륨 용량
  accessModes:                   # 파드의 접근유형
    - ReadWriteOnce              # 파드 하나에서만 사용가능
  nfs:                           # NFS 스토리지를 사용하는 볼륨
    server: nfs.my.network       # NFS 서버의 도메인 네임
    path: "/kubernetes-volumes"  # 스토리지 경로

<!-- Todo : 영구볼륨클레임 -->
# 이제 클러스터에 우리가 사용할 수있는 영구볼륨이 생겼다. 이 영구볼륨에는 접근 유형과 용량도 지정되어있다.
# 하지만 파드가 영구볼륨을 직접 사용하지 못한다.
# 영구볼륨클레임이란 것을 사용하여 볼륨 사용을 요청해야 한다.
# 영구볼륨클레임은 파드가 사용하는 스토리지의 추상이다. 애플리케이션에서 사용할 스토리지를 요청하는 역할을 한다. 
# 쿠버네티스에서 영구볼륨클레임은 요구 조건이 일치하는 영구볼륨과 함께 쓰인다.
# 다만 상세한 볼륨 정보는 영구볼륨(PV)에 맡긴다.

<!-- persistentVolumnClaim.yaml -->
# 전의 생성된 영구볼륨과 요구 조건이 일치하는 영구볼륨클레임이다.
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc            # 애플리케이션은 영구볼륨클레임을 통해 영구볼륨을 사용한다.
spec:
  accessModes:                  # 접근 유형은 필수 설정이다.
    - ReadWriteOnce
  resources:
    requests:
      storage: 40Mi             # 요청하는 스토리지 용량
  storageClassName: ""          # 스토리지 유형을 지정하지 않음

<!-- 디플로이먼트에서의 영구볼륨클레임 사용 -->
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
            value: /secrets/postgres_password
          volumeMounts:
            - name: secret
              mountPath: "/secrets"
            - name: data
              mountPath: /var/lib/postgresql/data
      volumes:
        - name: secret
          secret:
            secretName: todo-db-secret
            defaultMode: 0400
            items:
            - key: POSTGRES_PASSWORD
              path: postgres_password
        - name: data  
          persistentVolumeClaim:            # 영구볼륨클레임을 볼륨으로 사용
            claimName: postgres-pvc         # 사용할 영구볼륨클레임의 이름

<!-- 동적 볼륨 프로비저닝 -->
# 동적 볼륨 프로비저닝은 영구볼륨클레임만 생성하면 그에 맞는 영구볼륨을 클러스터에서 동적으로 생성해 주는 방식이다.
# 클러스터에는 다양한 요구사항에 대응할 수 있는 여러가지 스토리지 유형을 설정할 수 있는데, 이 유형 중에서 기본 유형을 다시 지정할 수 있다.
# 영구볼륨클레임의 정의에서는 이 스토리지 유형만 지정하면 된다.

<!-- 동적볼륨 프로비저닝 -->
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc-dynamic
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
      # storageClassName 필드가 없으면 기본 유형이 쓰인다.

<!-- 스토리지 유형 -->
# 스토리지 유형은 상당히 유연하다. 스토리지 유형은 표준 쿠버네티스 리소스로 생성되는데 스토리지 유형의 정의에 다음 세가지 필드로 이 슽리지가 어떻게 동작할지 지정한다.

1) provisioner : 영구볼륨이 필요해질 때 영구볼륨을 만드는 주체. 플랫폼에 따라 관리 주체가 달라진다. 예를들어 기본 상태의 AKS는 함께 통합된 애저 파일스가 스토리지를 만든다.

2) reclaimPolicy : 연결되었던 클레임이 삭제되었을때 남아있는 볼륨을 어떻게 처리할지 지정한다. 볼륨을 함께 삭제할 수도 있고, 그대로 남겨둘 수도 있다

3) volumeBindingMode : 영구볼륨클레임이 생성되자마자 바로 영구볼륨을 생성해서 연결할지 아니면 해당 클레임을 사용하는 파드가 생성될 때 영구볼륨을 생성할지 선택가능하다.

<!-- 영구볼륨클레임 스토리지클래스 정의 -->
<!-- Todo : storageClass/persistentVolumeClaim -->
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc-kiamol
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: kiamol    # 스토리지 유형은 스토리지의 추상이다.
  resources:
    requests:
      storage: 100Mi