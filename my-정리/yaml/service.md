<!-- 서비스 Service 정의 -->
# 위의 내용을 해결하기 위해 쿠버네티스 클러스터에는 전용 DNS(도메인 네임 서비스) 서버가 있다.
# 따라서 ip주소가 바뀌어도 도메인 네임 조회를 사용하여 서비스이름과 IP주소를 대응시켜준다.

<!-- service.yaml -->
apiVersion: v1    # 서비스는 코어 v1 API를 사용한다.
kind: Service
metadata:
  name: sleep-2   # 서비스 이름이 도메인 네임으로 사용된다.

# 서비스 정의에는 셀렉터와 포트의 목록이 포함되어야 한다.
spec:
  selector:
    app: sleep-2  # app 레이블의 값이 sleep-2인 모든 파드가 대상이다.
  ports:
    - port: 80    # 80번 포트를 주시하다가 파드의 80번 포트로 트래픽을 전달한다.

<!-- 로드밸런스 -->
apiVersion: v1
kind: Service
metadata:
  name: numbers-web
spec:
  ports:
    - port: 8080      # 서비스가 주시하는 포트
      targetPort: 80  # 트래픽이 전달될 파드의 포트
  selector:
    app: numbers-web
  type: LoadBalancer  # 외부 트래픽도 전달할 수 있는 서비스

  <!-- 노드포트(NODE-PORT) --> 
# 외부에서 클러스터로 들어오는 트래픽을 파드로 전달하는 역할을 하는 서비스 리소스의 유형이 한가지 더 있다. 이를 노드포트(NODE-PORT)라고 한다.

# 노드포트 서비스는 외부 로드밸런서가 필요없다.
# 클러스터를 구성하는 모든 노드가 이 서비스에 지정된 포트를 주시하며 들어온 트래픽을 대상 파드의 대상 포트로 전달한다.
# 노드포트 서비스는 서비스에서 설정된 포트가 모든 노드에서 개방되어 있어야 하기 때문에 로드 밸런서 서비스만큰 유연하지는 않다.
# 때문에 K3s나 도커 데스크톱에서는 잘 동작하지만, kind에서는 그렇지 못하다

apiVersion: v1
kind: Service
metadata:
  name: numbers-web-node
spec:
  ports:
    - port: 8080      # 다른 파드가 서비스에 접근하기 위해 사용하는 포트
      targetPort: 80  # 대상 파드에 트래픽을 전달하는 포트
      nodePort: 30080 # 서비스가 외부에 공개되는 포트
  selector:
    app: numbers-web
  type: NodePort      # 노드의 IP주소를 통해 접근 가능한 서비스 정의

<!-- 익스터널네임 서비스 -->
# 첫번째 선택지 : 익스터널네임(ExternalName) 서비스
# == 도메인 네임에 대한 별명
# 익스터널네임 서비스는 애플리케이션 파드에서 로컬 네임을 사용하고, 쿠버네티스 DNS서버에 이 로컬 네임을 조회하면 외부 도메인으로 해소해 주는 방식이다.

# 익스터널네임 서비스는 도메인 네임의 별명을 만들고
# 파드가 로컬 클러스터 네임 db-service 를 사용하면, 쿠버네티스 DNS서버에서 이 도메인 네임을 외부 도메인 app.database.io로 해소합니다.
# 따라서 파드는 클러스터 외부의 컴포넌트와 통신하지만, 이를 알지 못한다. 파드에서 사용하는 도메인 네임이 로컬 도메인 네임이기 때문이다.
apiVersion: v1
kind: Service
metadata:
  name: numbers-api   # 클러스터 안에서 쓰이는 로컬 도메인 네임
spec:
  type: ExternalName
  externalName: raw.githubusercontent.com # 로컬 도메인네임을 해소할 외부 도메인


  <!-- HTTP 서비스 의 문제점 , 헤드리스 서비스 -->
# TCP 라면 문제없지만 , HTTP 서비스라면 이야기가 달라진다.
# HTTP 요청의 헤더에는 대상 호스트명이 들어간다.
# 그리고 이 헤더의 호스트명이 익스터널네임 서비스의 응답과 다르다면 HTTP 요청이 실패한다.
# 클러스터 안에서만 유효한 로컬 도메인 네임을 외부 시스템으로 연결할 수 있는 방법이 한가지 더 있다,
# http헤더 문제를 해결하지는 못하지만 익스터널네임 서비스와 비슷하게 도메인 네임 대신 IP주소를 대체해주는 방식이다.
# 이런 서비스를 헤드리스 서비스라고 한다.
# 헤드리스 서비스는 클러스터IP 형태로 정의되지만 레이블 셀렉터가 없어 대상 파드가 없다.
# 그 대신에 해드리스 서비스는 자신이 제공해야할 IP주소 목록이 담긴 엔드포인트(endPoint) 리소스와 함께 배포된다.

<!-- 헤드리스 -->
apiVersion: v1
kind: Service
metadata:
  name: numbers-api
spec: 
  type: ClusterIP   # selector 필드가 없으므로 헤드리스 서비스가 됨
  ports:
    - port: 80
---
kind: Endpoints   # 한 파일에 두번째 리소스 정의
apiVersion: v1
metadata:
  name: numbers-api
subsets:
  - addresses:                # 정적 IP주소목록
      - ip: 192.168.123.234
    ports:
      - port: 80    # 그리고 각 IP주소에서 주시할 포트


# 스테이트풀셋의 정의에서는 서비스를 지정하여 각 파드의 도메인 네임을 정의했는데, 이를 위하여 서비스를 헤드리스 서비스로 만들어야 한다.

<!-- 헤드리스 -->
apiVersion: v1
kind: Service
metadata:
  name: todo-db   # 파드 셀렉터가 스테이트풀셋과 일치
  labels:
    kiamol: ch08
spec:
  ports:
    - port: 5432
      targetPort: 5432
      name: postgres
  selector:
    app: todo-db  
  clusterIP: None     # 이 서비스에는 IP 주소가 부여되지 않는다.

# 클러스터IP가 없는 서비스도 클러스터 DNS 서버에 도메인네임이 등록된다.
# 하지만 이 서비스는 고정된 IP 주소를 갖지 않으므로 네트워크 계층에서 실제 주소와 연결될 가상 IP 주소가 없는 대신에 스테이트풀셋 안에 있는 각 파드의 IP주소가 반환된다. 그리고 각각의 파드 역시 도메인 네임이 DNS 서버에 등록된다.