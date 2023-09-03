# Virtual DOM
Virtual DOM (VDOM)은 UI의 이상적인 또는 “가상”적인 표현을 메모리에 저장하고 ReactDOM과 같은 라이브러리에 의해 “실제” DOM과 동기화하는 프로그래밍 개념입니다. 

- 실제 DOM oject과 같은 속성을 담고 있는 복사본. 
- 실제 DOM 과는 달리  html 객체에 기반한 JS 객체 형태로 메모리 안에 저장 및 동작하며 실제 렌더링이 일어나지 않음.
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
- 브라우저에 있는 문서에 직접적으로 접근할 수 없다. Api 가 없음.

어떤 인터렉션에 의해 DOM의 변화가 발생하면 모든 요소들의 스타일을 다시 계산->render tree 가 그때마다 재생성 되어 다시 reflow과정과 repaintg하는 과정을 반복하는 이전을 보완하여 나온것이 Virtual DOM 입니다. Virtual DOM을 이용해 바뀐 부분만 실제 DOM에 부분적으로 적용시킵니다. Real DOM을 다시 렌더링하는 것에 자원 소모가 많아, Virtual DOM을 이용해 비교를 한 후 다른 부분만 렌더링하는 것이 “일반적으로” 성능적인 이점이 있습니다.
100개의 노드를 수정하였을 시, 100번의 레이아웃 계산과 100번의 리렌더링이 이루어지는 문제를 Virtual DOM은 이를 "더블 버퍼링"처럼 묶어서 최종적인 변화를 Real DOM에 적용시키는 방법입니다.
중요한 점은 이는 Virtual DOM을 사용하지 않아도 가능하다는 것 입니다. 오히려 이런 최적화 작업을 "잘" 손수했을 때 더욱 빠르다고 합니다.

### 하는 일 
DOM fragment의 변화를 묶어서 적용한 다음 기존 DOM에 던져주는 과정.-> 이를 자동화 추상화한 것 

### 왜 VDOM 인지 
DOM fragment 로만 필요한 작업들을 한번에 묶어서 던져줄 수 있지만 해당 과정을 하나하나 작업하지 않고 Vitual DOM을 통해 자동화, 추상화하여,일반적으로 빠른 성능과 함께 위 React 공식문서에서 말하고 있는 선언적 API를 가능하게 하는 것, 상태 중심 UI 개발을 상태 전환에 대해 생각하지 않고 개발할 수 있는 것이 Virtual DOM의 핵심이다.

*선언적 API란 DOM 관리를 Virtual DOM에 위임하여, 컴포넌트가 DOM을 조작할 때 다른 컴포넌트의 DOM 조작 상태를 공유할 필요가 없다는 것이다.


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

- element 속성값만 변한 경우: 속성 값만 업데이트
- element 태그, 컴포넌트가 변경된 경우: 해당 노드를 포함한 하위 모든 노드를 제거 unmount 후 새로운 vdom 노드로 대체


### 요약
Virtual DOM은 무조건적으로 좋은 것은 아니지만, 대체적으로 좋은 성능을 보여주고 선언적이고 상태 중심 UI 개발이 가능해짐

### 참고
https://joong-sunny.github.io/react/react2/

https://flykimjiwon.tistory.com/138

https://www.youtube.com/watch?v=PN_WmsgbQCo&t=243s
