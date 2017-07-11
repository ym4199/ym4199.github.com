# AWS


local 에서 전송시키기 위해서 다음과 같이 명령한다.

```
scp -i ~/.ssh/<인증키>.pem -r <프로젝트 폴더> ubuntu@<ec2 public domain>:<ec2 instance 에서 전송할 경로>
```

이때 permission deied 발생한다.

따라서 서버(ubuntu) 내에서 권한을 변경해준다.

```
sudo chown -R ubuntu:ubuntu srv
```
> 폴더 srv에 대한 root 로 되어있던 권한을 ubuntu로 변경하였다. 


## Nginx

> preference 내의 plugin 을 통해 Nginx support 를 받아준다. 

이후 .config_secret 내에 nginx.conf 와 ec2.conf 를 생성하여 관리한다.



```
ec2.conf

server {
	listen 80;
	server_name *.compute.amazonaws.com;
	sharset utf-8;
	client_max_body_size 128M;
	
	location / {
		uwsgi_pass 		unix:///tmp/ex2.sock;
		include 			uwsgi_params;
	}
}	
```


```
uwsgi > uwsgi.service


[Unit]
Description=uWSGI service
After=syslog.target

[Service]
ExecStart=/home/ubuntu/.pyenv/versions/deploy_ec2/bin/uwsgi --ini /srv/deploy_ec2/.config_secret/uwsgi/deploy.ini

Restart=always
KillSignal=SIGQUIT
Type=notify
StandardError=syslog
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```

새로 폴더를 server 환경으로 복사하고 pip install uwsgi 

user 를 deploy 로 만들어 주고(sudo adduser deploy)

```
sudo /home/ubuntu/.pyenv/versions/deploy_ec2/bin/uwsgi --ini /srv/deploy_ec2/.config_secret/uwsgi/deploy.ini
``` 
> 이렇게 진행되면 터미널로 빠져나오지 않고 진행되어야 정상적인 작동진행이다. 이때 정상작동을 확인하고자 한다면 logger를 추가해주자.

```
deploy.ini

# http 는 지워준다
#######
logger = file:/tmp/uwsgi.log
```
> tmp/uwsgi.log 내에서 실패내역을 확인해 볼 수 있다.

최종적으로 성공하고 DNS 로 접속하면 NOTFOUND ERROR 를 접하게 된다.

#### uwsgi 설정
##### uwsgi 정상작동확인

```
uwsgi --http :8080 --home <virtualenv경로> --chdir <django 프로젝트 경로> -w <프로젝트명>.wsgi


uwsgi --http :8080 --home ~/.pyenv/verisons/mysite-env --chdir /src/mysite/django_app -w mysite.wsgi
```
> 실행 후 <DNS>:8080 접속하여 요청을 잘 받는지 확인

uwsgi 사이트 파일 생성

```
sudo mkdir /etc/uwsgi
sudo mkdir /etc/uwsgi/sites
sudo vi /etc/uwsgi/sites/mysite.ini

[uwsgi]
chdir = /srv/mysite/django_app # Django application folder
module = mysite.wsgi:application # Django project name.wsgi
home = /home/ubuntu/.pyenv/versions/mysite # VirtualEnv location

uid = nginx
gid = nginx

socket = /tmp/mysite.sock
chmod-socket = 666
chown-socket = nginx:nginx

enable-threads = true
master = true
pidfile = /tmp/mysite.pid
```

uwsgi site 파일로 정상 작동 확인

```
uwsgi --http :8000 -i /etc/uwsgi/sites/mysite.ini
```

sudo 로 root 권한을 실행

```
sudo /home/ubuntu/.pyenv/versions/mysite/bin/wsgi --http :8080 -i /etc/uwsgi/sites/mystie.ini
```

uwsgi 서비스 설정파일

```
sudo vi /etc/systemd/system/uwsgi.service

[Unit]
Description=uWSGI Emperor service
After=syslog.target

[Service]
ExecPre=/bin/sh -c 'mkdir -p /run/uwsgi; chown nginx:nginx /run/uwsgi'
ExecStart=/home/ubuntu/.pyenv/versions/mysite/bin/uwsgi --uid nginx --gid nginx --master --emperor /etc/uwsgi/sites

Restart=always
KillSignal=SIGQUIT
Type=notify
StandardError=syslog
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```

리부팅시 자동으로 실행되도록 설정

```
sudo systemctl enable uwsgi
```


### Nginx 설정

```
sudo apt-get install software-properties-common python-software-properties
sudo add-apt-repository ppa:nginx/stable
sudo apt-get update
sudo apt-get install nginx
nginx -v
```

```
sudo cat vi /etc/nginx/nginx.conf
```

위의 내용을 그대로 복사하여 가져온다.  
(복사해서 비어있는 nginx.conf 파일에 그대로 복사, user는 deploy, server_names_hash_bucket_size 250;)

```
terminal 에서

sudo cp -f /srv/deploy_ec2/.config_secret/nginx/nginx.conf /etc/nginx/nginx.conf
sudo cp -f /srv/deploy_ec2/.config_secret/nginx/ec2.conf /etc/nginx/sites-available
sudo ln -s /etc/nginx/site-available/ec2.conf /etc/nginx/sites-enabled/ec2.conf
sudo ln -sf /etc/nginx/sites-avialable/ec2.conf /etc/nginx/sites-enabled/ec2.conf
sudo cp -f /srv/deploy_ec2/.config_secret/uwsgi/uwsgi.service /etc/systemed/system/uwsgi.service

sudo rm -rf /etc/nginx/sites-enabled/default
```

이제 nginx 와 uwsgi 를 재시작하자.

```
sudo systemctl restart nginx
sudo systemctl restart uwsgi
```

