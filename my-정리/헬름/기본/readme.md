<!-- web-ping-deployment.yaml -->
apiVersion: apps/v1
kind: Deployment                          # 일바적인 쿠버네티스 YAML 스크립트다.
metadata:
  name: {{ .Release.Name }}               # 릴리스 이름이 들어갈 템플릿 변수
  labels:
    kiamol: {{ .Values.kiamolChapter }}   # kiamolChapter 값이 들어갈 템플릿 변수
spec:
  selector:
    matchLabels:
      app: web-ping
      instance: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: web-ping
        instance: {{ .Release.Name }}
    spec:
      containers:
        - name: app
          image: kiamol/ch10-web-ping
          env:
            - name: TARGET
              value: {{ .Values.targetUrl }}
            - name: METHOD
              value: {{ .Values.httpMethod }}
            - name: INTERVAL
              value: {{ .Values.pingIntervalMilliseconds | quote }}

<!-- helm create 명령어 -->
# 헬름 차트의 파일 구조는 매우 상세하게 지정되어 있는데, 이 때문에 새로운 차트의 밑바탕 구조를 생성하는 helm create 명령이 제공된다.
# 최상위 디렉터리 이름이 차트 이름이 되며 이 디렉터리에는 최소한 다음 세가지 요소가 있어야한다. (필수 요소)

1) chart.yaml : 차트이름이나 버전 등 메타데이터를 기록한 파일
2) values.yaml : 파라미터 값의 기본값을 기록한 파일 (== 변수 지정 파일)
3) templates 디렉터리 : 템플릿 변수가 포함된 쿠버네티스의 매니페스트 파일을 담은 디렉터리

# 예제에 사용된 quote 함수는 삽입된 값에 앞뒤로 따옴표가 없을 때 따옴표를 붙여 주는 역할을 한다.
# 또한 템플릿에서 반복 또는 분기 로직으로 문자열이나 숫자를 계산할 수도 있고, 쿠버네티스 API를 통해 다른 쿠버네티스 객체의 정보를 참조 할 수도있다.