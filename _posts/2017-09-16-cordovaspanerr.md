---
layout: post
title: '[err] cordova Error: spawn EACCESS'
tags: [err,cordova]
background: '/img/posts/bg_web.jpg'
---
err,cordova,spawn eaccess

<br>

### 1. 무엇인가.

* permission이 주어지지 않아서 생기는 문제였다.

<br>
<br>
### 2. err 상황.

* cordova에서 android ploatform을 이미 add한 상태.


* 특정 플러그인을 설치하려고 시도.

{% highlight swift %}

Failed to install 'cordova-plugin-device': Error: spawn EACCES

{% endhighlight %}

<br>
### 3. 해결.
* anroid 폴더의 권한을 아래와 같이 open하였더니 해결.
* 특정 폴더와 연관되어있는것이라고 많은 stackoverflow에서 나왔지만, 찾진 못했다.. 

{% highlight swift %}

sudo chmod -R  777 platforms/

{% endhighlight %}

<br>
<br>
### 4. 마치며
* 이슈중 100에 30은 권한 문제였던 듯 싶다..
