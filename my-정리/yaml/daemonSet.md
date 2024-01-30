<!-- 데몬셋을 이용한 스케일링으로 고가용성 확보하기 -->
# 데몬셋(DaemonSet)은 리눅스의 백그라운드에서 단일 인스턴스로 동작하며 시스템 관련 기능을 제공하는 프로세스를 가리키는 말인 데몬(Daemon)(윈도우 서비스와 같은의미)에서 따온 이름이다.

# 쿠버네티스의 데몬셋은 클러스터 내 모든 노드 또는 셀렉터와 일치하는 일부 노드에서 단일 레플리카 또는 파드로 동작하는 리소스를 의미한다.
# 데몬셋은 각 노드에서 정보를 수집하여 중아의 수집 모듈에 전달하거나 하는 인프라 수준의 관심사와 관련된 목적으로 많이 쓰인다.
# 각 노드마다 파드가 하나씩 동작하면서 해당 노드의 데이터를 수집하는 식이다.

# 또한 한 노드에 여러 레플리카를 둘 정도로 부하는 없지만 단순히 고가용성을 확보하려는 목적에서 데몬셋을 활용할 수도 있다.

<!--데몬셋의 정의 -->
# 데몬셋의 YAML 정의이며, 다른 컨트롤러 리소스의 정의와 크게 다를 게 없지만 레플리카 수를 설정하지 않아도 된다는 것이 다르다.
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: pi-proxy
  labels:
    kiamol: ch06
spec:
  selector:
    matchLabels:        # 데몬셋에도 레이블 셀렉터가 있다.!!
      app: pi-proxy     # 데몬셋의 관리 대상인 파드를 결정하는 기준
  template:
    metadata:
      labels:
        app: pi-proxy   # 레이블 셀렉터와 레이블이 일치해야함
    spec:
      containers:
        - image: nginx:1.17-alpine
          name: nginx
          ports:
            - containerPort: 80
              name: http
          volumeMounts:
            - name: config
              mountPath: "/etc/nginx/"
              readOnly: true
            - name: cache-volume
              mountPath: /data/nginx/cache
      volumes:
        - name: config
          configMap:
            name: pi-proxy-configmap
        - name: cache-volume
          hostPath:
            path: /volumes/nginx-cache
            type: DirectoryOrCreate

<!-- 데몬셋의 다른 기능 -->
# 데몬셋은 단순히 각 노드마다 파드를 실행할 때만 사용하는 것이 아니다.
# 클러스터의 노드 중 일부만 외부에서 들어오는 트래픽을 받을 수 있다면, 이들 노드에만 프록시 파드를 실행하면 될 것이다.
# 이들 노드에 원하는 레이블을 부여하고 파드 정의에서 이 레이블을 선택하는 셀렉터를 추가하면 외부 트래픽을 받는 노드에만 프록시 파드가 실행된다.

<!-- Todo: pi/proxy/daemonset/nginx-ds-nodeSelector.yaml -->
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: pi-proxy
  labels:
    kiamol: ch06
spec:
  selector:
    matchLabels:
      app: pi-proxy 
  template:
    metadata:
      labels:
        app: pi-proxy
    spec:
      containers:
        - image: nginx:1.17-alpine
          name: nginx
          ports:
            - containerPort: 80
              name: http
          volumeMounts:
            - name: config
              mountPath: "/etc/nginx/"
              readOnly: true
            - name: cache-volume
              mountPath: /data/nginx/cache
      volumes:
        - name: config
          configMap:
            name: pi-proxy-configmap
        - name: cache-volume
          hostPath:
            path: /volumes/nginx-cache
            type: DirectoryOrCreate
      nodeSelector:                     # 특정 노드에서만 파드를 실행한다.          
        kiamol: ch06                    # kiamol=ch96 레이블이 부여된 노드만 대상이다.