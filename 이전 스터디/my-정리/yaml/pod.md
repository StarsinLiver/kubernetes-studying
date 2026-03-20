<!-- pod.yaml -->
# 매니페스트 스크립트는 쿠버네티스 API의 버전과
# 정의하려는 리소스의 유형을 밝히며 시작한다.
apiVersion: v1
kind: Pod

# 리소스의 메타데이터에는 이름(필수요소) 과
# 레이블(비필수요소) 가 있다.
metadata:
  name: hello-kiamol-3

# 스펙은 리소스의 실제 정의 내용이다.
# 파드의 경우 실행할 컨테이너를 정의해야한다.
# 컨테이너는 이름과 이미지로 정의된다.
spec:
  containers:
    - name: web
      image: kiamol/ch02-hello-kiamol