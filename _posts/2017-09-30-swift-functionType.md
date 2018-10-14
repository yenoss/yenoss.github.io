---
layout: post
title: '[swift] function type'
tags: [swift,funcionType]
background: '/img/posts/bg_swift.jpg'
---
swift,functonType

<br>
### 1. 왜.

* 공부.


<br>
### 3. 써보자.
<br>

>functionType as Return Types, functionType을 ReturnType으로 쓸 수 있습니다.

{% highlight swift %}

//앞으로 한칸
func stepForward(input: Int) -> Int {
    return input + 1
}

// 뒤로한칸
func stepBackward(input: Int) -> Int {
    return input - 1
}

//선택
func chooseStepFunction(backword: Bool) -> (Int) -> Int {
    return backword ? stepBackward : stepForward
}

//현재 3에 있을경우
var currentValue = 3

// moveTozero는 currentValue를 가지고 선택을 하도록합니다.
var MoveNeartToZero = chooseStepFunction(backword: currentValue > 0)

while currentValue != 0 {
	//movetozero가 최초 3 backward값이 true
	// chooseStepFunction에서 stepBackwoard가 호출되고 currentValue가 input값으로 들어가 -1감소하게 됩니다.
	//.... 반복
	// currentValue가 0 이되면 whil문이 더 이상 돌아가지않게됩니다.
    currentValue = MoveNeartToZero(currentValue)
}
{% endhighlight %}


* 살짝 헷갈려서 아래와같이 코드를 바꿔보았다.

{% highlight swift %}

func stepForward(input: Int) -> Int {
    return input + 1
}
func stepBackward(input: Int) -> Int {
    return input - 1
}

func chooseStepFunction(backword: Bool) -> (Int) -> Int {
    return backword ? stepBackward : stepForward
}

var currentValue = 3

while currentValue != 0 {
    currentValue = chooseStepFunction(backword: currentValue > 0)(currentValue)
}

{% endhighlight %}


* 이래보니 chooseStepFunction이 보다 눈에 잘 들어왔다. backward값 넣고, 두번쨰인자로 currentValue로 넣어 자연스럽게 최초 return값에 currentValue가 들어가진다는 것을 알 수 있다.

>functionType as Paramter Types, functionType을 매개변수 type으로도 쓸 수 있습니다.

{% highlight swift %}

func addTwoInts(a: Int, b: Int) -> Int {
    return a + b
}
var mathfunction: (Int, Int) -> Int = addTwoInts

func printMathResult(mathfunction: (Int, Int) -> Int, a: Int, b: Int ) -> Int {
    return mathfunction(a,b)
}
print(printMathResult(mathfunction: mathfunction, a: 102, b: 0))
{% endhighlight %}


<br>

> functionType, 함수도 타입으로 사용할 수 있습니다. 일반적으로쓰닌 int와 같이 말이죠.

{% highlight swift %}

func addTwoInts(a: Int, b: Int) -> Int {
    return a + b
}

//타입을 정해주고, addTwoInts의 값을 줍니다.
var mathfunction: (Int, Int) -> Int = addTwoInts

print("Result: \(mathfunction(10,20))")
{% endhighlight %}



<br>
### REF.
1. [swift-official-doc](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/TheBasics.html#//apple_ref/doc/uid/TP40014097-CH5-ID309)





