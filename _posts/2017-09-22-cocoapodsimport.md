---
layout: post
title: '[err] cocoapod import가 안되는 경우'
tags: [xcode,cocoapods]
background: '/img/posts/bg_swift.jpg'
---
xcode,cocoapods

<br>

### 1. 무엇인가.

* searchHeader 설정 이슈.

<br>
<br>
### 2. err 상황.

* cocoapods의 라이브러리가 import되지 않는 문제가 발생하였다.

<br>
### 3. 해결.
* 프로젝트설정 - buildSettings - HeaderSearchPaths 를 보면 여러 리스트가 나올텐데 inherts 바로 아래에 ${PODS_ROTOS}를 추가해주면된다.


{% highlight swift %}

${inherits}
${PODS_ROOTS}

{% endhighlight %}

* 단순 pods의 루트를 잡아주지 않아서 못찾는 것이었다.
* 해결과 별개로 inherits는 무엇일까.
	* inherits는 말그대로 상위 결과를 그대로 가져오는것을 의미하는데 buildsettings에 search옆 level을 클릭해보면 다양한 빌드 같은 옵션에 세팅이 여러개 나오게되는데 이중 왼쪽에서부터(우선순위로) 실제 적용이 이루어지게 되어있다. 
	* 그렇기에 inherits를 쓰게되면 앞에 세팅된것을 그대로 가져다 쓴다는 의미가 된다.


<br>
<br>
### 4. 마치며
* 요 이슈는 간단히 해결했지만,ios 세팅설정은 참으로 애매하고 어렵고, 복잡하다.
* 인턴하다가 config에서 library path설정 때문에 시간을 엄청 날려버렸던 기억이 흘러간다.
