---
layout: post
title: '[gRPC] golang:gRpc + protobuf + RestAPI'
tags: [grpc,rpc,protobuf,RestAPI]
background: '/img/posts/bg_linux.jpg'
---
grpc,rpc,protobuf,RestAPI

<br>

## 1. 왜.

* Remote Procedure Call 학습.



<br>

## 2. 그래서 무엇인가.

> high-performance, opensource univiesal RPC framework

* Google에서 만든 RPC 라이브러리이다. 
* RPC는 해석 그대로 원격에 있는 함수를 호출해주는 것을 컨셉으로 잡는다. 그렇기에 양쪽에 request, response를 다알아야함으로 인터페이스 규격과 코드를 protobuf로 만들고 만들어진 인터페이스를 통해서 실제 sever client가 통신을 한다. 
* Rest api는 응답값이 명시적이지 않아 서로의 인터페이스 약속을 명확히애야하고 json으로 http통신을 이용하여 전달하기때문에 상대적으로 느리다. 
* 주로 서버간 통신에서 많이 쓰이고 있다. 



<br>

## 3. 써보자.
* grpc + protobuf 를 이용한 restAPI를 만들어볼거다.



## Concept

- 사진과 같이 원래 기존 gRPC로 만들어진  echoService에 grpc-gateway 라이브러리를 이용하여 http 요청을 받는 proxy server를 한대 두어 proxy server가 client로 gRpc server에 붙는 모양으로 grpc+ restapi를 구현해 볼 예정이다.

<br>

![](/img/posts/hele-7ebd1358-a53c-45b2-b190-189f0e98939d.jpeg)

<br>

## 0. Installation

    {% highlight go %}
    
    go get -u github.com/grpc-ecosystem/grpc-gateway/protoc-gen-grpc-gateway
    go get -u github.com/golang/protobuf/protoc-gen-go
    (optional) go get -u github.com/grpc-ecosystem/grpc-gateway/protoc-gen-swagger
    
    {% endhighlight %}

