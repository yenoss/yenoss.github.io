---
layout: post
title: '[번역]uber cadence docs 02.Activity'
tags: [Cadence,Uber,Asynchronous,Workflow,Activity,한글,번역]
background: '/img/posts/bg_java.jpg'

---

Cadence,Uber,Asynchronous,Workflow,Activity,한글,번역

<br>

## [이 글](https://yenoss.github.io/2020/05/17/cadence-doc%EB%B2%88%EC%97%AD-fault-oblivious.html)에 이어서...

<br>

- ## Activites

  - Fault-oblivious stateful workflow code는  코어한 cadecne abstration이다.
  - 그러나 꼭 determinstiac해야하는 요구사항이 있습니다. 외부 api와의 direct가 안되고 대신 activity의 실행을 orchestration 합니다.
  - 간단하게 cadence activity는 fucntion, method이다. cadence는 failure cade의 activity의 상황을 recover해주지 않는다. 그러므로 activity function은 제한 없는 어떤 코드든지 상관없다. 
    - *workflow는 이러한 단일적인 acitvity를  컨트롤하고, 실제 이 activity단위로 recover여하가 결정된다.  activity자체는 내부에서 실패되든 말든 cadence의 recover관심 밖이다.(물론 activity  retry policy를 stickey 하게 제공해주긴 한다)
    - *예를 들면 workflow가 돌다가 activityA, activityB 사이에 sig kill등을 받게되면, 다른 worker에서 workflow가 실행될때 앞서 실행된 activtyA는 pass하고 activityB부터 실행된다.(workflow의  retry policy와는 상관없이) 
      - *이런 경우는  cadence가 orchestration해준다고 이야기할 수있는 부분이다. 그러나 activity는 workflow와 같이 중간에 죽는다고 특정 state를 이해하고 다시 실행되지 않는다. 진짜 리얼 사용자 코드(?) 이다. 날것 그자체.
  - activity는 taskLists를 통해 비동기적으로 동작합니다. task list는 activity task가 worker에 의해 pirk up되어질때까지  저장하기위한 필수적인 queue입니다 .
  - worker는 구현되어진 함수에 의해 activity를 처리합니다.
  - function 이 return될때 worker는 cadence service에 result를 report 하고 cadence service는 workflow에게 완료에 대해 알립니다. 그렇기에 다른 프로세스에서 완료하여 activity를 완전히 비동기식으로 구현할 수 있습니다. 
    - worfklow내부에서 각 activity간의 asynchronous할 수 있는 이유가 activity의 result를 cadence service⇒ workflow로 전달하기 때문이다 라고 이야기하는듯..

  <br>

  ### Timeouts

  - cadence는 acitivty duration에 관해 어떠한 시스템 제한을 두지 않습니다. 실행시간의 초과는 어플리케이션에게 달려있기 때문입니다.
  - 몇 가지  activity관련 timeout이 있습니다. 
    - ScheduleToStart: workflow가 worker에게 실행해달라고 요청하고 시작 실행의 maximum time입니다. 
      - 대개 이 timeout의 발현 이유는 worker가 다운 모두 다운되거나 요청 받는 속도의 제한입니다.
      - 모든 작업자가 일중이거나 중단이어도 workflow가 기다릴수있는 최대 시간을 권장합니다.
      - *workflow.ExecutionAcitivty  <———→ worker(activity실행 직전)사이의 시간이라고 보면된다. qeueing되어서 worker가 뽑아가 실행 할때까지의 시간이 될것이다.
    - StartToClose: activity가 worker에 의해 실행된 이후의 activity의 최대 시간을 의미합니다. 
      - *activty start ←——> end 까지의 시간이라고 보면되고 activity의 function이 실행되어질 최대시간이다.

  <br>

  ### Long Running Activities

  - long running acitivies를 위해서는 비교적 짧고 끊임없는 heartbeat timeout을 주는 것을 추천한다. 이방법은 long running acitivity에서 worker가 failure가 되어도 적시에 handle할수있게 해준다.
  - hearbit timeout이 지정된 activity에서는  heartmit method가 구현된 것을 지속적으로 call 하게  기대되어진다.
  - hearbeat request는 구체적 페이로드를 포함할 수 있다. 
    - 이것은 activity의 execution progress를 저장하는데 유용하다. 만약 activity timetout이 hearbeat miss로 발생했다면 다음 activity 실행시 progress에 접근할 수 있고 progress를 포인트로 그 이후에 작업을 실행 할 수 있다.
    - *RecordHeartbeat, GetHeartbeatDetails 등을 통해 heartbeat에 특정 값을 record하거나 가져올 수 있도록 되어 있는것으로 보인다.
    - *record되어진 정보의 라이프사이클이 어떠한 규칙으로 돌아가는지는 아직 명확하진 않다.
  - long running activity는 leader election같은 특별한 케이스에 사용될수 있다.  cadence timeout은 초단위로 동작하고 그래서 realtime app에는 적합하지 않을 수 있다. 그러나 만약 몇 초내에 프로세스 실패를 즉각적으로 알아야 되는 경우가 아니라면 heartbeat activity가 좋은 fit 이다.
  - leader election과같은 케이스의 공통된 경우는 모니터링이다. activity는 내부 루프로 주기적으로 api를 poll 할수 있고 condition을 체크할 수 있다. 매 주기에  heratbeat로 사용하면 된다. 만약 조건이 만족한다면 activity를 completes하고 workflow에게 handling을 맡기면 된다.
  - 만약 activity가 죽게되면  heartbeat interval이 초과되고, activity 의 timeout이 끝나면서 다른 워커에 retry 된다. 같은 패턴으로 워커가 새로운 amazon s3 bucket을 polling하거나 rest 의 응답을 받는등 asynchronous api에 사용될 수 있다.

  <br>

  ### Cancellation

  - workflow는 activity에게 cancel을 요청할 수 있습니다. 현재 activity가 cancelled를 알수 있는 유일한 방법은 hearbeating 뿐입니다. activity cancelled를 나타내는 error와 함께 heartbeat request fails이 됩니다.
  - cancelled후에 clean up이나 reporting activity의 구현에 달렸습니다.
  - 또다른 heartbeat failure 상황은 workflow의 완료 state입니다. 이 케이스또한 activity가  cleanup 되어야할 것입니다. 
    - *heartbeat의  cancel signal을 실어 보낸다는 이야기고, 내가 외부에서 cancel 요청을 날려도 해당 activity에 heartbeat주기에 맞춰서 cancel된다는 이야기다. 즉 내가 넣은 cancel이 realtime 으로 동작 되지 않을 수 있다.
    - *go 기준으로 activity의 ctx의 Done()함수로 cancel이 떨어진다.

  <br>

  ### Activity Task Routing through Task Lists

  - activity는 tasklist를 통해 worker로부터 실행됩니다. tasks list는 큐들이고 worker가 listen 합니다. task lists는 다이나믹하고 가볍습니다. 명시적으로 registered하지 않습니다. 그리고 worker하나당 하나의 tasklist를 가져도 괜찮습니다.
  - 하나의 tasklist에 두 개이상의  activity 를 가지는것이 일반적이고 보통의 경우는 (host routing처럼) 같은 activitytype을 여러 activitylist에서 호출하게 합니다.
  - 하나의 workflow에서 여러 activity task list가 사용되는 예 입니다. 
    - Flow control: worker는 이용가능한 capa가있을때  task list로 부터 consume합니다. 그래서 worker는 요청 급증으로부터 overload되지 않습니다. 만약 activity  request 요청이 worker가 처리하는 속도보다 빠르면  tasklist에서 block될것입니다.
    - Throttling: 각각의 activity worker는 tasklist의 activity를 처리하는 구체적 속도 제한을 가질 수 있습니다. 예비 capacitiy가 있어도 이를 초과하지 않습니다. 또한 global taks list속도 제한도 있습니다. 이 제한은 주어진 작업 리스트에 해당하는 모든 worker에게 모두 적용됩니다. 이것은 activity가 downstrema serviceload를 제한하는데 많이 사용됩니다. 
      - *tasklistname이 다르다는 이야기는 worker가 뜰때 잡는 option이 다르다는 이야기 된다. worker options에는 activity의  throttling을 control할 수 있는 요소들이 있다. 
        - MaxConcurrentActivityExecutionSize,WorkerActivitiesPerSecond등이 그것이다.
    - Deploying a set of activities independently:  host activity와 개별적으로 배포될수있는 activity와 workflow를 생각해 보십시오. activity task를 서비스에게 보내려면 분리된  tasklist 가 필요합니다.(??)
    - Workers with different capabilities: 예를들어 gpu box vs non gpu box 에 떠있는 워커를 생각해보자. 이경우 별도의 task list name이 존재한다면 분리하여 activity execution request를 보낼 수 있다. 
      - *gpu에서 돌 task list name과 그렇지 않은 task list name을 만들어 각 host에서 나누어진 tasklistname을 가지는 worker가 뜨면  routing이 될 것이다.
    - Routing activity to a specific host: 예를 들어 media encoding의 경우 transform과 upload 액티비티는 downalod하는 host의 같이 떠야한다.
    - Routing activity to a specific process: 예를 들어 특정 커다란 데이타를 셋 하고 프로세스에 caching하는 경우, 이 activity는 같은 process에 의존되어진다. 
      - *이런 특수한 작업은 하나의  tasklisname으로 관리되어 질 것이고, 해당 job에 맞는 host에 배포되어질 것이다. tasklistname이 label처럼 이용될 수 있다.
    - Multiple priorities,Versioning...

  

  <br>

  ### Asynchronous Acitivy Completion

  - 기본적으로  activity는 client library language로 함수나 메소드 단위 입니다.  함수가 return 되자마자  activity의 완료를 나타냅니다. 그러나 몇 경우는 asynchronous할 경우도 있습니다.
  - 예를 들면 외부 시스템이 메시지큐를 통한다면 응답은 다른 큐에서 받을 수  있습니다.
  - 이런 경우를 지원하기 위해 cadence는 acitivty의 function이 완료 될 때까지 activity를 완료하지 않는 구현을 지원 한다.
  - 이러한 경우  activity를 완료하기 위해 특별한(분리된) api를 사용해얗만 합니다.
  - 이 api는 다른 언어로 만들어진 다른 워커에서도 사용가능 하며, 다른 프로세스에서도 부를 수 있습니다.
  - *[asynchronous_complete](https://cadenceworkflow.io/docs/07_goclient/12_activity_async_completion)

  <br>

  ### Local Activity

  - 아주 짧게 존재하고, quening되거나 flow control, limiting, routing capabilities하지 않는  activity를 말합니다.
  - 이런 activity를 위해 local activity 라고 불리는 기능을 합니다.
  - local activites는 worker와 같은 프로세스서 workflow에 의해 호출되어 실행됩니다.
  - local activites의 고려사항은  아래와 같습니다. 
    - 길지 않은 수초이내.
    - global한 속도 제한이 필요없다.
    - 워커풀이나,  특정 워커에 라우팅 이 필요없다.
    - 호출하는 workflow와 동일한 binsary로 존재할 수 있다.
  - 가장 큰 이득은 cadence 리소스가 효율적으로 사용되고 일반적인 activity와 비해 낮은 overhead를 가집니다.

