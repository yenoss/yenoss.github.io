---
layout: post
title:  "CHAPTER 13 swift closure"
date:   2017-4-12 15:37:00
categories: Swift
---

Swift 
========




CHAPTER 13 swift closure
---
201704/12
---


//CHAPTER 13 클로져
//기본 클로져

~~~
let namesss: [String] = ["win","eric","yenos","jenny"]
func backwards(first: String, second: String) -> Bool {
    
    return first > second
}

let reversed: [String] = namesss.sorted(by: backwards)
~~~

//클로져로 확인해보자.

~~~
let reversedd: [String] = namesss.sorted(by: { (first: String, second: String) -> Bool in
    return first > second
})
~~~

//후행 클로져

~~~
let reverseddBackClouser: [String] = namesss.sorted(){ (first: String, second: String) -> Bool in return first > second }
let reverseddBackClouser2: [String] = namesss.sorted{ (first: String, second: String) -> Bool in return first > second }
~~~

//타입유추 클로저

~~~
let reversedBackClouser3: [String] = namesss.sorted{ (first, second) in
    return first > second
}
~~~

//$0, $1은 각각 처음 두번쨰 파라미터를 의미한다

~~~
let reversedBackClouser4: [String] = namesss.sorted {
    return $0 > $1
}
//심지어 return도 없앨 수 있다.
let reversedBackClouser5: [String] = namesss.sorted {
    $0 > $1
}
~~~

//값의 획득
//클로져는 자신의 정의된 위치의 주변문맥을 통해 상수나 변수를 획득 할 수 있습니다.

//자세히보면 반환하는것이 함수객체 그자체이다.

~~~
func makeIncrementer(forIncrement amout: Int) -> (() -> Int) {
    var runningTotal = 0
    func incrementer() -> Int {
        runningTotal += amout
        return runningTotal
    }
    return incrementer
}
~~~

//리턴이 ()-> 타입임임으로 이와맞게 변수를 만들어주고 이를 콜해본다.

~~~
let incrementByTwo: (() -> Int) = makeIncrementer(forIncrement: 2)

let first: Int = incrementByTwo()
let sec: Int = incrementByTwo()
let third: Int = incrementByTwo()

let incrementByTwonRef: (() -> Int) = makeIncrementer(forIncrement: 2)
//클로져는 참조형태이기떄문에 이렇게 할당시켜버리면 값에 영향을 미친다.
let incrementByTwonReff: (() -> Int) = incrementByTwonRef

incrementByTwonRef()
incrementByTwonReff()
~~~

//탈툴클로져
//함수가 종료될때 클로저가 함수를 탈출한다고 이야기함.
//sorted by처럼 끝나는 클로져(콜백)이 없을때는 @escaping 명시안해주어도된다.
//즉 콜백이있을때만 명시(리스너)

~~~
typealias VoidVoidClosure = () -> Void

//두개의 클로저는 값으로 사요될 수있다.
let firstClosure: VoidVoidClosure = {
    print("ClosureA")
}
let secondClosure: VoidVoidClosure = {
    print("closureB")
}
//값으로사용되는 두개의 클로저에게 @escaping을 붙여주었다.
func returnOneClosure(first: @escaping VoidVoidClosure, second: @escaping VoidVoidClosure, shouldReturnFirstClosure: Bool) -> VoidVoidClosure {
    return shouldReturnFirstClosure ? first : second
}

let returncedClosure: VoidVoidClosure = returnOneClosure(first: firstClosure, second: secondClosure, shouldReturnFirstClosure: true)
//closureA가 실행된다.
//@escaping을 명시하지않으면 컴파일 에러가날것이다.
returncedClosure()

var closures: [VoidVoidClosure] = []
//함수내에서 외부에변수에 할당되어사용될것임으로 escaping을 꼭 붙여주어야한다.
func appendClosure(closure: @escaping VoidVoidClosure){
    closures.append(closure)
}
~~~

//자동 클로저
//자동 클로저는 전달인자를 갖지 않는다.
//자동클로저는 호출되기전까지 내부코드는 동작하지 않는다.

~~~
var customersInLine: [String] = ["ya","san","Sung","Hami"]
print(customersInLine.removeFirst())
print(customersInLine.count)

// 클로저를 만들어두면 실행하지 않고 가지고만 있는다.
// 하나의 명령 문장.
let customerProvider: () -> String = {
    return customersInLine.removeFirst()
}
print(customersInLine.count)

print("Now serving \(customerProvider())")
print(customersInLine.count)
//customerProvider 는 매개변수는 없고 리턴은 String은 함수이다.
func serveCustomer(_ customerProvider: () -> String) {
    print("now Serving \(customerProvider())")
}
//함수를 매개변수로 전달하였다.
serveCustomer({customersInLine.removeFirst()})

~~~

//위의 예로 자동 클로저를 작성해본다.

//autoClosure 옵션을 주게되면 해당 매개변수는 클로저 대신 실행결과 string의 문자열을 전달인자로 받게된다.
//해당 string을 반환갑이 string인 클로저로 변환시켜주는 역할을한다
//해서호출시 {}등의 문법적 표현이 줄어든다.

~~~
func serveCustomer2(_ customerProvider: @autoclosure () -> String){
        print("now serving \(customerProvider())")
}

serveCustomer2(customersInLine.removeFirst())
~~~

