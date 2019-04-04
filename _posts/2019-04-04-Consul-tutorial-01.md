---
layout: post
title: 'Consul tutorial 01'
tags: [Consul,DiscoveryServer,HashCorp]
background: '/img/posts/bg_java.jpg'
---
Consul, Discovery Server

<br>

## 1. 왜.

* consul 학습



<br>

## 2. 그래서 무엇인가.

> Discovery서버는 무엇이며 어떻게 작동할것인가?

* Consul은  서비스를 구성하는데 여러 편리한 기능등을 제공해준다.
* Consul에서 이야기하는 유용한 경우는 아래와 같다.
  * Service Discovery
    * 서비스를 말그대로 찾는 기능이다. 
    * http,dns를 이용하여 실제 분산되어진 인프라 환경에서 원하는 서비스에 특정 콜을 날릴수 있도록 위치를(ip) 찾아준다. 
  * Service Segmentation
    * 정해진 네트워크에 의존하지 않고 안전하게 각 서비스들이 통신될 수 있다.
    * 분산인프라 환경에서 지속적 보안이 가능하다.
  * Service Configuration
    * 분산환경에서의 런타임 configuration update에 용이하다. 
    * 글로벌 인프라에 오케스트레이션이 가능하다.
* 경우에 맞게 써보지 않아서 잘 모르겠다..
* 감이 안오니 튜토리얼을 살펴보도록 한다.

<br>

## 3. 써보자.
* 결론: golang.org에서 제공해주는 effective go 를 정리해보자!
* 아주아주 간략히만 정리한다. 요약이 목적.

<br>

<br>
## Installation

- 컨설을 설치합니다.


    {% highlight go %}
    
    wget https://releases.hashicorp.com/consul/1.4.4/consul_1.4.4_linux_amd64.zip
    unzip consul_1.4.4_linux_amd64.zip
    sudo mv consul /usr/local/bin/
    consul -v 
      
    {% endhighlight %} 

<br>
## Run the Consul Agent

- consul이 설치되었으니 agent를 띄어봅시다.
- dc에 최소 한대 이상의 서버가 있겠지만, 3~5개의 cluster로 작동되도록 권장합니다.
  - sing server배포는 data loss 가 불가피하게 발생할 수 있기에..
- 모든 agent는 client 모드로 작동됩니다.
- client로 작동되는 프로세스는 매우 가벼우며 service register를 하고, health check, server query전송등을 합니다.
- 이 agent(client)들은 모든 노드에 돌아야하고 클러스터의 일부여야합니다.

<br>
## Starting the Agent

    {% highlight go %}
    
    consul agent -dev
    
    {% endhighlight %} 

- 개발모드로 agent를 실행합니다.

<br>
## Cluster Members

    {% highlight go %}
    
    consul members
    
    {% endhighlight %} 


- 현재 실행중인 client들을 확인 할 수 있습니다.

- -detailed

  - 추가 파라미터로 자세한 정보를 얻을 수 있습니다.

