Virtual DOM (VDOM)은 UI의 이상적인 또는 “가상”적인 표현을 메모리에 저장하고 ReactDOM과 같은 라이브러리에 의해 “실제” DOM과 동기화하는 프로그래밍 개념입니다. 

- 실제 DOM과 같은 내용을 담고 있는 복사본. 
- 실제 DOM 과는 달리  html 객체에 기반한 JS 객체 형태로 메모리 안에 저장
- 브라우저에 있는 문서에 직접적으로 접근할 수 없다. Api 가 없음.


Real DOM을 다시 렌더링하는 것에 자원 소모가 많아, Virtual DOM을 이용해 비교를 한 후 다른 부분만 렌더링하는 것이 “일반적으로” 성능적인 이점이 있습니다.
100개의 노드를 수정하였을 시, 100번의 레이아웃 계산과 100번의 리렌더링이 이루어지는 문제를 Virtual DOM은 이를 "더블 버퍼링"처럼 묶어서 최종적인 변화를 Real DOM에 적용시키는 방법입니다.
중요한 점은 이는 Virtual DOM을 사용하지 않아도 가능하다는 것 입니다. 오히려 이런 최적화 작업을 "잘" 손수했을 때 더욱 빠르다고 합니다.

### 하는 일 
DOM fragment의 변화를 묶어서 적용한 다음 기존 DOM에 던져주는 과정.-> 이를 자동화 추상화한 것 

### 왜 VDOM 인지 
DOM fragment 로만 필요한 작업들을 한번에 묶어서 던져줄 수 있지만 해당 과정을 하나하나 작업하지 않고 Vitual DOM을 통해 자동화, 추상화하여,일반적으로 빠른 성능과 함께 위 React 공식문서에서 말하고 있는 선언적 API를 가능하게 하는 것, 상태 중심 UI 개발을 상태 전환에 대해 생각하지 않고 개발할 수 있는 것이 Virtual DOM의 핵심이다.

*선언적 API란 DOM 관리를 Virtual DOM에 위임하여, 컴포넌트가 DOM을 조작할 때 다른 컴포넌트의 DOM 조작 상태를 공유할 필요가 없다는 것이다.



### 요약
Virtual DOM은 무조건적으로 좋은 것은 아니지만, 대체적으로 좋은 성능을 보여주고 선언적이고 상태 중심 UI 개발이 가능해짐

### 참고
https://joong-sunny.github.io/react/react2/
https://flykimjiwon.tistory.com/138
https://www.youtube.com/watch?v=PN_WmsgbQCo&t=243s
