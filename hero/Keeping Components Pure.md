# Keeping Components Pure

<br>

컴포넌트를 엄격하게 순수 함수로만 작성하면 규모가 커질수록 버그와 예상치 못한 동작을 없앨 수 있습니다.

순수함수란?
- 자신의 일만 합니다. 호출되기 전에 존재했던 객체나 변수를 변경하지 않습니다.
- 어떤 입력이 주어지면 항상 같은 결과를 반환합니다.
```js
const y = (x) => {
  return 2 * x;
}
// x가 1이면 y는 2, x가 2면 y는 4
// y는 항상 같은 결과를 반환하는 함수
```

<br>

## Side Effects

React는 작성되는 모든 컴포넌트가 순수 함수라고 가정합니다.
작성한 React 컴포넌트는 동일한 입력이 주어졌을 때 항상 같은 JSX를 반환해야 합니다.
또한 외부의 존재하는 객체나 변수를 변경하면 안됩니다. 이것은 컴포넌트를 순수하지 않게 할 수 있기 때문입니다.

```jsx
let number = 0;

function Introduce({ name }) {
  number += 1;
  return (
    <div>{number}. Hello! My name is {name}</div>
  );
}

export default function App() {
  return (
    <section>
      <Introduce name={'Kim'} />
      <Introduce name={'Park'} />
      <Introduce name={'Lee'} />
    </section>
  );
}
```
```
// 결과
2. Hello! My name is Kim
4. Hello! My name is Park
6. Hello! My name is Lee
```

제가 원하는 결과는 1, 2, 3 순서대로 나오는 것이었지만, 실제로는 2, 4, 6으로 나왔습니다.

이처럼 외부 객체나 변수를 사용해서 컴포넌트를 순수하지 않게 만들면 예상하지 못한 동작으로 원하는 결과를 얻을 수 없게 됩니다.

이 문제의 해결법은 prop으로 number를 전달하면 됩니다.

```jsx
function Introduce({ number, name }) {
  return (
    <div>{number}. Hello! My name is {name}</div>
  );
}

export default function App() {
  return (
    <section>
      <Introduce number={1} name={'Kim'} />
      <Introduce number={2} name={'Park'} />
      <Introduce number={3} name={'Lee'} />
    </section>
  );
}
```

이제 Introduce에 props로 number와 name을 전달하면 항상 같은 JSX를 반환할 수 있게 됩니다.

렌더링은 언제든지 발생될 수 있기 때문에, 컴포넌트들은 렌더링 순서에 의존하면 안되고 어디서 호출하든 동일한 결과를 내야합니다. 컴포넌트는 렌더링 중에 다른 컴포넌트에게 의존하지 않게 독립적으로 구성해야 합니다.

<br>
<br>

## Local Mutation

Side Effect 부분에 있던 예시에서 컴포넌트가 렌더링되는 동안 기존의 변수를 변경했기 때문에 예상하지 못한 동작이 일어났습니다. 이것을 변이(mutation)라고 부른다고도 합니다. 순수 함수는 범위를 벗어난 변수, 호출 전에 생성된 객체 등을 변이하지 않습니다.

```jsx
function Introduce({ number, name }) {
  return (
    <div>{number}. Hello! My name is {name}</div>
  );
}

export default function App() {
  const students = [];
  const names = ['kim', 'park', 'lee'];
  for (let i=1; i<=3; i++) {
    students.push(<Introduce number={i} name={names[i-1]} />);
  }
  return students;
}
```

하지만 위의 예처럼 컴포넌트를 렌더링하는 동안 생성된, 즉, 컴포넌트 내부에서 생성된 객체나 변수를 변경하는 것은 괜찮습니다. App 컴포넌트에서 일어난 변경을 다른 컴포넌트들은 모르기 때문입니다. 이것을 지역 변이(local mutation)이라고 부릅니다.

<br>
<br>

## Where you can cause side effects

함수형 프로그래밍은 순수성에 크게 의존하지만, 사이드 이펙트가 필요할 때가 있습니다.

사이드 이펙트는 화면 업데이트, 애니메이션, 데이터 변경과 같은 변경을 말합니다.
이것은 렌더링 중에 일어나는 것이 아니라 부수적으로 일어나는 일입니다.

React에서 사이드 이펙트는 보통 이벤트 핸들러입니다. 이벤트 핸들러는 사용자가 어떤 동작을 했을 때 React가 실행하는 함수입니다. 이벤트 핸들러는 컴포넌트 내부에 정의되어 있어도 렌더링 중에는 실행되지 않기 때문에 순수할 필요가 없습니다. 

만약 모든 방법을 찾아봤는데도 사이드 이펙트에 맞는 이벤트 핸들러를 찾을 수 없다면, useEffect 훅을 사용할 수 있습니다. 하지만 useEffect는 최후의 수단으로 사용해야 합니다.


<br>
<br>

## 리액트가 컴포넌트를 순수 함수로 유지하려는 이유?

순수 함수는 동일한 입력에 대해 항상 동일한 결과를 반환하기 때문에 

- 하나의 컴포넌트로 많은 사용자 요청을 처리할 수 있습니다.
- 캐싱하기에 안전합니다. 그래서 입력이 변경되지 않았다면 메모를 통해 렌더링을 건너뛰어서 성능을 향상시킬 수 있습니다.
- 테스트 하기 쉽습니다.
- 코드의 의도가 명확하기 때문에 유지보수성이 높아집니다.

모든 React의 기능은 순수성의 이점을 활용하기 때문에, 컴포넌트를 항상 순수하게 유지해서 React를 최대한 활용할 수 있습니다.

### 참고
- https://react.dev/learn/keeping-components-pure
- https://react-ko.dev/learn/keeping-components-pure

작성: 김영웅