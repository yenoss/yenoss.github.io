---
layout: post
title: ' swift  sort는 어떤 알고리즘으로 만들어졌을까?'
tags: [swift,introArray]
---
swift, introArray

<br>
### 1. 왜.

* 공부.

<br>
<br>
### 2. 써보자.
> swift의  sort는 어떤 알고리즘으로 만들어졌을까?

~~~
var names = ["names","ni","dd"].sorted(by:{ (s1:String, s2:String) -> Bool in
    return s1 > s2
})
~~~

* sorted라는 내장 함수이다. 이함수는 배열의 두개의 인자를 받아서 비교해서 큰값으로 재정렬이되 배열로만들어 반환해준다.

* sorted 함수는 어떻게 생겼을까? 

~~~
public func sorted(by areInIncreasingOrder: (Element, Element) throws -> Bool) rethrows -> [Element]
~~~

* 예상대로 튜플로 데이터 두개를 받고, bool로 리턴해주며 최종 sorted의 array를 리턴한다. 내부구현이 궁금하여 sorted origin소스를 보기로하였다.

~~~
  @_inlineable
  public func sorted(
    by areInIncreasingOrder:
      (Element, Element) throws -> Bool
  ) rethrows -> [Element] {
    var result = ContiguousArray(self)
    try result.sort(by: areInIncreasingOrder)
    return Array(result)
  }
~~~

* continuousArray는 라이브element에 지속적으로 추가가능한 c lavel의 array 라고한다.
* 해당 sorted가 콜될때 생기는 array에 areInIncreasignOrder를 다시 소트해준다.

~~~
  @_inlineable
  public mutating func sort(
    by areInIncreasingOrder:
      (Element, Element) throws -> Bool
  ) rethrows {

    let didSortUnsafeBuffer: Void? =
      try _withUnsafeMutableBufferPointerIfSupported {
      (bufferPointer) -> Void in
      try bufferPointer.sort(by: areInIncreasingOrder)
      return ()
    }
    if didSortUnsafeBuffer == nil {
      try _introSort(
        &self,
        subRange: startIndex..<endIndex,
        by: areInIncreasingOrder)
    }
  }

~~~

* 최종 소트 함수인대 해당  areIncreasingOrder를 가지고 introSort를 이용해 최종 소팅을 하게된다.
* 후에 자세히 블로깅하겠지만, introSort에서는 self를 inout 키워드로 받고,  엘리먼트를 분할하여 element의 distance에 따라 insertionSort,heapSort로 정렬한다.

### REF.
1. [swift-array.swift](https://github.com/apple/swift/blob/7dabd3ee1a0b36c8947b9598c798fb1002ed764a/stdlib/public/core/Arrays.swift.gyb)



