---
layout: post
title: '[ubuntu] NOPASSWD 옵션이 안 먹힐 때'
tags: [ubuntu]
background: '/img/posts/bg_linux.jpg'
---
ubuntu no password not working. nopassword 안먹힘.

<br>
### 1. 무엇인가.
* 특정 계정에게 sudo 명령을 칠때 password를 주지 않고 싶었다. 배포관련 권한을 특정 하기 위해서이다.


<br>
### 2. err 상황.
*  일반적인 방법으로 

{% highlight swift %}

sudo visudo

{% endhighlight %}
에 들어가서,

{% highlight swift %}

member1 ALL=(ALL) NOPASSWD:ALL

{% endhighlight  %}

member1에게 권한을 주었다.
하지만 member1은 sudo 명령을칠때마다 password를 원해다.


<br>
### 3. 해결.
visduo에서 여러 권한 설정을 해주게되는데 개인과 그룹으로 권한을 주게된다. 

이중 개인이 그룹에 속해 중첩되는 경우가 발생할 수있는데 이때 권한은 최종 라인에 있는 것을 기준으로 가지게된다.

무슨말이냐면

{% highlight swift %}

member1 ALL=(ALL) NOPASSWD:ALL
group1 ALL=(ALL:ALL) ALL

{% endhighlight  %}


sudo visudo에 위와 같이 세팅되어있고, member1이 group1에 포함되어있다면, 
최종 라인인 group1에 권한이 member1에게 속하게 된다. 

<br><br>
### 4. 마치며.
* 정식문서를 잘 `꼼꼼히` 읽자. 

<br>
### REF.
1. [sudoer man page](https://www.sudo.ws/man/1.8.15/sudoers.man.html)

2. [ask ubuntu](https://askubuntu.com/questions/100051/why-is-sudoers-nopasswd-option-not-working)

