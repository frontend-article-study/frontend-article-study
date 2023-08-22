---
description: 여러 state 업데이트를 큐에 담기
---

# Queueing a Series of State Updates

| <p>What “batching” is and how React uses it to process multiple state updates<br>(일괄처리(batching)이란 무엇이며 React가 여러 state 업데이트를 처리하는 방법)</p> |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| <p>How to apply several updates to the same state variable in a row<br>(동일한 state 변수에서 여러 업데이트를 적용하는 방법</p>)                                   |

### React는 state를 업데이트 하기 전에 이벤트 핸들러의 모든 코드가 실행될 때까지 기다린다.

```jsx
setNumber(0 + 1);
setNumber(0 + 1);
setNumber(0 + 1);
```

리렌더링이 일어나는 시점은? 👉🏻 setNumber가 모두 호출된 이후에 리렌더링이 일어난다.

> _리액트 === 웨이터라고 생각하자. 웨이터는 주문마다 주방에 전달하는 것이 아니라 모든 주문을 다 받고 주방에 전달한다._

### 렌더링 전에 동일한 state 변수를 여러 번 업데이트하기

다음 렌더링 전에 동일한 state를 여러 번 업데이트 하고 싶다면 값을 전달하는 것이 아니라 **함수**를 전달한다.

```jsx
export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber((n) => n + 1);
          setNumber((n) => n + 1);
          setNumber((n) => n + 1);
        }}
      >
        +3
      </button>
    </>
  );
}
```

- 여기서 `n => n + 1` 는 updater function이라고 부른다.
- React는 **이벤트 핸들러의 다른 코드가 모두 실행된 후에** 함수를 큐에 넣는다.
- 다음 렌더링 중에 React는 큐를 순회하여 최종 업데이트된 state를 제공한다.

### state를 교체한 후 업데이트했을 때의 상황

만약 state의 값을 교체한 후 updater function을 실행시키면 어떻게될까?

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber(number + 5);
          setNumber((n) => n + 1);
        }}
      >
        Increase the number
      </button>
    </>
  );
}
```

1. `setNumber(number + 5)` : number는 0이므로 setNumber(0 + 5)이다. React는 큐에 "5로 바꾸기"를 추가한다.
2. `setNumber(n => n + 1) : n => n + 1` 는 업데이터 함수이다. React는 해당 함수를 큐에 추가한다.
3. 다음 렌더링동안 React는 state 큐를 순회
   - 5로 바꾸기
   - n => n + 1, 현재 n은 5이므로 5 + 1 = 6
4. React는 6을 최종 결과로 저장하고 useState에서 반환

### 업데이트 후 state를 바꿨을 때의 상황

버튼을 클릭했을 때 number는 어떻게 될까?

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber(number + 5);
          setNumber((n) => n + 1);
          setNumber(42);
        }}
      >
        Increase the number
      </button>
    </>
  );
}
```

- React는 42를 최종 결과로 저장하고 useState에서 반환

### Fix a request counter

사용자가 “구매” 버튼을 누를 때마다 “진행중” 카운터가 1씩 증가해야 합니다. 3초 후에는 “진행중” 카운터가 감소하고 “완료됨” 카운터가 증가해야 합니다. 그런데 “보류 중” 카운터가 의도대로 작동하지 않고 있습니다. “구매”를 누르면 -1로 감소합니다(그럴 수 없습니다!). 그리고 빠르게 두 번 클릭하면 두 카운터가 모두 예측할 수 없게 작동하는 것 같습니다.

왜 이런 일이 발생할까요? 두 카운터를 모두 수정하세요.

```jsx
import { useState } from 'react';

export default function RequestTracker() {
  const [pending, setPending] = useState(0);
  const [completed, setCompleted] = useState(0);

  async function handleClick() {
    setPending(pending + 1);
    await delay(3000);
    setPending(pending - 1);
    setCompleted(completed + 1);
  }

  return (
    <>
      <h3>Pending: {pending}</h3>
      <h3>Completed: {completed}</h3>
      <button onClick={handleClick}>Buy</button>
    </>
  );
}

function delay(ms) {
  return new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}
```

위 코드의 문제는 delay라는 변수의 값을 전달하고 있기 때문이다. 버튼을 클릭하면 이벤트 핸들러 내의 함수는 큐에 담겨서 일괄처리된다. pending의 값은 0이므로,

1. 0 + 1
2. 3초 뒤, 0 - 1
3. 3초 뒤, 0 + 1

이 된다. 그렇기 때문에 pending의 값이 1이 되었다가 -1로 변경되는 것이다. 이를 해결하기 위해서는

```jsx
async function handleClick() {
  setPending(pending + 1);
  await delay(3000);
  setPending((pending) => pending - 1);
  setCompleted((completed) => completed + 1);
}
```

처음 setPending에는 값을 전달하고, 이후에는 업데이터 함수를 전달한다.

### Implement the state queue yourself

여러분의 임무는 각 케이스에 대해 올바른 결과를 반환하도록 getFinalState 함수를 구현하는 것입니다. 올바르게 구현하면 네 가지 테스트를 모두 통과할 것입니다.

두 개의 인수를 받게 됩니다: baseState는 초기 state(예: 0)이고, queue는 숫자(예: 5)와 업데이터 함수(예: n => n + 1)가 추가된, 순서대로 섞여 있는 배열입니다.

```jsx
export function getFinalState(baseState, queue) {
  let finalState = baseState;

  // TODO: do something with the queue...

  return finalState;
}
```

```jsx
export function getFinalState(baseState, queue) {
  let finalState = baseState;

  for (let update of queue) {
    if (typeof update === 'function') {
      finalState += update(finalState);
    } else {
      finalState = update;
    }
  }

  return finalState;
}
```
