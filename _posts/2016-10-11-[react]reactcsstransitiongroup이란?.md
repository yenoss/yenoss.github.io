---
layout: post
title: 2016-10-10-[React]ReactCSSTransitionGroup
---

##ReactCSSTransitionGroup이란?

ReactCSSTransitionGroup은 react-addons 의 css 애니메이션 효과이다.

애니메이션을 포함할 컴포넌트를 래핑하고 생명주기와 관련되 시점에서 **css애니메이션을 트리거**한다.

댜음과 같이 **설치**하고

~~~
npm install react-addons-css-transition-group
~~~

다음과 같이 임포트한다.

~~~
import ReactCSSTransitionGroup from 'react-addons-css-transition-group';
~~~


TransitionGroup은

각각 사용할 **트랜지션명, 시작, 종료** 를 타나내는 속성으로 이루어져있다.

* transitionName
* transitionEnterTimeout(ms)
* transitionLeaveTimeout(ms)

사용할 컴포넌트를 아래와 같이 감싼다

~~~
<ReactCSSTransitionGroup transitionName = "goyouID"
	transitionEnterTimeout={300}
	transitionLeaveTimeout={300}>		
	{test}
</ ReactCSSTransitionGroup>
~~~

card Details 컴포넌트를 호출될때 goyouID에 해당하는 css애니메이션효과가 시작시 및 종료시 0.25sec 만큼 일어난다.


CSS에서는 아래와같이  
enter시 항목이 왼쪽에서 오른쪽으로 나오도록
leave시 항목이 오른쪽에서 왼쪽으로 사라지는 효과를 입힐 수 있다.

~~~
.goyouID-enter{
	opacity:0;
	transform:translate(X(-250px)
}
.goyouID-enter.goyouID-enter-active{
	opacity:1;
	transform:translate(X(0)
	transition:0.3s;
}
.goyouID-leave {
  opacity: 1;
  transform: translateX(0);
}

.goyouID-leave.tt-leave-active {
  opacity: 0;
  transform: translateX(250px);
  transition: 0.3s;
}
~~~




