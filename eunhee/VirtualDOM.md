# Virtual DOM
Virtual DOM (VDOM)은 UI의 이상적인 또는 “가상”적인 표현을 메모리에 저장하고 ReactDOM과 같은 라이브러리에 의해 “실제” DOM과 동기화하는 프로그래밍 개념입니다. 

- 실제 DOM oject과 같은 속성을 담고 있는 복사본. 
- 실제 DOM 과는 달리  html 객체에 기반한 JS 객체 형태로 메모리 안에 저장 및 동작하며 실제 렌더링이 일어나지 않음.
- 브라우저에 있는 문서에 직접적으로 접근할 수 없다. Api 가 없음.
### 하는 일 
DOM fragment의 변화를 묶어서 적용한 다음 기존 DOM에 던져주는 과정.-> 이를 자동화 추상화한 것 

어떤 인터렉션에 의해 DOM의 변화가 발생하면 모든 요소들의 스타일을 다시 계산->render tree 가 그때마다 재생성 되어 다시 reflow과정과 repaintg하는 과정을 반복하는 이전을 보완하여 나온것이 Virtual DOM 입니다. Virtual DOM을 이용해 바뀐 부분만 실제 DOM에 부분적으로 적용시킵니다. Real DOM을 다시 렌더링하는 것에 자원 소모가 많아, Virtual DOM을 이용해 비교를 한 후 다른 부분만 렌더링하는 것이 “일반적으로” 성능적인 이점이 있습니다. 
### SPA로 제작된 큰 규모의 웹페이지에 성능이 좋아져..
SPA로 제작된 큰 규모의 웹페이지 에서는 vdom을 사용해 브라우저의 연산 양을 줄여 성능을 개선할 수 있는 반면 정보 제공만 하는 웹페이지라면 인터랙션이 발생하지 않기 때문에 일반 DOM의 성능이 더 좋을 수도 있습니다. 


100개의 노드를 수정하였을 시, 100번의 레이아웃 계산과 100번의 리렌더링이 이루어지는 문제를 Virtual DOM은 이를 "더블 버퍼링"처럼 묶어서 최종적인 변화를 Real DOM에 적용시키는 방법입니다.
중요한 점은 이는 Virtual DOM을 사용하지 않아도 가능하다는 것 입니다. 오히려 이런 최적화 작업을 "잘" 손수했을 때 더욱 빠르다고 합니다.

*더블 버퍼링이란?
더블 버퍼링이란 싱글 버퍼링으로 화면을 그릴 경우 데이터를 저장하는 동안에는 다음 그림의 데이터를 전송할 수 없기 때문에 지우고 그리고 지우고를 반복 할 경우 필연적으로 발생하는 깜빡임 등의 상황을 막기 위해서 사용되는 기법

  ### 왜 VDOM 인지 
DOM fragment 로만 필요한 작업들을 한번에 묶어서 던져줄 수 있지만 해당 과정을 하나하나 작업하지 않고 Vitual DOM을 통해 자동화, 추상화하여,일반적으로 빠른 성능과 함께 위 React 공식문서에서 말하고 있는 선언적 API를 가능하게 하는 것, 상태 중심 UI 개발을 상태 전환에 대해 생각하지 않고 개발할 수 있는 것이 Virtual DOM의 핵심이다.

*선언적 API란 DOM 관리를 Virtual DOM에 위임하여, 컴포넌트가 DOM을 조작할 때 다른 컴포넌트의 DOM 조작 상태를 공유할 필요가 없다는 것이다.


 ```html
<ul id="items">
  <li>Item 1</li>
</ul>
```
  
  ```js
  let domNode = {
    tagName : "ul",
    attributes: {id: "items"},
    children:[
      {
        tagName: "li",
        textContent: "Item 1",
      },
  ],
  };
  ```


### 더블 버퍼링 
https://goidle.github.io/static/258b43ce623e7b6340fc6aed969199ed/374ac/vDOM.png

virtual DOM은 간단히 말해 DOM차원의 '더블 버퍼링 개념'을 말한다.
DOM에 마운트된 current 트리와 Render phase에서 작업 중인 workInProgress 트리로 나뉘어 있습니다. 이 workInProgress 트리는 Commit phase를 지나면 current 트리로 관리됩니다. 이렇듯 더블 버퍼링 형태이기 때문에 리액트는 workInProgress에 작업을 하다가도 언제든지 버리고 처음부터 다시 작업하거나 또는 중지시켰다가 다시 시작하는 등 작업 우선순위에 맞게 유연하게 대처할 수 있기에 사용자 경험을 최우선적으로 고려할 수 있습니다.


