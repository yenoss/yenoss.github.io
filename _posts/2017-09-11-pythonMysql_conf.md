---
layout: post
title: '[err] python mysql_config not found'
tags: [ubuntu,err,python]
background: '/img/posts/bg_linux.jpg'
---
mysql_config not found

<br>
### 1. 무엇인가.
* python에서 mysql을 사용하기위한 client.


<br>
### 2. err 상황.
* pip3 install 로 설치하는 과정에서 아래와 같은 에러가 발생했다.

{% highlight swift %}

OSError: mysql_config not found

{% endhighlight %}

<br>
### 3. 해결.
* 아래와 같이 두가지 패키지를 설치해준다. 
* mysql 개발 해더들과 쓰이는 라이브러리를 먼저 설치해줘야 했던것이다.

{% highlight swift %}

sudo apt-get install python-dev libmysqlclient-dev # Debian / Ubuntu

sudo yum install python-devel mysql-devel # Red Hat / CentOS

{% endhighlight %}


* 파이썬3를 사용한다면, python3-dev를 추가 설치해줘야한다

{% highlight swift %}

sudo apt-get install python3-dev # debian / Ubuntu

sudo yum install python3-devel # Red Hat / CentOS

brew install mysql-connector-c # macOS (Homebrew)

{% endhighlight  %}


<br><br>
### 4. 마치며.
* 아직도 영어로 되어있는 것을 보면(깃,공식 문서..) 자세히 이해하려 하지 않고 코드만 보려고 하는 듯하다. 한글 문서 읽듯 `꼼꼼히` 읽자.
* README가 괜히 있는게 아니다. 
* 뭐든 정해진 가이드만 따르면 잘 되는게 대부분이다.


<br>
### REF.
1. [pymysql github](https://github.com/PyMySQL/mysqlclient-python)


