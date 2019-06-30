---
layout: post
title: 'Consul tutorial 02'
tags: [Consul,DiscoveryServer,HashCorp]
background: '/img/posts/bg_java.jpg'
---
Consul, Discovery Server

##### 다음 편이 준비되어 있습니다..

- [consul tutorial 1](https://yenoss.github.io/2019/04/04/Consul-tutorial-01.html)

- [consul tutorial 3](<https://yenoss.github.io/2019/04/04/Consul-tutorial-03.html>)



<br>

## 1. 왜.

* consul 학습


<br>

## 2. 그래서 무엇인가.

> Consul Cluster는 어떻게 작동되는가? 
* consul cluster 는  server client 구조로 여러 노드에서 consul이 하나의 시스템으로 작동하기위한 작업이다.
* clustering이 되어있으면 각 노드에서 개별로 서비스등록등이 가능하며  어느 노드에서든지 일관된 검색 결과를 가질 수 있다.

<br>

## 3. 써보자.

<br><br>
## Consul Cluster

- agent를 만들어보고 서비스를 등록하며 쿼리등을 agent에서 실행해봅니다.
- 서비스간 인증과 암호화에 대해서도 알아봅니다..
- 진짜 multiple members를 만들어보고 discovery 역할을 수행해봅시다.

<br>
## Starting Agent

- 실제 cluster를 simulating하기 위해 vagrant를 이용해 두 개이상의 노드를 구축할 수 있습니다.
- vagrant를 구동하기 위한 provider인 virtaulbox도 설치해줍니다.

    {% highlight go %}

    sudo apt-get install vagrant
    sudo apt-get install virtualbox

    {% endhighlight %} 

- vagrant  환경설정 세팅을 아래  [여기](https://github.com/hashicorp/consul/blob/master/demo/vagrant-cluster/Vagrantfile) 레포에서 다운받고 실행합니다.

    {% highlight go %}

    vagrant up

    {% endhighlight %} 

- **맥환경의 유저는 system에서 oracle을 막을 수 있습니다.  system환경, 보완및 개인정보에 들어가 oracle을 allow해줍니다.**

- 이제 `ssh n1` 으로   n1 node에 접속할 수 있습니다.

<br>
## SetUp Server Agent

- 이제 실제로 agent를 띄어볼겁니다.
- 그리고 첫번째 노드는  `서버` 로 사용될것입니다. 이제 실행할 agent의 플래그값중 일부를 확인해보겠습니다.
    - dev flag는 쓰지 않고, node에 이름을 명시해 줄겁니다.
    - bind address
      - 모든 클러스터가 접근할수있는 address여야 합니다.  production server들은 종종multiple interfaces 를 가지게 되는데  bind address가 명확하다면 잘못될 일이 없을것입니다.
    - bootstrap-expect 
      -  join될 agnet 들의 예상 수 입니다. consul은 지정된 수의 서버를 사용할 수 있을때까지 대기한 다음 클러스터를  bootstrap합니다. 이 과정에서 리더가 자동으로 선출됩니다.
    - enable-script-checks
      - healthcheck로  bash script을 쓸수 있도로고 하는 옵션입니다.

    {% highlight go %}

    consul agent -server -bootstrap-expect=1 -data-dir=/tmp/consul -node=agent-one -bind=172.20.20.10 -enable-script-checks=true -config-dir=/etc/consul.d

    {% endhighlight %} 

- 이제 `ssh n2` 으로   n2 node에 접속할 수 있습니다.
- 마찬가지로 client로 실행시켜줍니다.

    {% highlight go %}

    consul agent -data-dir=/tmp/consul -node=agent-two \
        -bind=172.20.20.11 -enable-script-checks=true -config-dir=/etc/consul.d

    {% endhighlight %}         

- 각각의 server와 client가 떴습니다. 그러나 아직 그 둘은 서로의 존재를 모릅니다.
- `consul members` 으로   멤버를 찾아보면 본인밖에 나오지 않을 겁니다.
- 이제 클러스터를 구축해야합니다.

<br>
## Join Cluster

- 서버 agent에게  다른 클라이언트 agent가 조인하겠다고 명령을 둡니다.
  
    {% highlight go %}
    
    consul join 172.20.20.11
    
    {% endhighlight %} 

- `consul members` 으로 각 노드에서 입력해보면 두개의 consul agent가 보일것입니다.

<br>
## Auto-joining a cluster on Start

- 이상적으로 생각하면 새노드가 등록되면 자동으로 붙으면 좋겠네요.
- consul은  aws,gcp등에서 tag key/value를 통해 이를 제공합니다. [여기](https://www.consul.io/docs/agent/cloud-auto-join.html)를 참고하세요.

<br>
## Querying Node

- 노드에 DNS HTTP 쿼리를 날려봅니다.

    {% highlight go %}
    
    dig @127.0.0.1 -p 8600 agent-two.node.consul
    
    {% endhighlight %} 

- 8600 포트는 consul의 dns 서버 포트입니다. consul agent가 실행될때 자동으로 dns서버가 뜨게됩니다.
- dig를 통해 agent-two.node.consul의  실제 ip주소를 알 수 있습니다.
- {NAME}.node.consul 혹은 {NAME}.node.{DATACENTER}.consul 의 형태로 실제 local domain을 얻어올 수 있습니다.

<br><br>
## 4.마치며

- 가장 궁금했던 cluster 이 어떻게 구축되는지 알 수 있게 되었다.
- 서버 분산환경에서 인프라 구축 혹은 service discovery 역할이 필수불가하다는 생각이 들었다.
- Vagrant를 통해 서버를 구성해보니 재미 있었고 인프라, 분산서비스 구축등을 테스트해 볼때 쓸모있겠다.
  

<br>

## Ref

- <https://learn.hashicorp.com/consul/getting-started/install>

