# 제어 컴포넌트와 비제어 컴포넌트

**제어 컴포넌트(Controlled Components)** 와 **비제어 컴포넌트(Uncontrolled Components)** 는 React에서 폼 요소를 다루는 두 가지 다른 접근 방식을 나타냅니다.

두 컴포넌트를 나누는 기준은 React가 값을 제어하느냐 그렇지 않느냐 입니다. 간단히 말해서 `상태`를 사용하는지 여부로 구분할 수 있습니다.

## 제어 컴포넌트(Controlled Components)

React 공식 문서에서는 제어 컴포넌트를 다음과 같이 정의하고 있습니다.

> HTML에서 `<input>`, `<textarea><select>`와 같은 폼 엘리먼트는 일반적으로 사용자의 입력을 기반으로 자신의 state를 관리하고 업데이트합니다. React에서는 변경할 수 있는 state가 일반적으로 컴포넌트의 state 속성에 유지되며 `setState()`에 의해 업데이트됩니다.
>
> 우리는 React state를 “신뢰 가능한 단일 출처 (single source of truth)“로 만들어 두 요소를 결합할 수 있습니다. 그러면 폼을 렌더링하는 React 컴포넌트는 폼에 발생하는 사용자 입력값을 제어합니다. 이러한 방식으로 **React에 의해 값이 제어되는 입력 폼 엘리먼트를 “제어 컴포넌트 (controlled component)“라고 합니다.**

즉, 제어 컴포넌트는 React 컴포넌트의 상태(state)로 폼 요소를 제어하는 방식입니다. 이는 React가 사용자 입력을 완전히 제어하고 모든 값 및 상태 변경 사항을 React 컴포넌트의 state에 반영하는 방법을 말합니다.

일반적으로 폼을 다룰 때 제어 컴포넌트를 많이 사용하기 때문에 사용 방법은 매우 익숙합니다.

- 폼 요소의 value는 state의 초깃값이나 state의 업데이트로 설정됩니다. → `<input value={state} />`
- 이벤트 핸들러 함수를 사용해 폼 요소의 변경 사항을 감지하여 state를 업데이트 합니다. → `<input onChange={e ⇒ setState(e.target.value)} value={state} />`

다음 코드는 React에 의해 값이 제어되고 있는 제어 컴포넌트의 예시 코드입니다.

```jsx
import React, { useState } from 'react';

function ControlledComponent() {
  const [inputValue, setInputValue] = useState('');

  const handleInputChange = (event) => {
    setInputValue(event.target.value);
  };

  return <input type="text" value={inputValue} onChange={handleInputChange} />;
}

export default ControlledComponent;
```

`inputValue`라는 상태를 통해 input의 value가 관리되고 있으며, onChange 핸들러를 통해 입력과 상태가 동기화되고 있습니다. 따라서 사용자가 입력한 값을 사용하고 싶다면 `inputValue`를 사용하면 됩니다.

또한, 제어 컴포넌트는 value 속성 값과 state를 결합하여 state를 신뢰 가능한 단일 출처로 사용합니다. 즉, 사용자가 입력한 값은 inputValue라는 하나의 자원으로 관리되기 때문에 SSOT 개념에 부합합니다.

> SSOT(Single Source Of Truth, 단일 진실 공급원)란 하나의 상태는 한 곳에만 존재해야 한다는 개념입니다.

### 제어 컴포넌트 정리

이렇게 제어 컴포넌트는 리액트가 값을 관리함으로써 state와 input의 value 값이 항상 일치함을 보장합니다.

또한, state의 특성 상 변경되면 **리렌더링**이 발생하기 때문에 **입력 때마다 즉각적인 유효성 검사**를 하는 경우나, 값이 **입력됐을 때에만 submit 버튼을 활성화**하는 등 **실시간적인 처리**가 들어가야할 때 유용합니다.

## 비제어 컴포넌트(Uncontrolled Components)

비제어 컴포넌트는 React 컴포넌트의 상태에 의존하지 않고 DOM 요소를 직접 다루는 방식을 가리킵니다. React가 폼의 입력을 제어하지 않기 때문에 전통적인 HTML Form Element 사용 방식과 동일합니다.

비제어 컴포넌트를 사용하기 위해서는 DOM에 직접 접근해야 하므로 useRef를 사용합니다.

- useRef 훅을 사용해 ref 객체를 생성합니다. → `const ref = useRef(null);`
- ref를 input 요소에 바인딩합니다. → `<input ref={ref} />`

다음 코드는 React가 값을 제어하지 않는 비제어 컴포넌트의 예시 코드입니다.

```jsx
import React, { useRef } from 'react';

function UncontrolledComponent() {
  const inputRef = useRef(null);

  const handleButtonClick = () => {
    alert('Input value is: ' + inputRef.current.value);
  };

  return (
    <div>
      <input type="text" ref={inputRef} />
      <button onClick={handleButtonClick}>Get Value</button>
    </div>
  );
}

export default UncontrolledComponent;
```

input의 값을 사용하려면 특정 이벤트가 발생했을 때 `DOM`에 접근함으로써 입력값을 참조할 수 있습니다.

이때 input의 value 속성 값이 신뢰 가능한 단일 출처가 됩니다.

또한, ref는 변경이 발생해도 리렌더링을 일으키지 않기 때문에 제어 컴포넌트와 비교했을 때 성능 상의 이점이 있습니다.

### 비제어 컴포넌트 정리

비제어 컴포넌트는 React가 입력을 제어하지 않기 때문에 DOM 요소를 직접 다뤄 입력 값을 참조할 수 있습니다.

또한, 제어 컴포넌트와는 다르게 입력이 발생할 때마다 리렌더링을 일으키지 않아 성능 상의 이점이 있고 모든 폼 요소를 state와 연결짓지(+핸들러를 작성할 필요도 없음) 않아도 되기 때문에 조금이나마 간결한 코드를 작성할 수 있습니다.

## 그럼 제어를 쓸까, 비제어를 쓸까?

공식 문서에서는 **제어 컴포넌트**를 사용하는 것을 추천하고 있습니다.

비제어 컴포넌트는 제어 컴포넌트보다 간단할 수 있지만, 제어 컴포넌트를 사용하면 상태와 데이터의 흐름을 활용할 수 있다는 장점이 있습니다.

두 방식 모두 각각의 장점이 있으므로 요구사항을 바탕으로 따져보고 결정하면 됩니다.
