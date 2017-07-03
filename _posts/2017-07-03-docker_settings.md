# Docker

docker 가 뭐요? 컨테이너 기반 오픈소스 가상환경

개발, 테스트 환경 등을 하나로 통일하여 관리할 수 있는 유용한 관리 시스템 (GITHUB와 비슷)

수정된 부분에 대한 내용만을 기록한다.


* 이미지 : 소스코드, 컴파일 실행 파일 등을 하나로 만든 형태

* 컨테이너 : 이미지를 실행한 상태, 격리된 공간에서 프로세스 동작하는 기술

저장소에 올리고 내리는 것은 이미지이다.

컨테이너에 올린 데이터는 컨테이너가 종료한 뒤 모두 삭제된다. 즉, 내부에 데이터를 갖게되면 큰일이 난다.

## Docker Settings

docker 를 다운받자.[docker 다운로드 페이지](https://docs.docker.com/docker-for-mac/install/)

귀여운 고래 이모티콘의 docker 가 생성된다.

terminal 을 통해 docker version 으로 버전 정보를 알 수 있다.

이미지로 저장하지 않는다면 docker 를 실행할 때마다 환경설정을 해줘야 한다. 즉, 우리는 이미지화 시켜서 바로 올릴 수 있도록 만들자.

```
# docker build -t <사용할 이미지 이름> <Dockerfile이 존재하는 경로>
FROM        ubuntu:16.04
MAINTAINER  dev@azelf.com

RUN         apt-get -y update
RUN         apt-get install -y python-pip
RUN         apt-get install -y git vim

## pyenv
RUN         apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils
RUN         curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash
ENV         PATH /root/.pyenv/bin:$PATH

RUN         pyenv install 3.6.1

RUN         apt-get -y install zsh
RUN         wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | zsh || true
RUN         chsh -s /usr/bin/zsh

RUN         echo 'export PATH="/root/.pyenv/bin:$PATH"' >> ~/.zshrc
RUN         echo 'eval "$(pyenv init -)"' >> ~/.zshrc
RUN         echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.zshrc
```

위의 코드로 기본적인 환경설정이 완성될 수 있다.

### docker image 

```
docker build -t <사용할 이미지 이름> <Dockerfile이 존재하는 경로>
```

### docker 실행

```
docker run -rm -it <이미지이름> /bin/bash
```