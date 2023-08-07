# Referencing Values with Refs 
#### 작성 : 손준석

- 참고 사이트
   - https://react.dev/learn/referencing-values-with-refs
   - https://react-ko.dev/learn/referencing-values-with-refs
   - https://linguinecode.com/post/how-to-use-react-useref-with-typescript
   - https://driip.me/7126d5d5-1937-44a8-98ed-f9065a7c35b5
     

## Ref
- ref를 사용하면, 컴포넌트가 특정 데이터를 저장합니다. 하지만 state와 달리 값이 변동 될시에 **새로운 렌더링을 촉발하지 않습니다**.

## 컴포넌트에 Ref 값 추가하기 
`useRef` 혹을 통해서 컴포넌트에 Ref를 추가합니다. 컴포넌트 내부에서 useRef 훅을 호출하고 참조할 값을 인자로 전달할 수 있습니다.
`useRef` 는 `current` key를 가진 객체를 반환합니다. 전달한 인자 값은 current의 값으로 할당이 됩니다. 현재 ref는 0을 가리키고 있지만, state와 마찬가지로 문자열, 숫자, 객체, 함수 등 어떤 값이든
참조 할 수 있습니다. 
```jsx
import { useRef } from 'react';

export default function Component(){
  // 참조할 초기 값으로 0을 전달
  const ref = useRef(0);
  console.log(ref)
}
```
<img width="377" alt="image" src="https://github.com/kd02109/react-article-study/assets/57277708/ea29489a-6a87-4833-90d2-4a264baafe13">

`ref.current`를 통해 ref에 할당된 현재 값에 엑세스 할 수 있습니다. 이 값은 **읽기와 쓰기** 모두 가능합니다. 의도적으로 변이(mutation) 가능하도록 만들었으며 React의 단방향 데이터 바인딩을
탈출할 수 있는 요소 중 하나입니다. 따라서 ref는 다음과 같은 경우에 사용이 됩니다. 

**1. DOM 조작시 DOM 객체 저장 (이는 Manipulating the DOM with Refs를 참고하세요)**

**2. 렌더링(JSX)에 사용되지 않는 값 관리.** 

**3. timeout ID 저장(setInterval, setTimeout)**

`ref`는 `useEffect`와 마찬가지로 탈출구 역할을 합니다. ref는 외부 시스템, 브라우저 API 작업시 유용합니다. 더불어 **렌더링 중에는 `ref.current`를 읽거나 쓰지 마세요.**
렌더링 중에 일부 정보가 필요한 경우, state 사용을 추천합니다. React는 ref.current 값이 언제 변경되는지 알지 못합니다.(참조하는 객체가 동일하기 때문입니다.) 따라서 렌더링 중에 읽어도 컴포넌트의 동작을 예측하기 어렵습니다.
(유일한 예외는 첫 번째 렌더링 중에 ref를 한 번만 설정하는 경우입니다.)
```jsx
if (!ref.current) ref.current = new Thing()
```

### 예제 stopwatch
```jsx
import { useState, useRef } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);
  const intervalRef = useRef(null);

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalRef.current);
    // interval ID는 렌더링에 사용되지 않으므로 ref에 보관할 수 있습니다.
    intervalRef.current = setInterval(() => {
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>
        Start
      </button>
      <button onClick={handleStop}>
        Stop
      </button>
    </>
  );
}
```
---

## Ref와 state의 차이점
| ref | state |
| --- | --- |
| `useRef`는 `{ current: initialValue }`을 반환  | `useState`는 state 변수의 현재값과 state 설정자함수를 반환 `([value, setValue])` |
| 변경 시 리렌더링을 촉발하지 않음 | 변경 시 리렌더링을 촉발함 |
| 렌더링 프로세스 외부에서 current 값을 수정하고 업데이트할 수 있음(mutable) | state setting 함수를 사용하여 state 변수를 수정해 리렌더링을 대기열에 추가해야함(immutable) |
| 렌더링 중에는 current 값을 읽거나 쓰지 않아야 합니다. | 언제든지 state를 읽을 수 있습니다. | 

---

> **ref는 내부에서 어떻게 작동하는가?**
>
>  useState와 useRef는 모두 React에서 제공합니다. useRef는 useState위에 구현될 수 있습니다. React에서 useRef를 다음과 같이 구현한다고 상상할 수 있습니다.
> ```jsx
> //  react 내부
> function useRef(initialValue) {
>   const [ref, unused] = useState({ current: initialValue });
>   return ref;
> }
> ```
>  첫 번째 렌더링 중에 `useRef`는 `{current: initialValue}`를 반환합니다. 이는 React에 의해 저장되므로, 다음 렌더링 중에 동일한 객체가 반환 됩니다. useRef는 항상 동일한 객체를 참조하기에
>  렌더링을 발생시키지 않습니다. 즉 ref는 set 함수가 없는 일반 state 변수입니다.