- directory
    - [github.com/rpcRestService/message](http://github.com/rpcRestService/message)
        - generate 및 생성된 코드들 저장
    - github.com/rpcRestService/server
        - 실제 생성된 코드로 돌릴 서버 저장



<br>

## 1. gRpc Server 만들기

### myservice.proto

    {% highlight go %}
    
    ~/github.com/rpcRestService/message/myservice.proto
    
    syntax = "proto3";
    package example;
    import "google/api/annotations.proto";
    
    message StringMessage {
            string value = 1;
    }
        service MyService {
                rpc Echo(StringMessage) returns (StringMessage) {
                        option (google.api.http) = {
                                post: "/v1/example/echo"
                                body: "*"
                        };
                }
        }
    {% endhighlight %}

  

* StringMessage 는  struct와 같은 개념을 가지고 안에  value가 멤버면수(?)와 같은 형태로 존재하게 ehlsek.

- 실제 언어별로 해당 타입들이 선언되는 모양은 [여기](https://developers.google.com/protocol-buffers/docs/proto#required_warning)를 참고.
- 멤버변수에 정의된 고유 숫자가 보인다.
    - 고유한 넘버를 하나씩 부여해주면된다.
    - 메시지의 바이너리 포멧의 고유값으로 사용되는데 한번 타입정해지고나서 수정하면 안된다.
    - 1~15까지 1바이트로 인코딩 되어 사용되어짐(16~는 2바이트). 그러므로 자주 쓰는 것을 15이하로 설정되도록 설계하여야 한다.

<br>

### Generate gRpcStub

```shell
{% highlight shell %}

protoc -I/usr/local/include -I. \
  -I$GOPATH/src \
  -I$GOPATH/src/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \
  --go_out=plugins=grpc:. \
  myservice.proto
  
{% endhighlight %}
```

<br>

### gServer.go

    {% highlight go %}
    
    github.com/rpcRestService/server/gServer.go
    
    package main
    
    import (
            "fmt"
            pb "github.com/rpcRestService/message"
            "golang.org/x/net/context"
            "google.golang.org/grpc"
            "log"
            "net"
    )
    type server struct{}
    
    func (s *server) Echo(ctx context.Context, in *pb.StringMessage) (*pb.StringMessage, error) {
            strmsg := &pb.StringMessage{Value: in.Value}
            return strmsg, nil
    }
    
    func main() {
            lis, err := net.Listen("tcp", ":5505")
            if err != nil {
                    log.Fatalf("failed to listen: %v", err)
            }
    
            s := grpc.NewServer()
            pb.RegisterMyServiceServer(s, &server{})
            s.Serve(lis)
    }	
    
    {% endhighlight %}

- RegisterMyServiceserver()
    - 실제 이 함수를 myservice.pb.go 에서 찾아보면  두번째 인자가 Echo함수를 가지는 Interface라는 타입이라는 것을 알 수 있다.
    - 그러므로 우리는 임의 server 구조체를 하나만들어  Echo 를 interface에 맞게 구현해주면 되는것이다.
    - Echo는 우리가 .proto에서 정의한 그대로 선언되어있으며, input output을 보고 이쁘게 가공해서 사용하면된다.
- 여기까지  작성 후
    - go run gServer.go
- 이제 rpc  client 콜을 받을 수 있는  rpc서버가 완성됨.
- 다음은 rpc서버로 proxing해줄  ResvereProxy서버를 만들어야됨.

<br>

## 2. ReverseProxy Server

### myservice.pb.gw.go

    {% highlight go %}
    
    protoc -I/usr/local/include -I. \
      -I$GOPATH/src \
      -I$GOPATH/src/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \
      --grpc-gateway_out=logtostderr=true:. \
      myservice.proto
      
    {% endhighlight %}

- github.com/rpcRestService/message/myservice.pb.gw.go  파일이 생기게된다.
- 내부 코드 및 함수를 이용하여  grpc서버에 넘겨주는 역할을 하게된다.

<br>

### main.go

    {% highlight go %}
    
    github.com/rpcRestService/server/main.go
    
    package main
    
    import (
            "flag"
            "github.com/golang/glog"
            "github.com/grpc-ecosystem/grpc-gateway/runtime"
            gw "github.com/rpcRestService/message"
            "golang.org/x/net/context"
            "google.golang.org/grpc"
            "net/http"
    )
    
    func run() error {
            ctx := context.Background()
            ctx, cancel := context.WithCancel(ctx)
            defer cancel()
    
            mux := runtime.NewServeMux()
            opts := []grpc.DialOption{grpc.WithInsecure()}
            err := gw.RegisterMyServiceHandlerFromEndpoint(ctx, mux, "localhost:1234", opts)
            if err != nil {
                    return err
            }
            return http.ListenAndServe(":8080", mux)
    }
    
    func main() {
            flag.Parse()
            defer glog.Flush()
            if err := run(); err != nil {
                    glog.Fatal(err)
            }
    }
    
    {% endhighlight %}

- RegisterMyServiceHandlerFromEndpoint
    - 위와 마찬가지로 myservice.pb.gw.go 에 선언되어있고 유사한 역할을 한다.
    - localhost:1234는 gRpc Server에 connection하기 위한 주소를 넣어주면 된다.
    - 내부적으로는 grpcDial()를 통해 client Connection을 Create해준다.
- go run main.go  를 통해 서버를 실행시킨다.

<br>

## 3. Test

- gServer, main  두 가지의 서버를 모두 잘 작동시키면 아래와 같은 결과를 얻게된다.

    curl -X POST -k localhost:8080/v1/example/echo -d '{"value":"this is vlaue"}'

    {"value":"this is vlaue"}

<br>

## Ref

- [https://grpc.io/blog/coreos](https://grpc.io/blog/coreos)
- [https://github.com/grpc-ecosystem/grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway)
- [https://developers.google.com/protocol-buffers/docs/reference/go-generated](https://developers.google.com/protocol-buffers/docs/reference/go-generated)

