---
layout: post
title: '[err] xcode build 이슈 /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/swiftc failed with exit code 1'
tags: [swift4,cocoapod,xcode9]
background: '/img/posts/bg_swift.jpg'
---

<br>
### 1. 무엇인가.

* xcode9. swift4.0.
* core data object의 파일을 찾지못한 빌드 문제였다. 

<br>
<br>
### 2. err 상황.

* coredata를 적용하던 도중. NSObject subclass를 만들고 지웠다를 몇번 반복하였더니 다음과 같은 에러가 발생했다.


{% highlight swift %}

Command /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/swiftc failed with exit code 1

{% endhighlight %}

<br>
### 3. 해결.

* 필자와 같은 경우는 문제는 같은 파일을 만들다 지우고, 빌드시 몇번 사용하고 보니까. 해당 파일이 남아있어서 발생하는 것이었던 것이다.(사실 명확하진 않고 정황상 그렇다.)
* 해서 몇 가지 시도를 통해 해결하게되었는데..


1. build 폴더를 삭제하는 clean builder  
  * `command` +` alt` + `shift` +`k` 
  * 일반 클린빌드는 현재 타겟에 대한 필요한 라이브러리등을  지웠다가 다시 삭제하는 정도다.  하지만 위의 명령어는 build폴더를 지웠다가 삭제하는 방식이다.

2. user-> library-> developer-> xcode->derived-data 삭제
	* derived-data는 빌드시 발생하는 파일들이 모여있다. 즉 여기서 아직 남아있는 파일들이 존재할수있기때문에 완전히 지워주는 것이다. 빌드하면 다시 폴더가 생기니 그냥 삭제해도 된다.

3. project target - buildphase - compileSource에 존재하는 copy file등에 혹시나 빨간색으로 위치를 인식하지못하는 것이 있다면 제거해야한다. 
	* 지금 이슈 외에도 가끔 copy에러가 발생할때 위와 같은 방법을 사용하곤했는데, 이도 문제가될 수 있는 듯하여 처리해주었다.(stackoverflow)

4. reboot
	* 사실 최종적으로 해결한 방법이 되겠다. 위와같은 일들을 다 처리하여도 빌드가 되질않아. 맥을 재실행하니 빌드가 잘되었다.


<br>
<br>
### 4.마치며
* 이상하다. 컴퓨터를 껐다 켰더니 마법같이 해결되었다는것이 이해가 되질 않았다. 2000년대 해결방식이 아니던가..
* 무튼 이제 저런 애매한 이슈는 완벽히 해결할 수 있게되었다.
