---
title: swift Monad
date: 2017-04-19 23:53:16
categories: swift
---

<br><br>
MONAD
====================


* 함수형 프로그래밍의 좋은 예.
* 특정한 상태로 값을 포장.
* 스위프트의 옵셔널은 스위프트의 모나드의 좋은 예.
* 특정기능이 아닌 디자인패턴 혹 자료구조.


<br>
Context
================

* 컨텐츠를 담고 있는 무언가이다.
* optional의 값을 가지않는다면 열거형에 .none, 가진다면 .some(value)로 값을 취한다.


<br>
ex1
------------

* addThree는 순수값 2를 받으면(컨텍스트에 담기지 않은) 함수를 정상 작동시킨다.
 
~~~

func addThree(_ num: Int) -> Int {
    return num + 3
}
~~~

* 그러나 optional로 감싼 2를 받게되면 오류가 발생하게된다.

~~~
addThree(Optional(2))
~~~


<br><br>
함수 객체
================

* map은 값을 변형시키는 고차함수이다. optional은 컨테이너와 값을 가지므로 맵 함수를 사용할 수 있다.


<br>
####ex1

* 옵셔널을 연산할 수 있도록 한다.

~~~
Optional(2).map(addThree)
~~~

<br>
####ex2
 
 
 ~~~
var value: Int? = 2
//Int는 optional로 감싸여 있음으로 optional(5)
print("\(value.map{ $0 + 3})")
value = nil
//nil == optional.none
print("\(value.map{ $0 + 3})")
 ~~~
 
* optional(2).map(addThree)는 어떻게 코드로 작동되는가?

1. optional(2)로 부터 값추출.
2. 전달받은 함수(addThree)에 값을 전달.
3. 다시 context에 담아 optional로 만들어준다.
4. !!@ 만일 값이 없다면 함수를 적용하지않는다. (아무일도 안함).

 

 
<br><br>
Real Monad
============

* monad는 함수 객체의 일종.
* 함수객체는 포장된 값에 함수 적용이 가능.
* monad는 컨텍스트에 포장된 값을 처리해 컨텍스트에 포장된값을 다시 반환하는 함수를 적용할 수 있음.
* 이러한 기능을 하는 녀석이 flatmap
* monad는 값이 있을수도 있고 없을 수도 있는 컨텍스트를 가지는 함수 객체 타입.

<br>
####ex1

* 복잡하다 코드를 참고하자.

1. 컨텍스트로부터 값을 추출(3).
2. 추출한값을 doubleEven 함수에 전달.
3. 값이 함수에 할당치 않아 nil을 반환함.


~~~
func doubledEven(_ num: Int) -> Int? {
    if num % 2 == 0 {
        return num * 2
    }
    return nil
}
Optional(3).flatMap(doubledEven)
~~~

<br>
####ex2

* 빈객체가 들어가면 어떻게 되지?

1. 빈 컨택스트의 빈 값.
2. 비어서 flatmap이 아무것도 안함.
3. 다시 빈 컨택스트 반환.

~~~
Optional.none.flatMap(doubledEven)
~~~

<br>
####ex3

* flatmap과 map의 차이는 내부의 값을 알아서 더 추출한다는 점
* 즉 optional을 벗겨서 준다. 
* 컨테이너를 풀어 값을 평평하게 만들어준다하여 flat map이라 한단다.

~~~
let optionalArr: [Int?]  = [1,2,nil,5]


let mappedArr: [Int?] = optionalArr.map({ (value:Int?) -> Int?
    in return value
})
//let mappedArr: [Int?] = optionalArr.map{$0}
let flatMapArr: [Int] = optionalArr.flatMap{ $0 }

print(mappedArr)
print(flatMapArr)
~~~

<br>
####ex4

* map 과 flatmap을 명확히 다시알아보자.
* 그리고 flatmap chainging

~~~
func stringToInt(str: String) -> Int? {
    return Int(str)
}

func intToString(integer: Int) -> String? {
    return "\(integer)"
}

var optionalString: String? = "2"
//flatmap은 계속해서 실제 값을 넘겨주기때문에 계속해서 다음 것을 체이닝 할수있다.
var result: Any = optionalString.flatMap(stringToInt).flatMap(intToString).flatMap(stringToInt)
//flatMap(stringToInt)
print(result)
//map은 반환을 optional로 하기때문에 체이닝을 걸수가 없다.
result = optionalString.map(stringToInt)
print(result)
~~~

<br>
####ex5

* flatmap chaning 중 빈 컨택스트를 만났을 경우.
* 중간에 nil을 만나게되면 모든 하위 메서드는 무시된다.

~~~
func intToNil(param: Int) -> String? {
    return nil
}

var optionalString: String? = "2"
var result: Any = optionalString.flatMap(stringToInt).flatMap(intToNil).flatMap(stringToInt)
print(result)
~~~

<br>
<br>
마치며
===========

* 스윕트의 절대 장점인 safe한 특징을 보다 쉽게 사용하기위해 만들어놓은 수단인것 같다
* 적당한 부분에서 사용해야할 것같고, optional chainging을 벗길경우 사용하면 유용할 듯 싶다.

<br><br>
참고 & 출처 [스위프트 프로그래밍](http://http://www.hanbit.co.kr/media/books/book_view.html?p_code=B5682208459)