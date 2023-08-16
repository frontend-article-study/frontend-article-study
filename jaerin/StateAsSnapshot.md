# State as a Snapshot

| How setting state triggers re-renders (state 설정으로 리렌더링이 촉발되는 방식)                                         |
| ----------------------------------------------------------------------------------------------------------------------- |
| When and how state updates (state 업데이트 시기 및 방법)                                                                |
| <p>Why state does not update immediately after you set it<br>(state를 설정한 직후에 state가 업데이트되지 않는 이유)</p> |
| <p>How event handlers access a “snapshot” of the state<br>(이벤트 핸들러가 state의 ‘스냅샷’에 액세스하는 방법)</p>      |

### state를 설정하면 렌더링이 촉발된다. <a href="#setting-state-triggers-renders" id="setting-state-triggers-renders"></a>

```jsx
import { useState } from 'react';

export default function Form() {
  const [isSent, setIsSent] = useState(false);
  const [message, setMessage] = useState('Hi!');
  if (isSent) {
    return <h1>Your message is on its way!</h1>;
  }
  return (
    <form
      onSubmit={(e) => {
        e.preventDefault();
        setIsSent(true);
        sendMessage(message);
      }}
    >
      <textarea
        placeholder="Message"
        value={message}
        onChange={(e) => setMessage(e.target.value)}
      />
      <button type="submit">Send</button>
    </form>
  );
}

function sendMessage(message) {
  // ...
}
```

Send 버튼을 클릭했을 때 어떤 일이 일어나는지 설명해보자.

- onSubmit 이벤트가 발생한다.
- `setIsSent(true)` 가 isSent의 값을 true로 설정하고새 렌더링을 큐에 대기시킨다.
- React는 새로운 `isSent` 값에 따라 컴포넌트를 다시 렌더링한다.

> _**state와 렌더링은 어떤 관계일까?**_

### 렌더링은 그 시점의 스냅샷을 찍는다.

렌더링이란 컴포넌트, 즉 함수를 호출하는 것. 컴포넌트에서 반환하는 JSX는 시간상 UI의 스냅샷과 같다.\
prop, 이벤트 핸들러, 로컬 변수는 모두 렌더링 당시의 state를 사용해서 계산된다.

**컴포넌트의 메모리로서 state는 함수가 반환된 후 사라지는 일반 변수와 다르다.**

state는 마치 선반에 있는 것처럼 React 자체에 존재한다. 컴포넌트가 호출되면 특정 렌더링에 대한 state의 스냅샷을 제공한다. 컴포넌트는 **해당 렌더링의 state 값을 사용해** 계산된 새로운 props 세트와 이벤트 핸들러가 포함된 UI의 스냅샷을 JSX에 반환한다.

\+3 버튼을 눌렀을 때 number값이 어떤 일이 발생할까?

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber(number + 1);
          setNumber(number + 1);
          setNumber(number + 1);
        }}
      >
        +3
      </button>
    </>
  );
}
```

1. setNumber가 3번 호출되었으니, 3만큼 증가한다.
2. setNumber가 여러 번 호출되어도, 1만 증가한다.

> state를 설정하면 다음 렌더링에서 state가 변경된다.

정답은 2이다. 첫 번째 렌더링에서 number는 0이다. 따라서 해당 렌더링의 onClick 핸들러에서 setNumber(number + 1)가 호출된 후에도 number의 값은 여전히 0이 되는 것이다.

setNumber(number + 1)를 세 번 호출했지만, 이 렌더링에서 이벤트 핸들러의 number는 항상 0이므로 state를 1로 세 번 설정하는 것과 다름없다.

### 시간 경과에 따른 state

\+5버튼을 누르면 alert 창엔 어떤 일이 일어날까?

```typescript
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber(number + 5);
          alert(number);
        }}
      >
        +5
      </button>
    </>
  );
}
```

0이 나오는 것을 짐작할 수 있다. 렌더링이 일어나기 전에 alert 함수가 실행될 것이고 현재 number의 값은 0이기 때문이다.

{% hint style="info" %}
**setTimeout을 이용해서 렌더링된 이후의 값을 보여준다면?**

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
          setTimeout(() => {
            alert(number);
          }, 3000);
        }}
      >
        +5
      </button>
    </>
  );
}
```

```jsx
// 변수 대신 값을 대입해보면,
setNumber(0 + 5);
setTimeout(() => {
  alert(0);
}, 3000);
```

{% endhint %}

state 변수의 값은 렌더링 내에서 절대 변경되지 않는다. 해당 렌더링의 onClick 내에서 setNumber(number + 5)가 호출된 후에도 number의 값은 계속 0이다. React가 UI의 스냅샷을 찍을 때 고정된 값이기 때문이다.

다시 렌더링하기 전에 최신 state를 읽고 싶다면 어떻게 해야 할까?

---

#### 참고

- [https://react-ko.dev/learn/state-as-a-snapshot](https://react-ko.dev/learn/state-as-a-snapshot)
- [https://react.dev/learn/state-as-a-snapshot](https://react.dev/learn/state-as-a-snapshot)