# Using ref with TypeScript
React와 TypeScript를 함께 사용하는 환경에서 `useRef`를 사용하는 경우 예상치 못한 애러를 마주치게 됩니다. 특히 다음과 같은 애러를 만나는 경우가 많습니다. JavaScript에서는 문제없이 동작하지만
TypeScript에서 오류를 발생시키고 있는 상황입니다.

<img width="276" alt="image" src="https://github.com/kd02109/react-article-study/assets/57277708/33eff1f0-1b4e-43d8-bd7d-74dcb1f53181">

```bash
'inputRef' is possibly 'null'.ts(18047)
const inputRef: null
```
ref가 DOM 객체를 참조할 때의 값을 typescript가 추론을 하지 못하고 있습니다. 이는 react에서 useRef의 type 지정을 3가지로 하고 있기 때문입니다. 

```ts
// index.d.ts
function useRef<T>(initialValue: T): MutableRefObject<T>;
function useRef<T>(initialValue: T|null): RefObject<T>;
function useRef<T = undefined>(): MutableRefObject<T | undefined>;

interface MutableRefObject<T> {
    current: T;
}

interface RefObject<T> {
    readonly current: T | null;
}

```
위에서 사용된 ref값이 추론된 결과를 본다면, `const domRef: React.MutableRefObject<null>`로 표기가 되고 있습니다. 즉 Ref를 정의한 3 가지중 첫 번째 경우에 해당합니다. 
저희는 초기 값으로 `null`을 전달했기 때문에, JSX에 ref를 연결히여 DOM 객체를 참조하여도 `function useRef<T>(initialValue: T): MutableRefObject<T>` 타입 선언에 따라 
typeScript가 변경된 type을 추론하지 못하고 있습니다. 단지 컴포넌트 내부 변수에서 값을 저장하기 위한 용도로 사용한다면 `MutableRefObject<T>`로 타입을 추론하는 것은 문제가 없습니다. 
문제는 ref를 통해 DOM 객체를 참조할 때 발생합니다.

저희는 TypeScript가 `MutableRefObjext<T>`가 아닌 `RefObject<T>`로 useRef의 반환값을 추론하기를 원합니다. 이를 위해서 제네릭을 통해 useRef 초기 값의 타입을 typescript에게 알려줄 필요가 있습니다.
이후 DOM 객체에 접근시 type 가드 혹은 optional chaining(`?.`)을 활용해 줍니다. 
```jsx
import { useEffect, useRef } from 'react'
import './App.css'

function App() {
  // const domRef: React.RefObject<HTMLInputElement>
  const domRef = useRef<HTMLInputElement>(null);
  console.log(domRef.current)

  useEffect(()=>{
    // const inputRef: HTMLInputElement | null  
    const inputRef = domRef.current
    console.log(inputRef)
    inputRef?.focus()
  }, [])

  return (
    <>
      <input ref={domRef} type='text'/>
    </>
  )
}

export default App

```

만약 초기 ref 값 설정시 인자에 초기 값을 전달하지 않는다면, ref type 지정 중 세 번째에 해당이 됩니다.
```jsx
// const domRef: React.MutableRefObject<HTMLInputElement | undefined>
const domRef = useRef<HTMLInputElement>();

```

<img width="277" alt="image" src="https://github.com/kd02109/react-article-study/assets/57277708/be5043ed-e954-4d96-a6b5-6b6fb52c8524">


```bash
Type 'MutableRefObject<HTMLInputElement | undefined>' is not assignable to type 'LegacyRef<HTMLInputElement> | undefined'.
Type 'MutableRefObject<HTMLInputElement | undefined>' is not assignable to type 'RefObject<HTMLInputElement>'.
Types of property 'current' are incompatible.
Type 'HTMLInputElement | undefined' is not assignable to type 'HTMLInputElement | null'.
Type 'undefined' is not assignable to type 'HTMLInputElement | null'.ts(2322)
```
즉 DOM 객체로 활용되어야 하는 ref의 반환 타입은 `RefObject<HTMLInputElement>` 여야 하는데 현재의 ref 타입 값은 `MutableRefObject<HTMLInputElement | undefined>`입니다.
따라서 jsx에서 ref값 참조시에 애러가 발생합니다. 

따라서 useRef는 사용 방식에 따라서 type 지정 방식을 달리 해야 합니다. 

- **컴포넌트 내부에서 데이터를 저장하는 용도로 사용된다면 별도의 type 처리를 필요로 하지 않습니다.** 기본적으로 typeScript가 추론하는 `MutableRefObject<T>`를 사용합니다.
- **ref를 통해 DOM 객체를 활용하기를 원한다면 별도의 type 설정과 반드시 초기값으로 ref에 null을 할당합니다.** 이를 통해 typeScript는 `useRef`가 반환한 타입이 `RefObject<T>`임을 알 수 있습니다.