두개의 트리를 비교할때 리액트는 비교알고리즘(Diffing Algorithm)을 사용한다.

DOM은 트리 구조이기 때문에 변경된 부분을 감지하기 위해서 트리구조를 비교해야한다.
다 비교하지 않고 필요한 부분만 비교한다.

같은 계층
비슷한 컴포넌트는 트리 내에 동일한 계층에 위치할 것이다. 따라서 이 컴포넌트가 갑자기 부모가 바뀌는 일이 없다고 가정하고 같은 계층에 있는 컴포넌트끼리 비교한다.

key가 있는지 확인
하나의 jsx태그는 자바스크립트 객체로 구성되어있다.
이 객체에는 해당 객체가 리액트의 virtual DOM 요소임을 확인해주는 symbol값과 각각의 virtual DOM을 고유하게 구분하는 key값이 들어가고, map등을 통해 생성된 요소들의 경우 고유한 key가 있다. 리액트는 이 key들의 비교를 통해 리스트의 요소가 추가되었거나 삭제되었을 때 해당 내용을 빠르게 감지하고 반영할 수 있다.



### 동작 과정 
1. 전체 virtual DOM이 업데이트된다.
2. virtual DOM을 업데이트 이전의 시점과 비교한다.
3. 실제 바뀐 부분만 real DOM에 바꾼다.
4. real DOM에서의 변화가 스크린에 그려진다.
   
```jsx
function Component(){
 return (
  <div className="tech-course">
    <h1> Hello Crews!</h1>
  </div>
);
}
```
바벨은 jsx 를 다음과 같이 React.createElement() 호출로 컴파일한다.
```js
function Component(){
 return React.creatElement("div",{
 className: "tech-course"
},React.createElement("h1",null,"Hello Crews!"));
}
```
 React.createElement()로 생성된 react elements는 내부적으로 다음과 같은 객체로 표현될 수 있다.
```js
const element = {
 type: "div",
 props: {
  className: "tech-course",
  children: [
    {
      type: "h1",
      children: "Hello Crew!"
    }
  ]
 }
}
```
### React element
DOM 요소의 가상 버전. 
- 가볍다.
- 상태를 가지지 않는다.
-  불변성을 가진다.

React element는 render에 의해 확장 객체인 fiber로 변환되어 실제 DOM 요소가 된다.
```js
const element = React.creatElement("div",{
 className: "tech-course"
},React.createElement("h1",null,"Hello Crews!"));

React.DOM.render(element, document.getElementByID("root"));
```
부분적으로 변경이 발생했을 때는 모든 VDOM Object 를 업데이트하고 이전 VDOM 스냅샷과 비교하여 정확히 어떤 부분이 바뀌였는 지 검사합니다. 이 과정을 Diffing 알고리즘을 통해 계산됩니다. 

*Diffing
virtual DOM이 업데이트되면 React는 virtual DOM을 업데이트 이전의 virtual DOM 스냅샷과 비교하여 정확히 어떤 Virtual DOM이 바뀌었는지 검사한다. ->리액트의 재조정과정에 해당한다.
```js
const element = {
 type: "h1",
 props: {
  title: "foo",
  children: "hello",
},
};
```
- type === type?  element 속성값만 변한 경우: 속성 값만 업데이트
- type !== type? element 태그, 컴포넌트가 변경된 경우: 해당 노드를 포함한 하위 모든 노드를 제거 unmount 후 새로운 vdom 노드로 대체
### key prop
 자식 노드들이 고유한 key Prop을 부여받아 key값으로 이전트리와 변경 이후의 트리를 비교해 변경된 부분만 정확히 알 수 있음

### 요약
Virtual DOM은 무조건적으로 좋은 것은 아니지만, 대체적으로 좋은 성능을 보여주고 선언적이고 상태 중심 UI 개발이 가능해짐

### 참고
https://joong-sunny.github.io/react/react2/

https://flykimjiwon.tistory.com/138

[https://www.youtube.com/watch?v=PN_WmsgbQCo&t=243s](https://www.youtube.com/watch?v=6rDBqVHSbgM)https://www.youtube.com/watch?v=6rDBqVHSbgM

https://velog.io/@syc765/%EB%A0%8C%EB%8D%94%EB%A7%81-%EC%B5%9C%EC%A0%81%ED%99%94
