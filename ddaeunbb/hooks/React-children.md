# React Children API에 대하여 알아보기

리액트 베타 공식문서에서 React children 관련된 API들이 모두 레거시처리가 되어있지만 디자인 시스템인 합성컴포넌트에 활용되는 부분이 있기 때문에, 리액트 children에 대해서 알아보고자 합니다. (+ 지식부족..)
<br>

### 📌 먼저 리액트에서 Children이라는 API가 왜 필요할까요?

우선, 컴포넌트의 `children prop`과 `Children API`는 이름이 비슷해서 헷갈리기 쉽습니다. 소문자로 시작하는 `children`은 우리가 흔히 알고 있는 `props`이라는 컴포넌트 함수의 매개 변수가 가지고 있는 하나의 속성이며 이를 통해 컴포넌트의 자식이 넘어오게 됩니다. 대문자로 시작하는 `Children`는 React에서 children prop을 효과적으로 다룰 수 있도록 제공하는 API 입니다.
<br>

### 📌 children Prop은 무엇이든 될 수 있다.

저는 잘 모르지만..허허 리액트를 많이 활용하면서, `children`은 다소 까다로운 편이라고 합니다. 그래서 API가 필요하게 된 것인데요. 왜 까다로울까요?
바로 `children Prop`이 넘어오는 것의 형태를 예상할 수 없기 때문입니다.

- 예를 들어, 아래와 같이 문자열로 넘어 올 수 있습니다.

```jsx
<OurComponent>Text</OurComponent>
```

- 또한 숫자로도 넘어 올 수 있습니다.

```jsx
<OurComponent>{100}</OurComponent>
```

- HTML요소나 React요소가 넘어올 수 있습니다.

```jsx
// HTML 요소
<OurComponent>
  <input />
</OurComponent>

// React 요소
<OurComponent>
  <Button />
</OurComponent>
```

- 함수가 넘어올 수도 있습니다.

```jsx
<OurComponent>{(data) => <span>{data}</span>}</OurComponent>
```

- 아무 것도 넘어오지 않을 수도 있습니다. (null)

```jsx
<OurComponent />
```

- 심지어 위 내용들의 조합으로 넘어올 수 있습니다. (섞여서)
  <br>

이처럼, React 컴포넌트의 자식으로 어떤것이 넘어올지 예상할 수 없기 때문에 children prop을 상대로 직접 프로그래밍하게 되면 버그가 발생하기 쉬워집니다. 따라서 React에서 Children이라는 별도의 API를 제공하는 이유이며, 우리는 Children API를 통해서 children prop을 좀 더 안전하게 다룰 수 있습니다.
(🥹타입스크립트를 쓰지 않는다는 가정하에...)

### 📌 Children API 접근 방법

```jsx
import React from "react";

function ReactChildren({ children }) {
  console.log(React.Children);
  return <>ReactChildren</>;
}
```

위와 같이 `React.Children`을 불러 콘솔을 찍게 되면, 사용할 수 있는 함수 5가지를 확인할 수 있습니다. (5가지의 API)

```javascript
{map: ƒ, forEach: ƒ, count: ƒ, toArray: ƒ, only: ƒ}
```

<br>

### 📌 Children.map()

`map`은 자바스크립트의 map과 매우 흡사한 API입니다.

- 첫번째 인자: `children prop`
- 두번째 인자: `(child, index)=>unknown`

첫번째 인자로는 children prop을, 두번째 인자로는 콜백함수를 받습니다. 콜백함수는 각 자식과, 인덱스가 인자로 주어지기 때문에 이 콜백함수를 활용해서 각 자식을 다른 형태로 변환할 수 있습니다.

```typescript
map(children: unknown, fn: (child: unknown, index: number) => unknown): unknown[]
```

예를 들어, 홀수번째 자식의 글씨를 굵게하고 짝수번째 자식에 밑줄을 그어주는 컴포넌트를 만든다고 가정해볼까요?

