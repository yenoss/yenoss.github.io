---
layout: post
title: 'Consul tutorial 03'
tags: [Consul,DiscoveryServer,HashCorp]
background: '/img/posts/bg_java.jpg'
---
Consul, Discovery Server

<br>

## 1. 왜.

* consul 학습


<br>

## 2. 그래서 무엇인가.

> Consul HealthCheck 어떻게 작동되는가? 
* consul HelathCheck는 말그대로 consul cluster로 구성된 서비스에 HealthCheck를 의미한다.
* 특정 노드, 서비스 등의 status를 관리하는 용도로 사용됩니다.

<br>

## 3. 써보자.

<br><br>



## Registering Health Checks

- 대략 consul의 동작을 알았습니다. service, node를 붙이고 query도 날릴 수 있게 되었죠.
- 이제 우리가 붙인 service에 어떻게 health check를  할지에 대해 알아봅니다.
- helath check는 무지무지 중요합니다. service의 상태는 전체 시스템에 동작하는 여부에 아주 큰 결정권을 가지고 있기 때문입니다.



<br>

## Defining Checks

- service register와 비슷하게 check definition을 이용하여 helth check 를 붙입니다. 물론 http 통신으로도 붙일 수 있구요.

- 여기선 bash를 통해서 보낼겁니다. 당연 `enable_script_checks` 가 true로 설정되어 돌아가있어야합니다.

- n2에 접속하여 config 파일을 추가해줍시다.

    {% highlight go %}
    
  echo '{"check": {"name": "ping",
    "args": ["ping", "-c1", "google.com"], "interval": "30s"}}' \

  > /etc/consul.d/ping.json

  echo '{"service": {"name": "web", "tags": ["rails"], "port": 80,
    "check": {"args": ["curl", "localhost"], "interval": "10s"}}}' \

  > /etc/consul.d/web.json
  
    {% endhighlight %} 

- 첫 번쨰 check는 host-level의 check입니다.

  - bash 기반의  health check는 exit code에 의해 결정됩니다.
  - exit code란 bash 명령어가 실행되고나서 리턴되는 내부적인 code값입니다. ( exit code가 궁금하시다면 [여기보세여](http://www.tldp.org/LDP/abs/html/exitcodes.html))
  - exitcode ≥ 2가 된다면 error를 1이라면 warning state를 떨궈줍니다.

- 두 번째 check는 서비스에 달린 check입니다.

  - web이라는 서버를 선언했으며 10초마다 helathcheck를 하게됩니다.
  - exit code 로 작동됩니다.

- 다됬네요. 리로드합시다.

    {% highlight go %}
    
    consul reload
    
    {% endhighlight %} 

- 실제 n2 agent에서 web은 에러가 날겁니다. 왜나면  we 서버가 돌고있지않아서 curl test 가 실패하기 떄문이죠!



<br>

## Checking Health Status

- 이제 우리는 간단한 check를 추가할 수 있습니다. HTTP api를 통해 health status를 봅시다.

    {% highlight go %}
    
  curl http://localhost:8500/v1/health/state/critical
  [
    {
      "Node": "agent-two",
      "CheckID": "service:web",
      "Name": "Service 'web' check",
      "Status": "critical",
      "Notes": "",
      "Output": "",
      "ServiceID": "web",
      "ServiceName": "web",
      "ServiceTags": [
        "rails"
      ],
      "Definition": {},
      "CreateIndex": 218,
      "ModifyIndex": 218
    }
  ]
  
    {% endhighlight %} 

- 위와 같이 critical한 status만 받아볼 수 있습니다.

- 실제 달린 서비스의 좀더 자세한 status를 보고 싶다면..

    {% highlight go %}
    
  curl http://127.0.0.1:8500/v1/agent/checks

  {
    "ping": {
      "Node": "agent-two",
      "CheckID": "ping",
      "Name": "ping",
      "Status": "passing",
      "Notes": "",
      "Output": "google.com\n",
      "ServiceID": "",
      "ServiceName": "",
      "ServiceTags": [],
      "Definition": {},
      "CreateIndex": 0,
      "ModifyIndex": 0
    },
    "service:web": {
      "Node": "agent-two",
      "CheckID": "service:web",
      "Name": "Service 'web' check",
      "Status": "critical",
      "Notes": "",
      "Output": "  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current\n                                 Dload  Upload   Total   Spent    Left  Speed\n\r  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0curl: (7) Failed to connect to localhost port 80: Connection refused\n",
      "ServiceID": "web",
      "ServiceName": "web",
      "ServiceTags": [
        "rails"
      ],
      "Definition": {},
      "CreateIndex": 0,
      "ModifyIndex": 0
    }
  }
  
    {% endhighlight %} 

<br><br>

## 4.마치며

- healthcheck는 discovery 서버의 가장 중요한 요소이다. 서비스의 상태 자체를 관리하기 때문이다.
- 상태를 관리하기 위한 조건이 중요할듯하다. 즉 bash 명령을 줄때 성공이 나오는 조건이 무엇이냐가 중요하다는 이야기다.
- 서비스가 상태가  passing인지 critical인지 등을 시스템의 민감도를 적당히 고민하여 정하여 할 것이다. 

<br>

## Ref

- <https://learn.hashicorp.com/consul/getting-started/install>