- members command는 [GossipProtocol](https://www.consul.io/docs/internals/gossip.html)을 기반으로 `최종적인` consistant data 입니다.

  - 즉, 어느 순간 로컬 agent의 state는 다를 수 있다는 이야기입니다.
  - 그러므로 실제 완벽히 consistant한 데이터를 얻으려면 HTTPAPI를 사용해야 합니다. 하기와 같이.

    {% highlight go %}
    
    curl localhost:8500/v1/catalog/nodes
    [{"Node":"Armons-MacBook-Air","Address":"127.0.0.1","TaggedAddresses":{"lan":"127.0.0.1","wan":"127.0.0.1"},"CreateIndex":4,"ModifyIndex":110}]
    
	{% endhighlight %} 


- 물론 DNS Interface역시 지원하기 때문에 local node name을 가지고 dns lookup query를 날릴 수 도 있습니다.

- dig(domain information Groper)를 사용하여서 쿼리를 날려봅시다.

    {% highlight go %}
    
    dig @127.0.0.1 -p 8600 {node_name}
    	
    {% endhighlight %}	

- 8600포트는 consul의 dns server 포트 입니다.

<br>
## Stopping the Agent

- Ctrl + C를 통해  agent를 종료합니다.
- 특정 node가 left하게되면 consul notify가 다른 cluster에게 알립니다.
- 특정 member가 강제적으로  fail 되면 health는 critical로 되지만 실제 컨설 기록에서 삭제되는 않습니다.
- consul은 자동적으로 failed node에게 reconnect request를 전송합니다.  물론 graceful leave client(개발자의 요청으로 삭제된, 정확하게는 unregister된것이지 않을까..) 에게는 시도하지 않습니다.
- adding 과 stopping에 관한 이야기는 [여기](https://learn.hashicorp.com/consul/day-2-operations/advanced-operations/servers)를 더 참고하면 좋습니다.

<br>
## Defining a Service

- 서비스는 서비스 정의에 대해서, 혹은 적합한 http api에 의해  생성될 수 있습니다.

1. configuration을 저장할 폴더를 만듭니다. 

   1. .d는 configuration 폴더라는 것을 의미 합니다.

    {% highlight go %}
    
	mkdir ./consul.d
    
    {% endhighlight %}

2. 80포트에서도는 web서비스를 만들어봅니다. 또한 추가적으로 tagging할 수 있습니다.

    {% highlight go %}
    
    echo '{"service": {"name": "web", "tags": ["rails"], "port": 80}}' \
        > ./consul.d/web.json
    
    {% endhighlight %}
    
3. configuration 으로 agent를 띄어봅니다.

    {% highlight go %}
    
    consul agent -dev -config-dir=./consul.d
    
    {% endhighlight %}
    
   - synced라는 메시지를 볼수 있게된다. 이는 등록한 service가 성공적으로 등록되어 기록되고 있다는 것을 의미합니다.


<br>
## Querying Services

- agent에 service가 등록되면 이제 dns, http api를 이용해 다양한 정보를 알 수 있습니다.

    {% highlight go %}
    
	dig @127.0.0.1 -p 8600 web.service.consul
    
    {% endhighlight %}
    
- web.servicec.consul로  a레코딩에 정의된 ip주소를 리턴 받을 수 있습니다.

- **또한**  서비스 되고 있는 실제 node이름을 받아 올수도 있습니다.

  - srv 레코드는 80포트에 존재하는 노드의 a레코드를 반환합니다.

- **또한** 앞서 설정한 tag값을 이용하여서도 검색이 가능합니다.

    {% highlight go %}
    
	dig @127.0.0.1 -p 8600 rails.web.service.consul
    
    {% endhighlight %}
    
- HTTPAI를 통해서도 query를 날릴 수 있습니다.

	{% highlight go %}
    
    $ curl http://localhost:8500/v1/catalog/service/web
    $ curl 'http://localhost:8500/v1/health/service/web?passing'
    
    {% endhighlight %}

<br>
## Connect

- consul은 TLS기반의 자동 connect를 제공합니다.
- Application은 수정될 필요가없습니다. Sidecar proxies를사용하면 connect를 의식하지말고  자동 TLS 연결을 할 수 있습니다.

<br>
## Starting a Connect-unaware Service

- connect를 서비스가 인지하지 못하도록 띄어봅니다.

- echo 해주는 socket프로그램을 띄어봅니다.

    {% highlight go %}
    
	socat -v tcp-l:8181,fork exec:"/bin/cat"
    
    {% endhighlight %}
    
- nc 명령어를 통해 connection 할 수 있습니다.

    {% highlight go %}
    
    nc 127.0.0.1 8181
    hello
    hello
    
    {% endhighlight %}    

<br>
## Registering the Service with Consul and Connect

- 새로운 서비스를 등록해봅니다.

    {% highlight go %}
    
    cat <<EOF | sudo tee ./consul.d/socat.json
    {
    "service": {
      "name": "socat",
      "port": 8181,
      "connect": { "sidecar_service": {} }
    }
    }
    EOF
    
    {% endhighlight %}    

- consul reload를 통해  SIGUP시그널을 컨설로 보내면 새로운 configuration 이 적용된 consul agent가 실행됩니다.

- connect를 통해 실제 특정 포트에서 연결된 inout bound를 넘겨주게됩니다.

    {% highlight go %}
    
	consul connect proxy -sidecar-for socat
    	
    {% endhighlight %}	

- 실제 코드를 실행해보면  8181포트에 한해서 proxy 프로세스가 실행됨을 확인할 수 있습니다.

  - 물론 지금 8181포트로 작동되는 서비스가 없기때문에 무수한 에러를 뱉을 것입니다.

<br>
## Connecting to the Service

- 실제 서비스에 연결해봅시다.

    {% highlight go %}
    
    consul connect proxy -service web2 -upstream socat:9191
    
    {% endhighlight %}    

<br>
## Summary

- 위에 수행한 connect관련 명령어들이 실행되어 있어야합니다.

	{% highlight go %}
    
    socat -v tcp-l:8181,fork exec:"/bin/cat"
    sudo consul agent -dev -config-dir=consul.d
    consul connect proxy -sidecar-for socat -log-level debug
    consul connect proxy -service web2 -upstream socat:9191
      
    {% endhighlight %}    

- 마지막으로 nc 명령어로 9191포트로 echo  message를 보내봅니다.  실제 8181  tcp포트를 열었지만 9191로 proxy되어 데이터가 전달되는 것을 확인 할 수 있습니다.

    {% highlight go %}
    
    nc 127.0.0.1 9191
    hello!
    hello!
    
    {% endhighlight %}    

작동되는 과정은 아래와 같습니다. 

- web2라는 임의의 서비스를  만들고, socatservice에 up stream으로 9191포트를 달아줍니다.
- 9191포트로 오는 모든 요청을 socat service에 넘기겠다는 것이며  넘어온 data 는  위에서 설정한 sidecar-for를 통해 프록시되어 최종 socat :8181 포트로 넘어가게됩니다.
- 하여 nc명령어를 날려보면 요청이가고 앞서 띄었던
  - socat -v tcp-l:8181,fork exec:"/bin/cat" 에 데이터가 프록시를 거쳐 잘 도착하게 됩니다.
- 여기서 말하는 service 이름은 실제 등록되는 서비스가 아닙니다. 단순히 해당 프록시를 나타내는 정보일 뿐입니다.
- connect proxy의 가장큰 강점은 긱존 8181로 띄어있는 application의 추가적인 작업을 하지 않고 프록시 할 수 있다는 점입니다.
- 8181로 열린 application에 단순  연결받을 포트(9191)를 정해주고, proxy service를 등록하는 것으로 application 포트를 9191도 받을 수 있게끔 만들었습니다!

![](/img/posts/img-consul.png)

<br>
## Registering a Dependent Service

- consul connect proxy를 이용하여 직접 연결해주었지만  config에  depends on 시켜서 sidecar proxy service를 붙일 수 있습니다.

    {% highlight go %}
    
    $ cat <<EOF | sudo tee ./consul.d/web.json
    {
    "service": {
      "name": "web",
      "port": 8080,
      "connect": {
        "sidecar_service": {
          "proxy": {
            "upstreams": [{
               "destination_name": "socat",
               "local_bind_port": 9191
            }]
          }
        }
      }
    }
    }
    EOF
    
    {% endhighlight %}    

- consul connect proxy -service web2 -upstream socat:9191

  - 위 명령을 하지않고 config에 web설정에 넣어두었음으로 connect proxy를 web으로 실행시켜줘야됩니다.

    {% highlight go %}
    
	consul connect proxy -sidecar-for web
    
    {% endhighlight %}

- 최종적으로 nc 127.0.0.1:9191 로 요청하면 마찬가지로 잘 작동하는 것을 확인할 수 있습니다.

<br><br>
## 4.마치며

- discovery server라는 것을 처음 알았는데 생각해보니 어디서든 언젠가 꼭 필요한 기능이란 생각이들었다.

- consul이 없었으면 실제 구현해야겠지.. 덜덜..

  

<br>

## Ref

- <https://learn.hashicorp.com/consul/getting-started/install>

