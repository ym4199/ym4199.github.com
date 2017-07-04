# docker settings

docker files 를 따로 관리할 수 있는 폴더를 생성하고   
docker 관련 파일을 몰아 넣는다.  
docker file 에는 기본 settings 의 설치항목과 가상환경 설정이 들어가야 한다.  
이때, 가상환경 설정시 직접 경로를 지정해야 한다.  

```
RUN         pyenv virtualenv 3.5.3 deploy_ec_docker

RUN         /root/.pyenv/versions/deploy_ec_docker/bni/pip install uwsgi
```

nginx 까지 설치해주자.

이를 통해 image 파일을 생성할 수 있다.

```
docker build -t <사용할 이름> <프로젝트 경로> -f <Dockerfile 이 존재하는 경로>

docker build -t ec_ubuntu . -f .dockerfiles/Dockerfile.ubuntu
```
이후 docker run 을 시도해 보는데 docker container 는 기본적으로 외부와 단절되어 있다. 따라서 포트를 지정해준다.

```
docker run --rm -it -p  <port1>:<port2> /bin/zsh
```
> 우리 컴퓨터의 8080포트로 접속하면 컨터이너의 포트 4040으로 연결


supervisor 폴더 안에 uwsgi.conf 를 생성해주고 다음과 같이 입력하자.
```
command = /root/.pyenv/verions/deploy_ec_docker/bin/uwsgi --ini /srv/deploy_ec_docker/.config/uwsgi/uwsgi-app.ini
```

uwsgi 의 폴더 안에 uwsgi-app.ini 파일을 생성한다.

```
[uwsgi]

http= :8000
chdir = /srv/deploy_ec_docker/django
home = /root/.pyenv/versions/deploy_ec_docker
module= config.wsgi.debug
```

이후 다시 image 생성해주면 된다.

이제 nginx 만 남았다.  
nginx의 경우 daemon 에서 작업이 돌고 있는데 supervisor 로 관리하기 위해서 daemon 으로 빠지면 안된다. 이를 방지하기 위해서 다음과 같이 명령한다.
> supervisor 로 관리하고자 하는 프로그램들은 모두 foreground 에서 진행되어야 한다. 

```
daemon off;
```


uwsgi 실행하기 위해 --ini 주소 입력

```
root/.pyenv/versions/deploy_eb_docker/bin/uwsgi --ini /srv/deploy_eb_docker/.config/uwsgi/uwsgi-app.ini
```

### docker tip

```
docker rm $(docker ps -a -q)

docker ps -a

docker rmi $(docker images | grep "<none>" | awk "{print $3}")
```
> docker container all stop and delete none tage file



docker shell 두개 사용

```
docker exec -it <container이름> /bin/zsh
```