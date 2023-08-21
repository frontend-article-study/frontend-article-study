* v16.12.0 버전 함수형 컴포넌트와 브라우저 환경을 기준입니다.
> author: 이은희<br/>date: 2023/08/21

## React는 어떤 과정을 거쳐 jsx element를 화면에 그리나요?
* 대부분의 사람들이 React 프레임워크내에 jsx 문법으로 코드를 작성하기에 jsx 문법으로 코드를 작성했다 가정했습니다.
react가 JSX Element를 화면에 그리는 단계를 크게 3단계 Trigger Phase, Render Phase,Commit Phase로 나눌 수 있습니다.   

### Trigger Phase
화면에 그려야 한다는 Trigger가 발동합니다.
### Render Phase
컴포넌트를 호출 후 react element를 반환합니다. react element는 VDOM의 노드객체인 fiber로 확장되고 VDOM이 재조정됩니다. 
* "랜더링과정"은 정확히 "컴포넌트를 호출 후 VDOM이 재조정되기"까지를 의미합니다.
? 해당 jsx로 작성했다면 바벨을 통한 jsx-> js 과정은 언제 일어나는 지 ? 
-> 렌더시점 이전인 빌딩시점에 일어난다는 이야기를 듣고 바벨 동장원리를 공부하거3
### Commit Phase
VDOM과 DOM을 비교하여 DOM에 삽입합니다.  DOM에 삽입하는 과정을 마운트한다라고 표현합니다.   

DOM 업데이트가 완료가 되면 Commit Phase이 끝나며 DOM을 토대로 브라우저 화면을 그리게 됩니다. 화면에 그려지는 것을 페인트한다라고 표현합니다. 

## React 의 패키지 구조
react는 크게 5가지 패키지로 나눌 수 있습니다. 컴포넌트 정의와 관련있는 react 코어, DOM에 플랫폼과 관련있는 renderer, 비동기로 실행되어야 할 작업들을 Task 형태로 우선순위 스케줄링하는 schedular, VDOM의 재조정일을 하는 등 핵심 패키지인 reconciler입니다.  