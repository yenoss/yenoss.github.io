---
title: swift Extension
date: 2017-05-10 23:53:16
categories: swift
---

<br><br>
# Extension


* 프로토콜 타입에 새로운 기능을 추가할 수있는 기능입니다.
* 타입만 알고있다면 그타입의 기능을 확장할 수 있습니다.
* objc 의 카테고리와 유사하죠.
* 새로운 정의는 가능하지만 기존 존재하는 기능을 재정의 할 수 없습니다.


<br>
### ex1
* 아래와 같은 형태로 사용합니다.
 
~~~
extension {이름}: {프로토콜1} {프로토콜2} {
}
~~~

<br>
## computedProperty

* 연산프로퍼티를 추가할 수 있습니다.	

<br>
### ex1

~~~

extension Int {
    var isEven: Bool {
        return self % 2 == 0
    }
    var isOven: Bool {
        return self % 2 == 1
    }
}
print (1.isEven)
var num: Int;
num = 3
print(num.isEven)
~~~

<br><br>
## Method

* 메서드를 추가 할 수 있습니다.

<br>
### ex1

* mutating을 통해 내부의 값을 변경할 수 있다고 알려줍니다.

~~~
extension Int {
    func multiple(by n:Int) -> Int {
        return n * self;
    }
    
    mutating func valueChangeAndMultiple(by n:Int) {
        self = self * n
    }
    static func isIntTypeInstance(_ isntance:Any) -> Bool {
        return isntance is Int
    }
    
}

print(1.multiple(by: 10))
var num: Int;

num = 10

num.valueChangeAndMultiple(by: 10)

print(num)

Int.isIntTypeInstance(10)
Int.isIntTypeInstance(110.3)
Int.isIntTypeInstance("sss")

struct Position {
    var x: Int
    var y: Int
}

extension Position {
    //inout 들어간값에 바로 값이들어간다.
    static prefix func ++(pos: inout Position) -> Position {
        pos.x += 1
        pos.y += 1
        return pos
    }
}

var poss:Position =  Position(x: 10, y: 20)
++poss

print( "\(poss.x) and \(poss.y)");
~~~



<br><br>
참고 & 출처 [스위프트 프로그래밍](http://http://www.hanbit.co.kr/media/books/book_view.html?p_code=B5682208459)