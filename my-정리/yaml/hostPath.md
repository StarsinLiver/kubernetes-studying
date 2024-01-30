<!-- Todo : 호스트경로(HostPath) -->
# 공디렉터리 볼륨보다 더 오래 유지되는 볼륨은 노드의 디스크를 가리키는 볼륨이다.
# 이런 볼륨을 호스트경로(HostPath) 볼륨이라고 한다.
# 호스트 경로 역시 파드에 정의되며 컨테이너 파일 시스템에 마운트되는 형태로 쓰인다
# 컨테이너가 마운트 경로 디렉터리에 데이터를 기록하면, 실제 데이터는 노드의 디스크에 기록된다.
# 호스트경로 볼륨은 파드가 교체되어도 데이터를 유지하지만, 파드가 같은 노드에 배치되었을 때만 데이터를 유지한다.

# 예제는 프록시 컨테이너가 /data/nginx/cache 디렉터리에 캐시 파일을 기록할 때 마다 데이터가 실제 기록되는 곳은 노드의 파일 시스템중 /volumes/nginx/cache 디렉터리이다.

<!-- hostPath.yaml -->
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pi-proxy
  labels:
    app: pi-proxy
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
              mountPath: "/etc/nginx/"      # nginx 설정 파일 설정
              readOnly: true
            - name: cache-volume            
              mountPath: /data/nginx/cache  # 프록시의 캐시 저장 경로
      volumes:
        - name: config
          configMap:
            name: pi-proxy-configmap
        - name: cache-volume
          hostPath:                         # 노드의 디렉터리를 사용함
            path: /volumes/nginx/cache      # 사용할 노드의 디렉터리
            type: DirectoryOrCreate         # 디렉터리가 없으면 생성할것
            ###
            type: Directory   # 경로에 디렉터리가 존재해야한다.


<!-- sleep/sleep-with-hostPath-subPath.yaml -->
# 노드의 파일 시스템을 최소한으로 노출하는 호스트경로 볼륨 정의
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
            - name: node-root             # 마운트할 볼륨 이름
              mountPath: /pod-logs        # 마운트 대상 컨테이너 경로
              subPath: var/log/pods       # 마운트 대상 볼륨 내 경로
            - name: node-root             # 두개 생성
              mountPath: /container-logs
              subPath: var/log/containers
      volumes:
        - name: node-root
          hostPath:
            path: /
            type: Directory               # 디렉토리 없으면 생성