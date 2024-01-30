# ubuntu 1.18.20 테스트 완료 - Rancher 1.8.1 버전 설치 - 쿠베네티스 버전 1.18.20선택 설치
1) Rancher 저장소 서명 추가
$ curl -s https://download.opensuse.org/repositories/isv:/Rancher:/stable/deb/Release.key | gpg --dearmor | sudo dd status=none of=/usr/share/keyrings/isv-rancher-stable-archive-keyring.gpg

% # 서명 만료시
% $ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 1F833FA1A6

2) Rancher 저장소 추가
추가된 저장소 서명을 사용하는 Rancher의 패키지 저장소를 추가
$ echo 'deb [signed-by=/usr/share/keyrings/isv-rancher-stable-archive-keyring.gpg] https://download.opensuse.org/repositories/isv:/Rancher:/stable/deb/ ./' | sudo dd status=none of=/etc/apt/sources.list.d/isv-rancher-stable.list

3) 저장
$ sudo apt update

4) Rancher Desktop 패키지 설치
 $ sudo apt install rancher-desktop

# 도커 설치하기
$ curl -fsSL https://get.docker.com | sh

# k3s 설치하기
curl -sfL https://get.k3s.io | sh -s - --docker - disable=traefik --write-kubeconfig-mode=644

% # 실습 환경을 가상머신에 꾸리고 싶고 베이그런트로 가상머신을 관리해보았다면
# kiamol 저장소의 최상위 디렉터리에서 출발
cd ch01/vagrant-k3s

# 가상 머신 관리 시작
vagrant up

# 가상 머신에 접속
vagrant ssh