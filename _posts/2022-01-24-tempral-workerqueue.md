---
layout: post
title: '[번역]Cadence,Temporal Queue, Worker'
tags: [cadence,temporal,queue,worker]
background: '/img/posts/bg_linux.jpg'
---
cadence, temporal Queue Worker

<br><br>

##Cadence,Temporal Queue, Worker

이글은 [Temporal의 queue worker](https://docs.temporal.io/docs/temporal-explained/task-queues-and-workers/)에 관한 docs를 번역한 글입니다.

### Task

* task는  worker가 특정 ,workflowactivity를 실행하는데 필요한 컨텍스트입니다.

<br><br>

### Task Queues

- Task Queue는 polling 하는 worker에게 할당되는 dynamically, lightway한 큐입니다.
- 2가지 type의 큐들이 있는데, Actitity Task Queues, Workflow Task Queuse 입니다. 그러나 두큐는 각각 같은 이름으로 존재할 수 있습니다.



<br>

![](/img/temporal-taskqueue01.png)

> Workflow나 Activity Task가  scheduled되면  각 task Queue에 실제  task가 담기게된다. 이 taskqueue는 client,server입장에서는 prgramically하게 정하는 task-list-name이 바로 이것이다.

<br>

- Task Queue는 매우 가볍습니다.

  - Task Queues는 명시적 등록이 필요하진 않지만, workflow,activity가 실행될때나 worker proceess가 subscription 을 할때 생성됩니다.
  - Temoral 어플리케이션이나 Temorpal cluster가 관리하는 Tasks queue의 수는  no limit입니다.

- Worker의 taskqueue 에 동기 rpc로 polling합니다. 이는 몇가지 이득이 있습니다

  - worker process는 포트를 열필요가 없어 안전합니다.
  - worker process는 dns나 discovery machanism이 필요없습니다.
  - worker process가 모두 내려때, 메시지는는 task queue에 있고, worker가  recover될때까지 기다립니다
  - worker process는 여유 capacity가 있을때만 폴링하므로 스스로 overloading(과부하)되는 것을 방지합니다.
  - 실제로, taskqueue는 많은 worker process를 rebalancing을 가능하게 합니다.
  - Task Queues는 serversid throttling을 지원합니다. worker가 높은 비율로 스파크를 튀기고 있을때도 task dispatching rate pool로 제한이 가능합니다.
  - Task Queues는 Task의 구체적 task, worker로 심지어 구체적 process로 routing이 가능합니다,

### Worker Process



<br>

![](/img/temporal-taskqueue02.png)

<br>

> 그림을보면, taskQueue protocol로 dequeue task하여  task를 execute한다. task queue protocol은 workerprocess에서 구현되어진 workflow/activity를 register 하고  trigger 하는 일련의 protocol이다.



- worker process는TaskQueus에서 polling하고 Task를 빼내고, Task에 대한 응답으로 코드를 실행하고, task queue에 응답하는 작업을 담당합니다.




<br>

![](/img/temporal-taskqueue03.png)

<br>

> task queue는 worklfow,activity Task가 여러개 들어갈수있고(n:1),
>
>  workflow,activity Task는 worker1개의 여러개가 할당될수있습니다.(worker에서는 동시에 실행가능하니까),
>
>  worker는 해당  task를 실행하기위해 activity TaskQueue를 listening하고 있어야합니다.



