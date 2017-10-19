---
layout: post
title: '[swift] function'
tags: [swift,funcion]
---
swift,functon

<br>
### 1. 왜.

* 공부.


<br>
### 3. 써보자.
<br>


>In-Out Paramters, 기본 매개변수는 상수이고, 해당값을 임의로 조조작하려면 컴파일 오류가발생합니다. 매개변수값을 수정하고, 계속 유지하려면 in-out을 지정해줘야합니다.


~~~
func swapAB( a: inout Int, b: inout Int) {
    let tmpa = a
    a = b
    b = tmpa
}


var a = 10
var b = 20
print(swapAB(a:&a, b:&b))
~~~

* inout 을 해주지 않으면 `Cannot assign to value: 'a' is a 'let' constant` 에러가 발생합니다.


<br>

> Variadic Parameters, 하나도없거나, 많은 벨류들을 받을 수있다. 변화가있는 파라미터들..

~~~
func arithemeticMean( numbers: Double...) -> Double {
    var total: Double = 0
    for number in numbers {
        total += number
    }
    return total / Double(numbers.count)
}

<br>

arithemeticMean(numbers:1.1,2.4,3.5,1.2)
~~~

> Default Parameter Value
 

~~~
func someFunction(parameter: Int, parameterWithDefault: Int=12) -> Int {
    return parameter + parameterWithDefault
}
print(someFunction(parameter: 10))
print(someFunction(parameter: 10,parameterWithDefault: 20))
~~~

<br>

> Optional Tuple Return Types


~~~
// array부터 최소 최대 출력 함수.
// nil을 반환할 수 있음으로 optional하게 선언하기위해 return에 ? 가 들어감.
func minMax(array:[Int]) -> (min:Int, max:Int)? {
    if array.isEmpty { return nil }
    var curMin = array[0]
    var curMax = array[0]
    
    for value in array {
        if value < curMin {
            curMin = value
        }
        if value > curMax {
            curMax = value
        }
    }
    return (curMin,curMax)
}


var randomArr = [4,2,5,1,0,10,400,200,500,39,27,50,41]
if let tup = minMax(array: randomArr){
    print(tup.min)
    print(tup.max)
}else {
    print("return is nil")
}
~~~



<br><br>
### REF.
1. [swift-official-doc](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/TheBasics.html#//apple_ref/doc/uid/TP40014097-CH5-ID309)





