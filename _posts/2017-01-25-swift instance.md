---
layout: post
title:  "CHAPTER 11 인스턴스"
date:   2017-4-11 15:37:00
categories: Swift
---

Swift 
========




CHAPTER 11 인스턴스
---
201704/11
---

~~~
class Personn{
    var name: String
    var age: Int?
    //init() 안에는 무조건 stored property 와 binding 되어있어야하지만
    //age를 값을 가지지 않는 optional로 설정해두면 꼭 binding해주지않아도된다.
    //이때 age에는 자동적으로  nill이 붙게된다.
    init(name:String){
        self.name = name
    }
}

var choi: Personn = Personn(name:"choiPyeongKang")
~~~

//위와같이 매번 init에 sotreproperty를 매칭하는것은 너무 귀찬은일임
//struct는 memberwise initailize를 제공해줌 => stored property일경우 자동으로 init을 만들어주는것

~~~
struct animal{
    var name: String
    var kind: String
}

var someAnimal: animal = animal(name: "tiger", kind: "catt")
someAnimal.name
someAnimal.kind
~~~

//but class 는 지원하지 않음.


//실패가능한 initialzer

~~~
class Personnn{
    let name: String
    var age: Int?
    
    init?(name: String){
        if name.isEmpty{
            return nil
        }
        self.name = name
    }
}

//초기화자체가  null 일수 있음으로 ?를 붙여준다.
let choi01: Personnn? = Personnn(name: "")

//
if let p:Personnn = choi01 {
  print(p.name)
} else {
  print("has null value")
}
~~~

//함수를통한 store property setting
//아래와같이 초기화를 간단히 함수를통해 설정 할 수 있다.

~~~
struct StudentA {
    var name: String
    var age: Int
}
class schoolClassA {
    var student: [StudentA] = {
        
        var arr: [StudentA]  = [StudentA]()
        for num in 1...100 {
            var student:StudentA = StudentA(name: "hi", age: num)
            arr.append(student)
            
        }
        return arr
    }()
}

var choi02:schoolClassA = schoolClassA()
choi02.student
~~~


//Deinitializer
//class의 인스턴스가 소멸되기전에 구현된다. 크래스에서만 구현됩니다. 
//단하나만 구현될수있습니다.

~~~
class someClassB {
    deinit{
        print("delete this classs")
    }
}
var choi03: someClassB? = someClassB()

choi03 = nil
~~~


CHAPTER 12 접근제어
---
* open - framework수준의 클래스 오픈 (개방접근/only class)
* public -어디서든쓰일수있음
* internal - 모듀내부(import 로 불러오는 하나의 단위)
* fileprivate - 파일내에서만 그요소가 구현된 소스파일
* private - 기능정의에서만 같은소스라도 타입이나 기능이다르면 쓸수없음
