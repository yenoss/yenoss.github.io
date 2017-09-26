---
layout: post
title: 'django logging 처리'
tags: [django,logging]
---
django,logger,logging

<br>
### 1. 왜.


* 서비스에서 발생되는 모든 로그들은 수집되어야한다. 


* 그것들은 err/exception로그 일수도 있고, 단순 작동했다는것을 명확히 보여주는 info 로그 혹은 개발자 필요에의한 로그들이 있을것이다.


* 더나아가 서비스 활용에 필요한 주요 로그일수 도있다.


* 로그는 개발에서 뿐아니라, 운영,기획에서도 아주 중요한 다음 의사 결정 지표가 된다.

<br><br>
### 2. 그래서 무엇인가.
* django에서 제공되는 LoOGGING 세팅을 이용해본다.

<br>
### 3. 써보자.

<br>
##### 설정


* {project}/settings.py 를 아래와 같이 설정해본다.


```
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    // 로그가 찍힐때 formatting이 어떻게 될것인가
    'formatters': {
        'simple': {
            'format': '%(asctime)s %(levelname)s: %(message)s'
        },
    },

    'handlers': {
        'console': {
            'level': 'DEBUG',
            'class': 'logging.StreamHandler',
            'formatter': 'simple'

        },
        'logfile': {
            'level':'INFO',
            'class':'logging.FileHandler',
            'filename': "log/service.log",
            'formatter': 'simple'


        },
    },
    'root': {
        'level': 'INFO',
        'handlers': ['console', 'logfile']
    },
}

```
<br>

* version :	현재 포맷 버전을 말한다.


* disable_existing_loggers : 아래 로그들을 활성화할지에대한 이야기다. default가 true이다.


* handler : 로그핸들러이다. 콘솔에 찍힐경우, 로그파일로 저장할경우 어떻게 할건지 설정하게된다


* level : 찍힐 로그의 레벨을 의미한다. 파이선 로깅에있는 레벨들로 CRITICAL/ERROR/WARNING/INFO/DEBUG/NOTSET 이 있다.


* root: 최종적으로 모든 핸들러를 관리한다. 여기서 가지는 level은 어떤 level들보다 최우선으로 적용된다. root내부에 handlers들이 다른 levels를 가진다고해도 root의 level의 위로 넘어갈수없다. (handlers안이 모두 debug 여도 root가 info면 info밖에 찍히질않는다)

<br><br>
### 4.마치며
* 로그는 자알~ 찍어둬야 한다.
* 가끔 다른 wsgi를 돌릴때 기존 로그들이 안먹는 경우가 있어서 자알~ 유심히 봐둬야 한다.


### REF.
1. [djangoLogging](https://docs.djangoproject.com/en/1.11/topics/logging/#custom-logging-configuration)

2. [loggingLevels](https://docs.python.org/2/library/logging.html)