```jsx
function Map({ children }) {
  return React.Children.map(children, (child, i) =>
    i % 2 === 0 ? <b>{child}</b> : <u>{child}</u>
  );
}
```

아래를 예시로 `children prop`을 넘긴다면, AAA, CCC, EEE는 굵게 표시가 되고, 나머지는 밑줄이 그어지게 될 것입니다.

```jsx
<Map>
  <span>AAA</span>
  <span>BBB</span>
  <span>CCC</span>
  <span>DDD</span>
  <span>EEE</span>
  <span>FFF</span>
</Map>
```

<br>

### 🧐 그냥 children prop의 map을 사용하면 안될까?

이제 위의 예시를 보았다면 그냥 `children prop`을 상대로 바로 `map()`을 호출하면 안될까? 라는 생각이 들 수도 있습니다. 하지만 그렇게 호출을 하게 되면, 우리는 children prop이 무엇인지 알 수 없습니다.

운좋게 배열이 되면 `map` 함수를 사용할 수 있지만 만약에 null이나 배열이 아닌 다른 것이 들어오게 된다면 `map`을 사용할 수 없게 됩니다. 따라서 `React.Children` API를 사용해서 안전하게 항상 배열을 다루는 것처럼 사용하는 것이 좋습니다.
<br>

### 📌 Children.forEach()

Children API의 `forEach()` 함수는 방금 살펴본 `map()` 함수와 거의 비슷하지만 두 번째 인자로 넘어가는 콜백 함수가 아무것도 반환하지 않는다는 차이가 있습니다.

```typescript
forEach(children: unknown, fn: (child: unknown, index: number) => void): unknown[]
```

예를 들어, 모든 자식이 담고 있는 문자열의 길이를 더해서 표시해주는 컴포넌트를 작성해보겠습니다.

```jsx
function ForEach({ children }) {
  let count = 0;
  React.Children.forEach(children, (child) => {
    count += child.length;
  });
  return (
    <>
      {children}
      {`(총 글자수: ${count})`}
    </>
  );
}
```

위의 `ForEach`함수를 아래의 예시로 활용하게 된다면, `총 글자수는: 18`이라는 메시지가 보이게 될 것입니다.

```jsx
<ForEach>
  {"AAA"}
  {"BBB"}
  {"CCC"}
  {"DDD"}
  {"EEE"}
  {"FFF"}
</ForEach>
```

<br>
### 📌 Children.count()
Children API의 `count()` 함수는 자식의 개수를 구할 때 사용하는데요. 인자로 `children prop`` 하나만 받으며, 해당 컴포넌트로 넘어온 자식이 몇개인지를 반환합니다.

children prop이 반드시 배열이 아니더라도 `Children.count()` 함수는 안전하게 작동합니다. 즉, `Children.count() 함수는 자식이 null이면 0을 반환하고, 자식이 하나 밖에 없으면 1을 반환합니다.

예를 들어, 자식의 개수를 화면에 추가로 보여주는 컴포넌트를 작성해보겠습니다.

```jsx
function Count({ children }) {
  const count = React.Children.count(children);
  return (
    <>
      {children}
      {`(총 자식수: ${count})`}
    </>
  );
}
```

작성한 Count 컴포넌트의 자식으로 다음과 같은 6개의 `<span>` 요소를 사용해보면, `총 자식수: 6`라는 메시지가 보일 것입니다.

```jsx
<Count>
  <span>AAA</span>
  <span>BBB</span>
  <span>CCC</span>
  <span>DDD</span>
  <span>EEE</span>
  <span>FFF</span>
</Count>
```

<br>

### 📌 Children.toArray()

Children API의 `toArray()`함수는 자식을 일반 자바스크립트 배열로 변환해주는데요. 자식을 상대로`join( )`, `reverse( )`, `sort( )`, `filter( )`, `reduce( )`와 같은 자바스크립트 배열에서 제공하는 함수를 사용하고 싶을 때 유용합니다.

예를 들어, 홀수 번째 자식만 화면에 그려주는 컴포넌트를 작성해보겠습니다.
