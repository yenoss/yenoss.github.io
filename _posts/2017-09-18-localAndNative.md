---
layout: post
title: '[cordova] Localstorage And Nativestorage'
tags: [cordova]
---
cordova,localstorage, nativestorage

<br>
### 1. 왜.
*  모바일 앱에서 사용자의 메타 데이터등을 가지고 있어야한다. 서비스 내에서 지속적으로 사용딜 글로벌 데이터이기 때문이다.
*  이러한 데이터를 어디에 저장해야 가장 바람직할까?

<br>
<br>
### 2. 그래서 무엇인가.

<br>

* localStorage 
	* localStorage는 물리저장소에 저장된다. 즉 i/o 작업이다. 이는 큰데이터를 사용하기엔 올바르지 않다.
	
	* localStorgae는 데이터를 삭제하거나,앱을 지우기전까지 유지된다.

	
	* 보안적으로 다소 위험하다. 값이 그대로 노출되어있어 xss공격등에 대해 개방되어있다. 방어를 해도 노출되어있단 사실은 변함없다.(다만 앱이라는 환경은 일반 web에서 사용하는것보단 독립적으로 사용됨으로 안전할것이다)

<br>

* native Storage
	*  native Storage는 물리저장소에 저장된다.

	
	*  native Storage는 데이터를 삭제하거나,앱을 지우기전까지 유지된다.

	
	*  native ios/android에서는 userDefault,sharedPreference를 사용한다.

	
	* localstorage와 같이 브라우저, 즉 웹애플리케이션에 저장되는 것이아니라 디바이스 앱 자체에 존속되는 값이다(적어도 userdefault는 그러하다). 그러므로 상대적으로 보안이 우수하다.(앱 외부에서 접근하기 어렵단이야기다.




<br>
### 3. 써보자.
* localstorage는 웹에서 많은 예제로 쉽게 쓰임으로 패스한다.

* nativesgorage에 대해 사용해보자.

<br>
##### 설치

```
cordova plugin add cordova-plugin-nativestorage
```

<br>
##### 설정

* key/value 쌍으로 사용됨으로 쓰기 매우 수월하다.


* 자세한 예는 도큐먼트를 참고하자.

```
var obj = {name: "yenos", age: 24};
//저장하기.
//저장 로직 후 성공,실패 콜백이 떨어진다.
NativeStorage.setItem("reference", obj,function(obj){

}, function(err){

});
//가져오기 후 성공,실패 콜백이 떨어진다.
NativeStorage.getItem("reference", obj,function(obj){

}, function(err){

});
//삭제 후 성공,실패 콜백이 떨어진다.
NativeStorage.remove("reference", obj,function(obj){

}, function(err){

});
```

<br>
<br>
### 4.마치며
* 데이터를 어떻게 어디에 저장해야할까? 라는 질문엔 몇가지 포인트로 그 결정을 내릴 수 있을것이다.
	1. 저장할 데이터는 보안등급이 얼마나되는가.
	2. 저장 데이터의 지속성은 어떻게 되는가.(휘발성 데이터인가.영속석 데이터인가.)
	3. 얼마나 자주 읽고쓰게 되는가.(성능이슈)

* 서비스의 사이즈/상황마다 다르겠지만,필자의 상황에서는 위 3가지가 포인트였고, 경우에 따라 타협할수있는	storage가 위이 두 storage였다. 


* 모두 경우가 있다. 어떠한 선택을 할지, 그리고 부족한점은 어떠한 방식으로 채워나갈지에 대한 고민이 끝없이 이어져야 될것이다.

<br>
### REF.
1. [cordova-nativestorage](https://github.com/TheCocoaProject/cordova-plugin-nativestorage)






