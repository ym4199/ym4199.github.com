# AWS 

> 다음 내용들은 누구에게 정보전달의 목적을 하지 않는다. 시간이 경과함에 따라 아래 과정들이 기억속의 찌꺼기가 됐을 때를 위해 기록한다.

AWS 의 계정을 생성하기 위해 해외결제 가능한 카드가 필요하고(VISA, MASTER) 계정을 생성하면 1년간 기본적인 기능을 제공한다.

![free_ubuntu](https://{{site.url}}/image/free_ubuntu.png)

지역 설정을 **seoul** 로 설정해야 반응이(?) 빠르다고 한다. 

![location](https://{{site.url}}/image/location.png)

aws service 창에서 ec2 검색 virtual servers in the cloud

![ec2](https://{{site.url}}/image/EC2.png)

ec2 의 create free version 선택 후 다른 옵션의 변경 없이 바로 만들자!  
그렇다면 이제 중요한 key pair 을 설정하는 작업이 남았다.

외부의 접근을 허용하지 않고 ssh 를 생성하여 나만 가지고 있어야 한다.

하지만 아직 키페어가 없기 때문에 새로 생성해야한다. 

**만들었을 때 단 한번만 확인할 수 있으니 주의하자.**

download 했다면 **파일명.pem** 으로 확인가능할 것이다.

이제 해당 파일을 우리가 관리하는 .ssh 속에 해당 파일명.pem 에 넣어서 관리하자

```
mv 파일명.pem ~/.ssh
```
![ssh](https://{{site.url}}/image/ssh.png)

해당 파일이 제대로 .ssh 안에 들어가있음을 확인 할 수 있다.

이제 접근을 해보자. 접근하는 방법은 간단하다 EC2 의 dashboard 상에서 instance 내에서 하단의 public DNS 가 주소이다. 해당 주소로 접근을 하면 된다.

![instance_id](https://{{site.url}}/image/instance_id.png)

이때 방법은 aws ec2 ssh 로 googling 해도 알수가 있다. [google 결과 보기](http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)  
다음 명령을 통해 인스턴스에 연결한다.

```
ssh -i /path/my-key-pair.pem ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com
```
아마 접근할 수 없을 것이다.  
이때 접근할 수 없는 경우는 해당 파일에 대한 권한 설정을 변경해줘야 하는데

```
chmod 400 /path/my-key-pair.pem
```

이렇게 키페어의 권한을 변경하고 다시 

```
ssh -i /path/my-key-pair.pem ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com
```
위의 값을 반복적으로 사용할 것이기 때문에 복사해두는 것이 좋다.

```
vim .zshrc

alias 명령어="ssh -i /path/my-key-pair.pem ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com"
```

이제 ubuntu 을 열때 명령어를 통해 쉽게 접근할 수 있게 됐다.


## Ubuntu 환경설정

ubuntu 가상환경으로 접속했음을 확인할 수 있다.

이제 ubuntu 환경에서 사용할 패키지들을 설치해주자.

python 설치

```
sudo apt-get install python-pip
```

zsh 설치
```
sudo apt-get install zsh
```

oh-my-zsh 설치
```
sudo curl -L http://install.ohmyz.sh|sh
```

zsh 을 설치했다면 기본 shell 로 설정해주자.

```
sudo chsh ubuntu -s /usr/bin/zsh
```

shell 의 변경값은 터미널을 종료하고 다시 들어와야 확인가능하다.   
zsh 로 바뀐것을 눈으로 확인하고 싶다면 
```
echo $SHELL
```

위의 명령으로 지금 기본 사용하는 shell 의 정보를 얻을 수 있다.

개발자가 설정해놓은 개발환경을 맞춰줘야 원활하게 진행할 수 있다. 즉, 가상환경의 조건을 맞춰줘야한다.

```
sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils
```

가상환경을 관리해주는 pyenv까지 설치해주자

```
curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash
```
pyenv 설정을 사용하기 위해 .zshrc에 해당 내용을 기록하자

```
vi ~./zshrc	로 zshrc를 열고

export PATH="/home/ubuntu/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

pillow 라이브러리는 사용자의 선택사항이다. 해당 라이브러리는 이미지 필드를 사용할 때 필요사항이다. 프로젝트를 진행하면서 이미지 필드를 사용할지 모르겠으나 일단 설치해해본다.

```
sudo apt-get install libtiff5-dev libjpeg8-dev zlib1g-dev \
    libfreetype6-dev liblcms2-dev libwebp-dev tcl8.6-dev tk8.6-dev python-tk
```


## Upload 파일

scp 를 사용하여 파일을 전송할 것이다.

```
scp -i /path/my-key-pair.pem /path/SampleFile.txt ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com:<파일위치>
```

파일위치로 적당한 위치는 ubuntu 의 루트에서 srv 라는 폴더이다.

해당 폴더의 권한을 아래의 명령을 통해 변경하자.

```
sudo chown -R ubuntu:ubuntu /srv/
```

srv 에 로컬에서 전달하면 굳이 해당 폴더가 없더라도 생성되어 전송된다.

```
scp -i <인증서위치> -r <프로젝트폴더> ubuntu@<인스턴스 Public DNS>:/srv/폴더명
```


## Django 설정

가상환경 설정한 위치에서 설치해주자

```
pyenv install 버전
pyenv virtualenv 버전 가상환경이름
pyenv local 가상환경이름
```
위의 install 과정은 상당히 오래 걸린다. 설치하는 동안 쉬자.
동일한 가상환경의 밑받침이 준비가 되었다.

```
pip install -r requirements.txt
```

친절한 개발자가 기본 환경에 대한 정보를 requirements.txt 에 넣어서 제공할 것이다. 이를 이용하여 쉽게 개발 환경과 동일한 환경을 구현해주자. 이처럼 친절한 개발자가 되어야 한다.

이제 서버를 구동시킬 전반적인 과정이 마무리되었다. 확인을 해보자.

```
./manage.py runserver --settings=config.settings.debug
```

아마 실패했을 것이다. 현재 python 의 버전을 살펴보면 2.x 버전이 사용되고 있을 것이다.  
exit 으로 외부로 나온 뒤 다시 들어가서 python의 버전을 확인해보자. 만약 아직도 버전정보가 업데이트 안됐다면 새로 python 을 설치해보자.

외부에서 접근이 가능하도록 만들기 위해 0:8000 을 붙여주자.

```
./manage.py runserver --settings=config.settings.debug 0:8000
```
서버는 정상적으로 돌고있음을 터미널에서 확인하고 브라우저를 통해 해당 url 로 접속해보자.  

접속에 실패했을 것이다. 이유는???  
방화벽으로 기본적으로 막혀있다.

security group으로 설정되어 있는 lauch-wizard-1 을 대신할 새로운 group을 생성하자.

이때 Description 은 한글로 작성하면 생성이 불가능 하다.  
outbound 는 두고 inbound 를 설정하는데 

type|SSH
---|---
protocol|TCP
Port Range|22
Source|My IP

하나더 Rule 을 만들어서

type|Custom TCPI
---|---
protocol|TCP
Port Range|8000
Source|Custom


이제 필요가 없어진 launch-wizard-1 에 연결됐던 것을 바꿔주자.  
Instance에서 Action 의 change 에서 새로 넣고자 하는 Group ID 를 넣고 wizard를 취소해주자.  
이제 wizard 를 지울 수 있게 되었다.

# BooooooM!!!

연결이 됐음을 확인할 수 있다.

```
./manage.py runserver --settings=config.settings.debug 0:8000
```

![check](https://{{site.url}}/image/check.png)