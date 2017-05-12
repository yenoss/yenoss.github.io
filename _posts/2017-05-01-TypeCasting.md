---
title: swift typeCasting
date: 2017-05-01 23:53:16
categories: swift
---

<br><br>
# TypeCasting

* swift는 암시적 타입캐스팅을 지원하지 않습니다. 
* 꼭 명시를 해야한다는 이야기입니다. 
* is 와 as를 사용합니다.


<br>
## ex1

* 아래 예제를 통해 타입확인을 해봅니다.
 
~~~


class Coffee {
    let name: String
    let shot: Int
    
    var description: String {
        return "\(shot) shots(s) \(name)"
        
    }
    init(shot: Int){
        self.shot = shot
        self.name = "coffee"
    }
}

class Lattee: Coffee {
    var flavor: String
    override var description: String{
        return "\(shot) shots \(flavor) lattee"
    }
    
    init(flavor: String, shot: Int){
        self.flavor = flavor
        super.init(shot: shot)
    }
}


class Americano: Coffee {
    let iced: Bool
    override var description: String{
        return "\(shot) shots \(iced ? "iced":"hot") americano"
    }
    init(shot:Int, iced:Bool){
        self.iced = iced
        super.init(shot: shot)
    }
}


~~~

<br>
### ex1

* is 를 이용하여 해당 타입이 맞는지 확인 합니다. 

~~~
let coffee: Coffee = Coffee(shot:1)
print(coffee.description)

let myCoffee: Americano = Americano(shot: 1, iced: true)
print(myCoffee.description)


print(myCoffee is Coffee)
print(coffee is Americano)
~~~

<br><br>
## Meta type

* 타입의 타입을 의미합니다
* 구조체 타입, 열거형 타입 프로토콜 타입
* .Type으로 메타타입을 나타냅니다.

<br>
### ex1

* 타입을 타입으로 둡니다.

~~~

protocol SomeProtocol {}
class SomeClass: SomeProtocol {}

//인트타입을 가지는 변수 입니다.
let intType: Int.Type = Int.self
//스트링 타입을 가지는 변수입니다.
let stringType: String.Type = String.self

let classType: SomeClass.Type = SomeClass.self

let protocolProtocol: SomeProtocol.Protocol = SomeProtocol.self

//어느 타입이든 받을수있는 변수 someType입니다.
var someType: Any.Type

//어느타입이든 받을수있는 변수에 inttype을 넣으니 int가 찍힙니다. 
//다른코드도 마찬가지입니다.
someType = intType
print(someType)

someType = stringType
print(someType)

someType = classType
print(classType)

someType = protocolProtocol
print(someType)
~~~

* 뒤에 .self를 입력한것은 타입을 표현하는 값을 반환합니다.
* String.self 는 String 타입을 나타냅니다.

* of로 체크도 가능합니다

~~~
//true 커피 인스턴스가 Cofee타입이 맞느냐고 묻게됩니다.
type(of:coffee) == Cofee.self 
~~~



<br><br>
## downCasting


<br>
### ex1

~~~
let actingConstant: Coffee = Latte(flavo: "vanilla", shot: 2)
print(actingConstant.description)
~~~

* 위의 예는 lattee를 instance를 상위 Coffee 클래스의 인스턴스가 가지고 있는 형태입니다.
* 이러한상황에서 actingConstant가 Lattee에 접근하려 할수있습니다.
* 그렇게되면 down cating을 해야합니다.
* 타입캐스트 연산자인 as? as!를 사용합니다. 보다시피 실패할 수 있는 경우가 있음으로 ?일경우 nil을 ! 일경우 런타임오류를 발생시킵니다.

~~~
if let actiongOne: Americano = coffee as? Americano {
print("this si Americano")
}else {
	print(coffee.description)
}
~~~

* 위의 예제의 경우 coffee의 참조 인스턴스가 Americano타입이라면 actingOne에 할당하라는 것입니다. 
* 만약 Americano타입이 아니었다면 아래 커피 디스크립션이 출력될겁니다. 


<br><br>
## Any,AnyObject Type casting
* 특정 타입을 지정하지않고 여러타입의 값을 할당할 수 있는 Any,AnyObject라는 특별한 타입이 있습니다.

<br>
### ex1

~~~

func checkType(of item: AnyObject){
    if item is Lattee {
        print("item is Latte")
    } else if item is Americano {
        print("item is Americano")
    } else if (item is Coffee){
        print("item is Coffee")
    } else {
        print("unKown Type")
    }
}

checkType(of: coffee)
~~~

* item으로  어떠한 타입이든 받을수있게 한뒤 해당타입을 검사해 출력합니다.
* 간단합니다.

<br>
<br>
## 마치며

* 스위프트는 type safe한 언어입니다. 매우 중요한 파트라 생각합니다.
* 스위프트의 직관적인? 표현에 많이 익숙해진거같습니다. as 라는 표현이 굉장히 매력적이네요

<br><br>
참고 & 출처 [스위프트 프로그래밍](http://http://www.hanbit.co.kr/media/books/book_view.html?p_code=B5682208459)