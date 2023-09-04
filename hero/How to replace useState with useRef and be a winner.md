# How to replace useState with useRef and be a winner

이 글은 [How to replace useState with useRef and be a winner](https://blog.thoughtspile.tech/2021/10/18/non-react-state/)라는 글을 읽고 번역 및 정리한 글입니다.

그대로 전체 번역한 것이 아니고 읽으면서 정리하고 요약한 글입니다. 원본 글을 보고 싶으시다면 링크로 가주시면 감사하겠습니다!

<br>

## React state?

React state는 React 앱의 핵심이며, 앱을 동적으로 만드는 요소입니다.

React state는 useState, useReducer, 클래스 컴포넌트의 this.state 등이 있습니다. React state를 업데이트 하면 컴포넌트가 리렌더링됩니다. 다시 말하면, React state를 업데이트하는 것만이 리렌더링을 하게 합니다. 

ReactDOM.render는 리렌더링을 하는 것이 아니라 앱을 렌더링하는 것이기 때문에 React state는 리액트의 핵심입니다.

만약 React로 관리할 필요가 없는 값을 state로 관리하게 된다면 버그는 보이지 않겠지만, 컴포넌트를 복잡하게 만들고 속도를 느리게 만듭니다.

<br>

## 앱에서 state가 있을 수 있는 위치

1. useRef().current
2. 클래스 컴포넌트의 클래스 프로퍼티
3. 모든 객체의 모든 프로퍼티
4. 상태 관리자
5. DOM 상태 - 리액트가 관리하지 않는 모든 DOM 트리 요소와 속성
6. 변수의 값 - 클로저가 읽을 수 있는 메모리에 있는 값이기 때문

<br>

## React state를 언제 사용해야 할까?

React는 state에 따라 앱을 동적으로 만듭니다. 

이것은 프론트엔드 프레임워크의 핵심 기능이기 때문에 수많은 사례가 존재할 것이라고 생각할 수 있습니다. 

하지만 React state를 사용해야 하는 상황은 단 2가지입니다.

컴포넌트의 DOM에 영향을 미치는 모든 동적인 값은 React state입니다.
```jsx
function Incrementer() {
  const [value, setValue] = useState(0);
  return (
    <button onClick={() => setValue(value + 1)}>
      Clicked {value} times
    </button>
  );
}
```
하지만 VDOM에 영향을 주지 않는 값은 여전히 React state입니다. -> effect를 트리거하기 위해서
```jsx
function TitleRandomizer() {
  const [title, setTitle] = useState('');
  useEffect(() => {
    document.title = title;
  }, [title]);
  return (
    <button onClick={() => setTitle('' + Math.random())}>
      randomize page title
    </button>
  );
}
```
이것은 hooks에만 적용되는 것이 아닙니다. `componentDidUpdate`도 마찬가지입니다. 컴포넌트가 업데이트되었을 때만 호출되기 때문입니다.
```jsx
componentDidUpdate() {
  document.title = this.state.title;
}
```

React state는 `JSX에서 사용`하거나 `useEffect 또는 생명 주기에서 부작용을 유발할 수 있는 값`에서 사용해야 합니다.

<br>

## React state를 사용하지 않았을 때의 이점

1. non-react state는 작업하기가 쉽습니다.
  
    non-react state에 대한 업데이트는 동기로 작동하기 때문에 useEffect를 사용하거나 setState 콜백을 사용할 필요가 없습니다. 또한 직접 할당이 가능해서 편합니다.
    ```jsx
    // react state
    setChecked({ ...checked, [value]: true });
    // non-react state
    checked[value] = true;
    ```

2. non-react state를 업데이트해도 리렌더링되지 않습니다.

    단점일 수도 있지만, 장점으로 활용할 수 있습니다. 렌더링이 일어나지 않는다는 것은 강력한 성능 최적화가 가능합니다. 
    
    또한 refs는 상수-참조 변경가능한 객체이기 때문에 콜백에 의존하는 콜백을 만들 필요가 없습니다. 따라서 memo된 children을 리렌더링하는 과정을 생략할 수 있습니다.
    ```jsx
    const onCheck = useCallback((value) => {
    // re-render, including children
    setChecked({ ...checked, [value]: true });
    }, [checked]);

    const onCheckRef = useRef((value) => {
      // relax, react, nothing happened
      checked[value] = true;
    }).current;
    ```

3. [layout thrashing](https://www.afasterweb.com/2015/10/05/how-to-thrash-your-layout/)에 해당하는 리액트의 문제인 render thrashing을 피할 수 있습니다.

    state의 변경으로 더 많은 state를 변경해야하는 effect가 트리거되어, state가 안정될 때까지 계속 리렌더링을 하는 경우입니다. 정확한 ref 업데이트는 이 함정을 피하는 데 매우 효과적입니다.

    > `layout thrashing이란?`
      <br>
      자바스크립트가 지속적으로 리플로우를 유발하는 방식으로, DOM을 반복적으로 읽고 쓸 때 발생합니다.

4. 과도하게 사용했을 경우 앱이 더 복잡해 보일 수 있습니다.

    state는 리액트에서 매우 중요합니다. state를 건드리면 DOM 변경과 부작용이 발생할 수 있습니다.

<br>

## state를 ref로 대체했을 경우 유용한 예

### Values you only need in callbacks

이벤트 핸들러나 이펙트와 같은 콜백에만 사용하는 경우에는 state가 필요하지 않습니다.
```jsx
function Swiper({ prev, next, children }) {
  const [startX, setStartX] = useState();
  const detectSwipe = e => {
    e.touches[0].clientX > startX ? prev() : next();
  };
  return <div
    onTouchStart={e => setStartX(e.touches[0].clientX)}
    onTouchEnd={detectSwipe}
  >{children}</div>;
}
```
`startX`는 DOM에 영향을 주거나 이펙트를 실행하지 않으며, 나중 `touchend`때 읽기 위해 저장할 뿐입니다. 

`touchstart`때 state를 조작했으므로 쓸모없는 렌더링이 발생합니다.

<br>

```jsx
function Swiper({ prev, next, children }) {
  const startX = useRef();
  const detectSwipe = e => {
    e.touches[0].clientX > startX.current ? prev() : next();
  };
  return <div
    onTouchStart={e => startX.current = e.touches[0].clientX}
    onTouchEnd={detectSwipe}
  >{children}</div>;
}
```
`useRef`를 사용해서 바꾼 예입니다. 이제 Swiper에서 `touchstart` 이벤트가 발생해도 리렌더링이 되지 않습니다. 

또한 `detectSwipe`는 변경되는 startX 참조에 의존하지 않기 때문에 `useCallback`을 사용할 수도 있습니다.

> 참조에 DOM 노드를 저장하는 것은 특수한 경우입니다. callback에서만 노드에 접근하기 때문에 작동합니다.

<br>

### Buffering state updates

```jsx
function Swiper({ children }) {
  const startX = useRef(null);
  const [offset, setOffset] = useState(0);
  const onStart = (e) => {
    startX.current = e.touches[0].clientX;
  };
  const trackMove = (e) => {
    setOffset(e.touches[0].clientX - startX.current);
  };
  return <div
    onTouchStart={onStart}
    onTouchMove={trackMove}
  >
    <div style={{ transform: `translate3d(${offset}px,0,0)` }}>
      {children}
    </div>
  </div>;
}
```
`touchMove` 이벤트는 프레임당 4~5번의 렌더링을 일으킵니다. 사용자는 마지막 렌더링만 보기 때문에 앞 4번의 렌더링은 필요가 없습니다.

`requestAnimationFrame`는 swipe 위치를 ref로 기억하지만 프레임당 한 번만 업데이트 하기 때문에 위의 예시에 적합합니다.

```jsx
const pendingFlush = useRef();
const trackMove = (e) => {
  if (startX.current != null) {
    cancelAnimationFrame(pendingFlush.current);
    pendingFlush.current = requestAnimationFrame(() => {
      setOffset(e.clientX - startX.current);
    });
  }
};
```

<br>

위의 예시말고도 다른 예시가 있습니다. `requestAnimationFrame`를 취소하는 대신에 모두 실행하게 하고, 상태를 동일한 값으로 설정해서 하나만 리렌더링 시킬 수 있습니다. 
```jsx
const pendingOffset = useRef();
const trackMove = (e) => {
  if (startX.current != null) {
    pendingOffset.current = e.clientX - startX.current;
    requestAnimationFrame(() => {
      setOffset(pendingOffset.current);
    });
  }
};
```

이 예시들은 state와 ref를 함께 사용해서 커스텀 업데이트 일괄 처리 메커니즘을 구현한 것입니다.

<br>

### State that you want to manage yourself

사용자가 커서를 움직이면 React가 current offset을 결정하고 그에 따라 style을 업데이트하도록 합니다. 

React는 빠를지 모르지만 `trackMove`가 `transform`을 변경한다는 사실을 알지 못하기 때문에 렌더를 호출하고, vDOM을 생성하고, Diff를 생성한 다음, transform을 업데이트해야 하는 등 많은 추측을 수행해야 합니다. 

하지만 우리가 무엇을 하고 있는지 알고 있고, 직접 수행함으로써 React의 모든 수고를 덜어줄 수 있습니다.

```jsx
function Swiper({ children }) {
  const startX = useRef(null);
  const transformEl = useRef();
  const onStart = (e) => {
    startX.current = e.touches[0].clientX;
  };
  const trackMove = (e) => {
    const offset = e.touches[0].clientX - startX.current;
    transformEl.current.style.transform = `translate3d(${offset}px,0,0)`;
  };
  return <div
    onTouchStart={onStart}
    onTouchMove={trackMove}
  >
    <div ref={transformEl}>
      {children}
    </div>
  </div>;
}
```

위의 예시에서는 렌더링이 0입니다! 특히 여러 가지 요소가 DOM에 영향을 미칠 수 있는 경우 이 방법은 매우 쉽게 속일 수 있습니다. 

이것은 애니메이션이나 제스처와 같은 저수준의 작업을 자주 사용하는 경우에 사용하면 큰 차이를 만들 수 있습니다.

<br>

### Derived state

값이 항상 React staet와 함께 업데이트되는 경우, 우리는 그 리렌더링에 편승하여 반응 상태가 아닌 다른 것을 업데이트할 수 있습니다. 

이것은 매우 깔끔할 수 있습니다. 모든 변수가 state를 보유한다고 말했던 것을 기억하시나요?

```jsx
const [value, setValue] = useState(0);
const isValid = value >= 0 && value < 100;
```

이것은 더 까다롭고 참조가 필요할 수 있지만, `useMemo`처럼 겉으로 보기에는 여전히 간단하지만 내부 깊숙이 참조를 사용합니다.

```jsx
const [search, setSearch] = useState('');
const matches = useMemo(() => {
  return options.filter(op => op.startsWith(search));
}, [options, search]);
```

두 경우 모두 non-react state를 사용하여 업데이트를 마스터 상태와 신중하게 동기화합니다. cascading state update보다 훨씬 낫습니다.

```jsx
const [search, setSearch] = useState('');
const [matches, setMatches] = useState([]);
useEffect(() => {
  setMatches(options.filter(op => op.startsWith(search)));
}, [options, search]);
```

<br>

## 요약

- 리액트 앱의 state는 `React state`(this.state, useState, useReducer) 또는 `non-react state`(ref.current, 객체 속성, 변수 값 등)일 수 있습니다.
- 반응 상태에 대한 업데이트만 리액트를 다시 렌더링하므로 vDOM이 의존할 때 또는 `useEffect`를 트리거할 때 사용해야 합니다.
- 상태를 사용하지 않으면 몇 가지 장점이 있습니다:

    - 렌더링 횟수 감소
    - 보다 안정적인 콜백
    - 캐스케이딩 상태 업데이트, 일명 렌더링 쓰레싱이 없습니다.
    - 데이터를 동기적으로 변경하는 것이 매우 좋습니다.
    
<br>

- non-react state에 의존하는 4가지 최적화 방법

    - 값이 콜백에서만 사용되는 경우 - 값을 참조로 만듭니다(DOM 참조 포함).
    - ref는 보류 중인 상태 업데이트를 위한 buffer가 될 수 있습니다.
    - 리액트를 사용하지 않고도 DOM을 직접 업데이트할 수 있다고 생각되면 ref를 사용하면 됩니다.
    - 파생 상태도 핵심 상태 변경에 따라 신중하게 업데이트되는 ref에 의존합니다.