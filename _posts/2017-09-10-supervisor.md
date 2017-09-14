---
layout: post
title: 'ubuntu Supervisor를 설치해보자.'
tags: [ubuntu,python]
---
process monitoring,supervisor,forever

### 1. 왜.
* 서비스를 운영하다보면 나의 서비스(프로세스)가 잘 돌고있는지 모니터링을 해야하고, 해당 프로세스가 죽었을 경우 다시 살리는 로직등을 오토매틱하게 구현해 두어야한다. 
손이 가는 작업이지만 꼭 필요한 중요한 작업이다.

* nodejs에서 사용하는 forever과 비슷한 역할을 한다.

<br>
### 2. 그래서 무엇인가.
* superviosr는 서비스를 모니터링 하며, 손쉽게 서비스를 온오프 한다. 기본적으로 서버가 죽을경우 재실행시켜 장애를 최소화 한다.

<br>
### 3. 써보자.

##### 설치

```
apt-get install supervisor
```

##### 설정
* supervisor 세팅을해보자.

* /etc/supervisor/conf.d 안에 {service_name} 으로 파일을 만들고 아래와 같이 서비스에 대한 세팅을 한다.

```
[program:{service_name}]
//실행할 서비스 스크립트
command =  /home/ubuntu/{project}/{실행할 shell script}
//루트디렉토리
directory = /home/ubuntu/{project}
//실행할유저
user = ubuntu
//로그저장할곳
stdout_logfile = /home/ubuntu/{project}/{log저장할 파일이름}
//스크립트내의 에러 출력 여하이다. false로 할시 log에 로그가쌓이지않는다.
redirect_stderr = true
//프로세스를 그룹으로 관리할지 여부. false라면 supervisor를 껐다켰을떄 
stopasgroup=true
```


#### 명령어
* supervisord 실행

sudo supervisord

* 실행

```
sudo supervisorctl start all 혹은 start {program_name}...
```

* 정지

```
sudo supervisorctl stop all 혹은 start {program_name}...
```


* 현재 상태

```
sudo supervisorctl status
```


* 자세한건? [supervisor docs](http://supervisord.org/running.html)

##### 웹으로보자

* 명령어를 통해 관리할 수있지만 웹 또한 제공해준다. 관리 웹을 띄어보자


* /etc/supervisor/supervisord.conf 에 들어가서 마지막줄에  아래와 같이 세팅해준다.

```
[inet_http_server]
port=0.0.0.0:9001
username={너의 id}
password={너의 password}
```

* 자, 이제 해당포트로 들어가 id/pw를 검색하면 아주 깔끔한 페이지가 나올것이다.

### 4.마치며

* 사실 서비스가 어떠한 애러로 인해 죽는 일은 발생하면 안 된다.
* 하지만 대비 & 모니터링이라는 측면에서 superviosr는 서비스를 띄울때 필수적인 도구라고 생각한다. 
* 부분적인 장애로 인해 다른 api를 제공할 수없다면 더욱 큰 서비스 장애로 일어날것이기 떄문이다.







