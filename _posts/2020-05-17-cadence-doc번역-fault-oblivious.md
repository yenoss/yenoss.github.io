---
layout: post
title: '[번역]uber cadence docs 01.Fault-Oblivious'
tags: [Cadence,Uber,Asynchronous,Workflow,Activity,한글,번역]
background: '/img/posts/bg_java.jpg'

---

Cadence,Uber,Asynchronous,Workflow,Activity,한글,번역

<br>

## 1. 왜.

- [이 글](https://yenoss.github.io/2020/03/08/cadence.html)을 쓰다보니 더욱 이해하지 못할 일들이 많이 발생해서.. 뭘 제대로 보고있나 싶길래..

- uber cadenc를 더 잘 이해하기 위해 [doc 번역](<https://cadenceworkflow.io/docs>).

- 잘 못 이해하고 작성된 부분이 있으니 원본과 같이 보시길 바래요.

- *로 된 설명은 부가적인 docs외 필자의 설명이 첨가 되어있습니다.

  

  

## Concept

- Cadecne는  분산어플리케이션을 처음 개발하는 개발에게 friendly한 방법입니다.
- 이것은 workflow-automation 영역으로부터 용어들을 가져왔습니다. 그래서 [workflow](https://cadenceworkflow.io/docs/03_concepts/01_workflows)와 [activites](https://cadenceworkflow.io/docs/03_concepts/02_activities)가 concept 에 포함되어 있습니다.
- [deployment topology](https://cadenceworkflow.io/docs/03_concepts/06_topology)는 어떻게 모든 컨셉들이 배치할수있는 컨포넌트들과 연결되어있는지 설명하고 있습니다.
- *workflow는 concept으로 만들어진 서비스는 aws에도 있습니다.



<br>

## Fault-Oblivious Stateful Workflow Code

### Overview

- cadence의 주요 관념은 fault-oblivious stateful workflow 입니다.(*fault를 인식하지 못하도록 워크플로우가 `알 아 서 우리가 원하는(?) 예상하는`  처리해주는 방식을 이야기함)
- local variables 와 threads들의 workflow 상태는 process와 cadence service의 영향을 받지 않습니다. (*사용자의 코드는  그대로 `독립적으로 동작함`을 의미합니다.)
- 프로세스의 스레드처리, timer, event handler 를 캡슐하고있는 컨셉으로 매우 강력합니다. (*retry policy등의  timer, activity를 이용한 각 logic의 핸들링 등을 의미하는듯..)

<br>

### Example

- [cadence sample](https://github.com/uber-common/cadence-samples)과 [go clinet](https://cadenceworkflow.io/docs/07_goclient/) 를 참고하면 더 많은 정보를 보실 수 있습니다.



<br>

### State Recovery and Determinism

- workflow state recovery는 event sourcing으로 이용됨으로 몇 가지 코드를 작성하는데 제한이있습니다. 주요제한은 workflow code가 `derterministic` 해야 된다는 것이고 이것은 `여러번의 실행에서도 같은 결과`를 생성해야합니다.

-  workflow의 외부 api들은 간헐적 실패나 출력을 변경할 수 있으니 제외시켜야합니다. 그렇기에 모든 `외부 연결과의 커뮤니케이션`은 activity에서 일어나야 합니다. 같은 이유로 workflow 코드는 꼭 cadence api의  time,sleep, new thread를 사용해야 합니다. 

  - *cadence/workflow에서 제공해주는 workflow.sleep, workflow.go, 등이 그것입니다. cadenc api를 사용하지 않게되면 의도하지 않은 에러가 발생 할 수 있습니다.
  - *일례로 workflow.sleep을 쓰지않고 time.sleep을 쓰게되면 workflow execution timeout에 sleep된시간이 포함되지 않게됩니다.

  

  <br>

### ID Uniquenes

- `workflow ID`는 start workflow시에 client에 의해 발급됩니다. 대개 business level의 id, customer id, order id 같은 것이됩니다.
- cadence 한번에 domain 한 개당 하나의 ID를 workflow에게 발급해주는 것을 보장합니다. 같은  ID로 workflow을 실행한다면 `workflow execution already sarted` error를 반환합니다.
- 만약 완료된 workflow와 같은 ID를 발급받기 시도한다면,`workflowIdReusePolicy option`을 따릅니다. 
  - `AllowDuplicateFailedOnly`:  같은 ID로 이전에 workflow가 실패한 경우에 실행가능합니다.
  - `AllowDuplicate`: 이전 workflow가 compoletion 상태면 독립적인 실행을 허용합니다.
  - `RejectDuplicate`: 같은 ID로의 실행을 전혀 허용하지 않습니다.
- default는 AllowDuplicatedFailedOnly입니다.

<br>

### Child Workflow

- workflow 는 `child workflows`를 실행할 수 있습니다. child workflow는 성공하거나 실패하고 parent에게 report 되어집니다.
- 몇 가지 경우 child workflow가 사용됩니다. 
  - parent의 workflow가 포함되지 않는 개별의 worker에서 child workflow가 host가 될 수 있습니다.
  - single workflow는 size의 한계가 있습니다. 예를 들어 100k activities/workflow. child workflow는 작은 chunks로 파티셔닝 할 수 있습니다. 1개의 parent, 1000개의 children은 각 1000개의 activity가 실행가능하고 1 million의  activity가 실행 가능합니다.
  - child workflow로 uniquniness를 보장하기 위해 ID로 리소스 관리를 할 수 있습니다. 
    - 예를들어 호스트를 업그레이드 하는 workflow는 child workflow를 host마다 가질수있습니다. (hosname = workflowID) 그리고 모든 작업들이 sereialized 될 수 있습니다.
  - child workflow는 주기적인 로직을 parent history size를 늘리지 않고도 실행 할 수 있습니다. 
    - 예를 들어 parent가 child를 시작하면, 주기적인 로직을 필요한만큼 여러번 실행시킬 수 있습니다.
- child workflow는 single workflow가 모든 logic을  연이어 실행하는것과 대비되어 shared state가 없다는것이 주요 한계입니다. (*parent-child,child-child간의 하나의 workflow를 구성하는것처럼 쉽게 shared state를 사용하기 힘들다..) 
  - pareint와 child는 오직 asynchronus signal로만 통신이 가능합니다. 그러나 만약 둘사이가 tight coupling 된경우는  shared object state에 의존해 single workflow로 구성하는 편이 더 간단할것입니다.

<br>

### Workflow retires

- [good example](https://cadenceworkflow.io/docs/07_goclient/06_retries)



