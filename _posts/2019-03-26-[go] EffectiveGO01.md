---
layout: post
title: '[go] Effective GO 01편'
tags: [go,effective-go]
background: '/img/posts/bg_java.jpg'
---
go,effective-go

<br>

## 1. 왜.

* effective go



<br>

## 2. 그래서 무엇인가.

> go를 사용하는 best practice는 무엇일까?

* go언어를 처음 써본다. 잘 쓰려면 어떻게 해야될까?
* go 소스 코드를 공부한다.  go는  go( +c )로 만들어졌다. (실제로 추천하신 분이 계심.. 흠.. 말은 쉽긴한대..)

* 모든 언어(?) 의 Best Practice인 effetive 시리즈를 본다. (effetive java, effective python..)



<br>

## 3. 써보자.
* 결론: golang.org에서 제공해주는 effective go 를 정리해보자!
* 아주아주 간략히만 정리한다. 요약이 목적.

<br>

<br>

## Names

<br>

### Package names

- 좋은패키지: 이름은 짧고, 명확하고, 기억나기 좋도록 지어야 한다.
- 편의성을위해: 소문자로 싱글단어가 괜찮다. underscore, mixedCap 은 지양해야한다.
- 모든소스에서 고유할필요는 없다. 로컬에서 바꿀수도 있으니까.
- 예를 보자
  - buffered reader는 bufio 패키지의 Reader를 콜한다. bufReader가아니라.  bufio.Reader가 더 명확하기 때문.
  - once.Do, No once.DoOrWaituntilDone()

<br>

### Getter

- 자동제공하진 않지만 적절히 사용하는것은 좋다. 그러나 이름에 get을 넣거나 하는건 너무 관용적임. 
- Owner를 get하려면 getowner 보다 owner를 걍 사용해라.


    {% highlight go %}
    
    owner := obj.Owner()
    if owner != user {
        obj.SetOwner(user)
    }
    
    {% endhighlight %}




<br>

### Inerface names

- er suffix 를 사용해라. 
  - Reader,Writer,Formatter,CloseNotifier ..
  - 다만 read,write와같은  signiture한 이름을 피해라



<br>

## SemiColons

- go는 인터프린팅될때 규칙에 의해 세미콜론을 자동으로 붙인다.
- 개행전에 마지막 토큰이 식별자(int, string)인 경우, 숫자 또는 문자열 상수와 같은 기본 토큰중 하나 일때 붙인다.
  - break continue fallthrough return += --
- 토큰뒤에는 항상 세미콜론을 붙이게됨.



> if new line comes after atoken that could end a statement, insert a semicolon



- brace closing될때도 바로 붙게됨된다.
- 여러줄을 나눠줘야할 경우에만 세미콜론을 붙이게 된다.
  - if data,err = getData().Error ; err!=nil { 



<br><br>

## Control strcutures

<br>

### if


    {% highlight go %}
    
    if x >0 {
        return y
    }
    
    if err := file.Chmod(0554); err != nil {
        log.Print(err)
        return err
    }
    
    {% endhighlight %}


- 에러처리는 위와 같이 필요에따라 else 없이 하는 것이 좋다.

<br>

### Redeclaration and reassignment


    {% highlight go %}


    f, err := os.Open("data")
    d, err := f.Stat()
    
    {% endhighlight %}


- err는 위에서 사용되고 아래에 새로운 값으로 쓰인다.

<br>

### For


    {% highlight go %}
    
    sum := 0
    for i := 0; i < 10; i++ {
        sum += i
    }
    
    for key, value := range oldMap {
        newMap[key] = value
    }
    
    a := []int{1, 2, 3}
    for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
        a[i], a[j] = a[j], a[i]
    }
    
    {% endhighlight %}




- 대부분 일반 스크립트 언어와 비슷한데 마지막 코드는  multiple일 경우 ++ 대신 +1 -1 을 사용해야 된다.
- ++등은 명령문이기 때문에 라는데(잘 모르겠다.)

<br>

### Switch

- if else 체인 느낌식으로 사용하면 된다.


    {% highlight go %}


    func unhex(c byte) byte {
        switch {
        case '0' <= c && c <= '9':
            return c - '0'
        case 'a' <= c && c <= 'f':
            return c - 'a' + 10
        case 'A' <= c && c <= 'F':
            return c - 'A' + 10
        }
        return 0
    }
    
    {% endhighlight %}


<br><br>

## Functions

<br>

### Mutliple return valuse

- 함수는 많은 리턴을 가질수 있다. 


    {% highlight go %}
    
    func (file *File) Write(b []byte) (n int, err error)
    
    {% endhighlight %}




<br>

### Named result parameters

- 함수의 파라미터 결과 리턴은 이름을 줄수있다.
- 이름을 짓는법이 정해져있진않지만 짧고 분명해야 한다.


    {% highlight go %}
    
    func nextInt(b []byte, pos int) (value, nextPos int) {
    
    {% endhighlight %}



    {% highlight go %}
    
    func ReadFull(r Reader, buf []byte) (n int, err error) {
        for len(buf) > 0 && err == nil {
            var nr int
            nr, err = r.Read(buf)
            n += nr
            buf = buf[nr:]
        }
        return
    }
    
    {% endhighlight %}


<br>

### Defer

- Defer는 함수의 콜에 상태를 스케쥴 한다.
- 반환되기 직전에 실행해야되는것.


    {% highlight go %}
    
    // Contents returns the file's contents as a string.
    func Contents(filename string) (string, error) {
        f, err := os.Open(filename)
        if err != nil {
            return "", err
        }
        defer f.Close()  // f.Close will run when we're finished.
    
        var result []byte
        buf := make([]byte, 100)
        for {
            n, err := f.Read(buf[0:])
            result = append(result, buf[0:n]...) // append is discussed later.
            if err != nil {
                if err == io.EOF {
                    break
                }
                return "", err  // f will be closed if we return here.
            }
        }
        return string(result), nil // f will be closed if we return here.
    }
    
    {% endhighlight %}


- 리소스 반환같은곳이 중요한 쓰임새 중 하나이다.
- LIFO와 같은 형태에서 defer가 유용하게 쓰일수 있다.


    {% highlight go %}
    
    func trace(s string)   { fmt.Println("entering:", s) }
    func untrace(s string) { fmt.Println("leaving:", s) }
    
    // Use them like this:
    func a() {
        trace("a")
        defer untrace("a")
        // do something....
    }
    
    {% endhighlight %}



## 4.마치며

- effective하게 go를 쓰는 그날까지.. 
- 익숙해져서 잘썻으면 좋겠다. Go 짱..

<br>

## Ref

- https://golang.org/doc/effective_go.html

