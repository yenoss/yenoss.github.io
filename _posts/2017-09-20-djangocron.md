---
layout: post
title: 'django command와 crontab을 함께 써보자.'
tags: [django command, crontab]
---
django,command,crontab

### 1. 왜.

* 서비스에서는 주기적으로 해야하는 작업들이 많다. 매일 매일 갱신되어야하는 데이터 예를 들면 출첵 같으것이 그것이 된다. 


* 주기적 작업을 하가위해선 cron작업이 들어가게된다.


* django 프로젝트를 진행하고 있고 우린 개별적으로 특정 코드를 특정 시간에 실행시키고 싶었다.


<br><br>
### 2. 그래서 무엇인가.

* django에서 shell command(특정코드)를 쓰기위해서 기본적으로 custom django command를 제공해준다.


* custom django command 와 django-crontab 라이브러리를통해 주기적인 작업 처리를 해본다.

<br>
### 3. 써보자.
<br>
#### django command

* django 프로젝트를 만들어 개인 프로젝트내에 아래와같은 트리가 만들어 질 수 있도록 한다.

```
project
│   manage.py    
│
└───{urProject}
    │─── management
			│─── commands   
					 {urCronProgram}.py   
    
```

<br>

* 그림처럼 프로젝트하위에 management폴더를 만들고, commands 폴더를 만들고 아래에 실행할 python파일을 만든다.

* urCronProgram.py는 아래와같이 코딩한다.


```
from django.core.management.base import BaseCommand, CommandError

class Command(BaseCommand):
    def handle(self, *args, **options):
    	print("프린트하는 프로그램");

```

<br>
* handle안에 원하는 프로그램 로직을 짜면 된다.


<br>
##### command 직접 실행.

* 앞서 말했듯 command는 기본 django 프로젝트 환경내에 특정코드를 실행하게해준다.  
* 아래와 같은 명령어로 실행이 가능하다.

```
python3 manage.py urCronProgram

```
<br>

* 만일 당신이 settings 옵션을 통해 런을 하고있다면, --settigns = {settings 위치} 옵션을 추가로 명령 해주어야한다.

<br>
#### crontab
<br>
> 소프트웨어 유틸리티 Cron은 유닉스 계열 컴퓨터 운영 체제의 시간 기반 잡 스케줄러이다. 소프트웨어 환경을 설정하고 관리하는 사람들은 작업을 고정된 시간, 날짜, 간격에 주기적으로 실행할 수 있도록 스케줄링하기 위해 cron을 사용한다. - 위키

<br>

* cron의 위키백과 정의이다. crontab(환경이자 파일)에 의해 cron이 구동된다고한다.

* cron은 unix corntab을 이용하여 해당 명령어를 등록해줄 수있지만, 서비스에 적용하기위해선 서버환경 depdency는 최대한 피하는게 좋기때문에 django에서 관리해주는 django-crontab모듈을 사용한다.

<br>
##### 설치

* 도큐먼트대로 설치해본다.

* pip로 라이브러리를 설치하고
 
```
pip install django-crontab

```

* settings app을 추가해준다.

```
INSTALLED_APPS = (
    'django_crontab',
    ...
)
```


* 다음 CRONJOBS라는 corn configure을 설정해준다. 

```
CRONJOBS = [
('0 0 * * *','django.core.management.call_command',['urCronProgram'],{},'>> '+BASE_DIR+'/log/urCronProgram 2>&1'),
]
```

* 파라미터를 살펴보면,
	1. cron 시점, 언제 해당 프로그램이 run 될것인가. [여기를 보면 한눈에 알수있다.](https://crontab.guru/#*_*_*_*_*)
	2. run될 프로그램의 위치이다. 이 라이브러리는 우리와같이 django command를 쓰지 않는 경우도 실행될 수있도록 되어있기 때문이다. 우리는 django command를 이용해서 작업했음으로 똑같이 작성해준다.
	3. arguments 값이다. 프로그램에 필요한 값을 넣어주게되는데 우린 django command를 사용했음으로 여기에 해당 파일이름을 넣어준다. (정확히 list of positional arg라고 명시되어있다)
	4. arguments 값이다. 3과 마찬가지지만 dic타입이다(dict of keyword arguments)
	5. 마지막은 stdout,err의 저장위치이다. 필자는 baseDir에 log아래에 파일을 만들기로 하였다.

	
* 기본적인 사항이고, 추가적인 부분은 doc를 보면된다.


<br>
##### crontab 명령어
<br>

* 추가

```
python manage.py crontab add
```

<br>

* 삭제

```
python manage.py crontab remove
```

<br>
* 제거


```
python manage.py crontab remove
```

<br><br>

* 꼭 삭제하고 추가 안 해도된다. 기존 돌고있는것에 추가를 하면 기존 옵션이 사라지고 자동 새롭게 추가된다.


* 뭔가 새로운 스케쥴링을 만드는것과 같이 보이지만 사실 unix cron에 지금 설정사항들이 알맞게 설정 되어있다. crontab -l을 쳐보면 알게될 것..

<br><br>
### 4.마치며

* 간단하지만 잘만들었다라는 생각이 들었다. 위에 얘기했듯이 unix 기본기능을 매핑을 잘해두었고, django에 의존성이생겨 더욱 설정이 깔끔해진것 같다.

<br>
### Ref.

1. [django-command](https://docs.djangoproject.com/en/1.11/howto/custom-management-commands/)

2. [django-crontab](https://github.com/kraiz/django-crontab)




