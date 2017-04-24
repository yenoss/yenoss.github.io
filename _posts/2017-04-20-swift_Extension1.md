---
title: swift Extension2
date: 2017-04-24 23:53:16
categories: swift
---

<br><br>
Extension(2)
====================

* 기능확장을 위한 여러가지들을 알아보자.

<br>
Class Initialization
================


* DesignatedInitialization(지정 초기화) 인스턴스를 생성할때 사용하는 초기화입니다.

<br>
ex1
------------
* 생긴게 이렇죠.
* 흔히 우리가 생각하는 그것입니다.
 
~~~

init() {
}
~~~

ex2 
------------

* Convenience Initializer(편의 초기화) 인스턴스를 생성할때는 아무 상관없는 초기화 입니다. 
* 클래스 설계자가 필요로 의해 필수적으로 초기화 할경우 이곳에 초기화하게 됩니다. 

~~~
convenience init(){
}
~~~

<br><br>
Desiginated/Conveience Initialization
================

* 자식 클래스의 디자이네이티드이니셜라이저는 부모클래스의 디자이네이티드이니셜라이저가 먼저 호출되어야만 합니다.
* 컨비니언스 이니셜라이저는 자신이 정의된 클래스의 다른 이니셜라이저를 반드시 호출해야합니다.
* 컨비니언스 이니셜라이저는 결국 디자이네이티드 이니셜라이저를 반드시 호출하게되어있습니다.

<br><br>
Swift safty-check
================

* 습트는 초기화 안전을위해 4가지 체크를 한답니다

1. 자식 클래스의 디자이네이티드 이니셜라이저가 부모 이니셜라이저를 호출전에 모든 프로퍼티를 초기화 하여야한다.
2. 자식 클래스의 다자이네이티드 이니셜라이저는 상속받은 프로퍼티에 값을 할당전 부모 클래스의 이니셜라이저를 먼저 초기화 하여야한다
3. 컨버니언스 이니셜라이저는 어떤 프로퍼티의 값을 할당하기전 다른 다자이네이티드 이니셜라이저를 호출 해야야한다
4. 1단계의 safecheck를 마치기전까지 인스턴스 메서드를 호출할수도, 프로퍼티값을 읽을 수도 없다




<br>
ex1
------------

* 위의 조건을 예로 초기화가 잘 이루어진경우

~~~
class Person {
    var name: String
    var age: Int
    
    init(name: String, age:Int){
        self.name = name
        self.age = age
    }
}

class Student: Person {
    var major: String
    init(name: String, age: Int, major: String){

        //1.부모클래스를 초기화 하기전에 자식클래스 프로퍼티는 모두 초기화 되어야한다.
        //2.부모클래스의 값을 할당하기전에 부모 이니셜라이저를 초기화 해야한다.(당근)
        
        self.major = "Swift"
        super.init(name: name, age: age)
    }
    //3.컨버니언스이닛은 값을 할당하기전  다른 이니셜라이저를 호출해야한다.
    convenience init (name: String){
        self.init(name:name, age: 7, major: "")
    }
}
~~~

<br>
ex2
------------
* 상속하여 같은 디자이네이티드 이니셜라이저를 쓰려면 override를 붙이면됩니다.
* convenience에선 붙이진 못합니다.(당연히 자식에서 부모 conveniece를 호출할수 없기에, 아래 특정한경우를 제외)
 
 ~~~
class Person {
    var name: String
    var age: Int
    
    init(name: String, age: Int){
        self.name = name
        self.age = age
    }
    convenience init(name: String){
        self.init(name: name, age:0)
    }
}

class Student: Person {
    var major: String
    override init(name:String, age: Int){
        self.major = "swift"
        super.init(name: name, age: age)
    }
    convenience init(name: String){
        self.init(name: name, age:1)
    }
}

~~~
 

 
<br><br>
Initialization 자동 상속
============

* 자식클래스에서 별도로 디자이네이티드 이니셜라이저를 구현하지 않는다면, 부모클래스의 이니셜라이저가 자동 상속됩니다.
* 위의 규칙에따라 자동상속받는경우 부모클래스의 컨버니언스 이니셜라이저역시 자동 상속됩니다.

<br>
ex1
------------

~~~
class Person {
    var name: String

    init(name: String){
        self.name = name
    }
    convenience init(){
        self.init(name:"unkown")
    }
}

class Student: Person {
    var age: Int = 0
}
//이게 되는 이유가바로 부모를 자동 상속
var student1: Student = Student(name: "hi")
print(student1.name)

//student2는 컨버니언스 이니셜라이저가호출되었다.
var student2: Student = Student()
print(student2.name)
~~~


<br><br>
Required Initialization
============

* 리콰이어 이니셜ㄹ아이저는 상속받은 자식클래스에서 반드시 이니셜라이저가 이루어져야합니다.
* 자동상속이 가능함으로 아래 예제와같이 값이 변경할경우에 표기하게 됩니다.

<br>
ex1
------------

~~~

class Person {
    var name: String

    init(name: String){
        self.name = name
    }
    //요구 이니셜라이저
    required init(){
        self.name = "unkown"
    }
}

class Student: Person {
    var major: String = "unkown"
    init(major: String){
        self.major = major
        super.init()
    }
    required init() {
        self.major = "unkOnwn"
        super.init()
    }
}
class UniversityStudent: Student {
    var grade: String
    init(grade: String){
        self.grade = grade
        super.init()
    }
    required init() {
        self.grade = "F"
        super.init()
    }
}

let jisoo: Student = Student()
//required init을 통해 unkown이 저장되고 name역시 언노운으로 설정된다
print(jisoo.major)
print(jisoo.name)

let pk: UniversityStudent = UniversityStudent(grade:"a")
print(pk.grade)
~~~



<br>
<br>
마치며
===========

* 다른 언어에서 보기힘든 다양한 분야의 상속 방식의 특성이 신기하였고, 어떻게 적용할지 고민하게되는 아주 좋은 공부였다다
<br><br>
참고 & 출처 [스위프트 프로그래밍](http://http://www.hanbit.co.kr/media/books/book_view.html?p_code=B5682208459)