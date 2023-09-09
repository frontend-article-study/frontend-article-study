[리액트의 공식 문서](https://react.dev/learn/you-might-not-need-an-effect)를 보면 `effect`는 React 패러다임에서 벗어날 수 있는 탈출구라고 소개합니다. effect를 사용해 네트워크, 브라우저 DOM과 같은 외부 시스템과 리액트 컴포넌트를 동기화할 수 있기 때문입니다.

<br />

이 말은 다르게 말하면 외부 시스템이 관여되지 않은 경우 effect가 필요하지 않다는 뜻입니다.

리액트에서 소개하는 effect의 사용 의도와 다르게 사용할 경우, 코드의 흐름을 따라가기 어렵고 오류가 발생하기 쉬운 코드가 됩니다.

<br />

effect의 올바른 사용법을 익혀 깨끗한 코드를 작성하기 위해 [리액트 공식 문서](https://react.dev/learn/you-might-not-need-an-effect)를 읽으며 useEffect를 사용할 때 지양해야 하는 패턴에 대해 이해한 내용을 정리했습니다.

## 불필요한 effect를 사용한 예시와 개선 방법

다음 예시는 effect를 사용할 필요가 없는 대표적인 예시입니다.

1. 렌더링을 위한 데이터 변환에 effect 사용하기
2. 사용자 이벤트를 처리하기 위해 effect 사용하기

<br />

좀 더 구체적인 예시를 보며 effect가 불필요한 상황에 대해 알아보겠습니다.

### props나 state를 바탕으로 또 다른 state 업데이트하기

다음과 같이 2가지 상태가 있습니다.

```tsx
const [firstName, setFirstName] = useState('Taylor');
const [lastName, setLastName] = useState('Swift');
```

사용자가 입력한 firstName(상태), lastName(상태)를 바탕으로 `fullName`을 얻고 싶은 상황이라면 어떻게 코드를 작성해야 할까요?

<br />

firstName과 lastName이 변할 때마다 실행돼야 하므로, fullName이라는 상태를 선언하고 다음처럼 effect를 활용해 업데이트할 수 있습니다.

```tsx
// 🔴 bad : 불필요한 상태와 불필요한 이펙트
useEffect(() => {
  setFullName(firstName + ' ' + lastName);
}, [firstName, lastName]);
```

하지만 위 코드는 firstName과 lastName이 바뀔 때 렌더링되고, 이후 실행되는 useEffect의 콜백을 실행하면서(fullName을 셋팅하면서) 상태가 변경되어 한 번 더 렌더링이 발생하기 때문에 비효율적입니다.

firstName과 lastName이 바뀌면서 렌더링할 때 fullName을 계산하는 방식으로 변경하는 것이 좋습니다. (또한, 리액트에서는 [기존 props나 state를 사용해 계산할 수 있는 것은 state로 사용할 필요가 없다](https://react.dev/learn/choosing-the-state-structure#avoid-redundant-state)고 말합니다.)

```tsx
const [firstName, setFirstName] = useState('Taylor');
const [lastName, setLastName] = useState('Swift');

// const [fullName, setFullName] = useState(''); -> 불필요한 상태
// 🟢 good
const fullName = firstName + ' ' + lastName;
```

이렇게 작성하면 코드가 빨라지고 간단해지며 두 가지 상태 변수가 서로 동기화되지 않아 발생하는 버그를 피할 수 있습니다.

### 비싼 연산 캐싱하기

다음과 같이 할 일들을 담은 상태가 있습니다.

```tsx
const [todos, setTodos] = useState<Todo[]>([]);
```

전체 할 일들 중, 완료되지 않은 할 일만을 UI로 보여주고 싶다면 코드를 어떻게 작성해야 할까요?

이전 예시를 잘 이해하셨다면, state와 effect를 사용할 필요가 없고 렌더링 때 계산하면 되겠다고 생각할 수 있을 것입니다.

```tsx
// 🔴 bad : 불필요한 상태와 불필요한 이펙트
const [visibleTodos, setVisibleTodos] = useState<Todo[]>([]);

useEffect(() => {
  setVisibleTodos(getFilteredTodos(todos, filter));
}, [todos, filter]);

// 🟢 good : todos가 변경되어 렌더링할 때 계산하기
const visibleTodos = getFilteredTodos(todos, filter);
```

위 코드는 state와 effect를 불필요하게 사용하지 않은 좋은 코드입니다.

다만, getFilteredTodos 함수는 이 함수와 직접적인 관련이 없는 상태가 업데이트되어 렌더링할 때도 실행됩니다. 따라서 이 함수와 직접적인 관련이 있는 상태가 변경되었을 때만 실행하고 싶다면, useMemo 훅을 사용하여 계산을 캐싱하면 됩니다.

```tsx
// todos와 filter가 변경되어 리렌더링이 발생했을 때만 함수를 실행합니다.
const visibleTodos = useMemo(() => getFilteredTodos(todos, filter), [todos, filter]);
```

이렇게 작성하면 todos나 filter 조건이 변경되지 않는 한 내부 함수가 다시 실행되지 않기 원한다고 리액트에게 알릴 수 있습니다.

### props가 변경되면 모든 state를 초기화하기

userId를 props로 받는 ProfilePage 컴포넌트에서 userId에 해당하는 사용자가 입력한 댓글을 다루는 comment 상태를 가지고 있습니다.

```tsx
const ProfilePage = ({ userId }) => {
  const [comment, setComment] = useState('');

  // ...
};
```

props로 받은 userId가 바뀌면 comment를 빈 문자열로 초기화하고 싶을 때 어떻게 작성해야 할까요?

```tsx
// 🔴 bad
useEffect(() => {
  setComment('');
}, [userId]);
```

위처럼 effect를 사용해 쉽게 처리할 수 있을 것 같지만, 이는 효율적이지 않습니다.

effect는 첫 렌더링이 끝나면 실행되기 때문에, 원래있던 comment의 값으로 렌더링을 한 다음에야 `setComment(’’)`가 실행됩니다. 그리고 ProfilePage 내부에 있는 모든 컴포넌트에서도 동일한 작업을 해야 하므로 코드가 복잡해집니다.

이러한 경우 컴포넌트에 명시적인 key를 전달하여 userId가 바뀌면 다른 프로필 페이지가 돼야 한다는 것을 알리는 것이 좋습니다.

```tsx
<ProfilePage userId={userId} key={userId} />
```

일반적으로 React는 동일한 컴포넌트가 같은 위치에 렌더링될 때 상태를 보존합니다.

이때 userId를 key로 전달하면, 상이한 userId를 갖는 컴포넌트는 상태를 공유할 수 없는 별개의 컴포넌트로 취급합니다. 따라서 userId가 변경될 때마다 해당 컴포넌트와 그 자식 컴포넌트들의 상태를 재설정합니다.

### props가 변경될 때 일부 state 조정하기

이전 예시와 props가 변경될 때 다르게 전체를 재설정하는 것이 아닌 일부만을 재설정하고 싶을 수 있습니다.

다음처럼 items를 props로 받는 List 컴포넌트가 있고, items가 바뀔 때마다 selection을 null로 변경하고 싶은 상황이라 생각해 봅시다.

```tsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // ...
}
```

이전 예시에서 언급했듯, items가 변경될 때마다 `setSelection(null)`이 호출되도록 effect 훅을 사용하는 것은 바람직하지 않습니다. effect 훅을 사용하면, 항상 이전 값을 사용해 렌더링을 마친 다음에야 selection 상태가 업데이트된다는 것을 생각해야 합니다.

effect를 제거하고 렌더링 중에 상태를 조정해 보겠습니다.

```tsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // Better: 렌더링 중에 조정
  const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
  // ...
}
```

prevItems을 사용한 것처럼, 이전 렌더링의 상태를 저장하는 패턴이 이해되지 않을 수 있지만, 이는 effect를 통해 상태를 업데이트하는 것보다 낫습니다.

위 코드에서 `setSelection`은 렌더링 도중에 호출되어 List의 return문이 끝나면 즉시 리렌더링 합니다. 이때 리액트는 List의 children을 렌더하지 않았고 DOM을 업데이트하지도 않은 상태기 때문에 stale한 상태를 가지고 렌더링하는 것을 막아줍니다. (반면 effect를 사용했다면 stale한 상태를 가지고 렌더링을 마친 다음에서야 상태가 업데이트되어 두 번째 렌더링 때 최신의 상태 값으로 렌더링 해 보여주겠지요.)

<br />

그런데 이 패턴은 effect보다 효율적일지라도, 필요한 패턴은 아닙니다.

어찌 됐든 props나 state에 따라 다른 상태를 조정하면 데이터의 흐름을 이해하고 디버깅하기 어려워집니다. 따라서 **key를 사용해 모든 상태를 리셋**하거나, **모든 것을 렌더링 중에 계산**할 수 있는지 생각해야 합니다.

다음 코드는 **모든 것을 렌더링 중에 계산하는 방식**으로 작성한 코드입니다.

```tsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);
  // 🟢 Best: 모든 것을 렌더링 중에 계산하기
  const selection = items.find((item) => item.id === selectedId) ?? null;
  // ...
}
```

위처럼 작성하면 상태를 조정할 필요가 없습니다.

선택한 ID를 가진 항목이 items에 있으면 선택된 상태로 유지되며, 찾을 수 없다면 렌더링 중에 계산된 선택 항목은 null이 됩니다. (이 동작은 items가 변경됐을 때 겹치는 item이 있다면 선택을 유지합니다만, 대부분의 항목 변경이 선택 내용을 유지하므로 더 나은 방법이라고 할 수 있습니다.)

## 이벤트 핸들러 간 로직 공유하기

제품을 구매할 수 있는 두 개의 버튼(장바구니에 담기, 구매하기)을 포함하는 페이지가 있다고 가정해 봅시다.

사용자가 제품을 장바구니에 넣을 때마다 alert을 띄우고 싶은데, 두 버튼의 클릭 핸들러에서 showNotification 함수를 호출하는 것은 반복적이라고 느껴져 effect를 활용해 코드를 작성했습니다.

```tsx
function ProductPage({ product, addToCart }) {
  // 🔴 Avoid: 이벤트 관련 로직 배치
  useEffect(() => {
    if (product.isInCart) {
      showNotification(`Added ${product.name} to the shopping cart!`);
    }
  }, [product]);

  function handleBuyClick() {
    addToCart(product);
  }

  function handleCheckoutClick() {
    addToCart(product);
    navigateTo('/checkout');
  }
  // ...
}
```

여기서 effect는 불필요하고 버그를 유발할 가능성이 높습니다.

만약 앱이 장바구니를 기억한다면, 카트에 제품을 추가하고 페이지를 새로 고칠 때마다 알림이 계속해서 호출될 것입니다.

<br />

코드를 effect 내부에 둬야할지 이벤트 핸들러에 둬야할지 고민되는 경우, 이 코드가 실행돼야 하는 이유에 대해 생각해 보세요.

이 컴포넌트 자체가 사용자에게 표시되었기 때문에 실행돼야 하는 코드라면 effect 내부에 두면 됩니다.

<br />

그런데 위 예제는 페이지가 표시되었기 때문이 아닌 사용자가 버튼을 눌렀기 때문에 실행돼야 하는 코드이므로, effect를 사용할 필요가 없습니다.

effect를 제거하고 공유 로직을 분리해 함수로 만든 다음 두 이벤트 핸들러에서 호출하세요.

```tsx
function ProductPage({ product, addToCart }) {
  // 🟢 Good: 이벤트 관련 로직은 이벤트 핸들러에 배치
  function buyProduct() {
    addToCart(product);
    showNotification(`Added ${product.name} to the shopping cart!`);
  }

  function handleBuyClick() {
    buyProduct();
  }

  function handleCheckoutClick() {
    buyProduct();
    navigateTo('/checkout');
  }
  // ...
}
```

### 페이지가 표시되었기 때문에 실행돼야 하는 코드

사용자가 Form을 열람하면 애널리틱스 요청을 보내고 싶습니다. 이러한 경우 애널리틱스 요청을 보내는 코드는 Form이 표시되었기 때문에 실행하는 것이 옳습니다. 따라서 effect로 사용하기 적합한 경우입니다.

반면, Form을 submit하는 버튼을 클릭했을 때 등록 요청을 보내는 것은 버튼을 클릭했기 때문에 실행돼야 하는 것이므로 버튼 클릭 핸들러에서 처리하는 것이 옳습니다.

```tsx
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // 🟢 Good: 컴포넌트가 보여졌기 때문에 실행돼야 하는 코드
  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  // 🔴 Avoid: 특정 이벤트 관련 로직을 effect에 배치
  const [jsonToSubmit, setJsonToSubmit] = useState(null);
  useEffect(() => {
    if (jsonToSubmit !== null) {
      post('/api/register', jsonToSubmit);
    }
  }, [jsonToSubmit]);

  function handleSubmit(e) {
    e.preventDefault();
    setJsonToSubmit({ firstName, lastName });
  }
  // ...
}
```

이처럼 배치 위치를 이벤트 핸들러와 effect 사이에서 고민하고 있다면, 사용자 관점에서 어떤 종류의 로직인지 생각해 보면 됩니다.

사용자의 특정 상호 작용 발생 때문이라면 핸들러에, 사용자가 화면에서 이 컴포넌트를 보았기 때문이라면 effect에 배치하면 됩니다.

### 연쇄적인 계산

```tsx
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);
  const [isGameOver, setIsGameOver] = useState(false);

	// 1
  useEffect(() => {
    if (card !== null && card.gold) {
      setGoldCardCount(c => c + 1);
    }
  }, [card]);

	// 2
  useEffect(() => {
    if (goldCardCount > 3) {
      setRound(r => r + 1)
      setGoldCardCount(0);
    }
  }, [goldCardCount]);

	// 3
  useEffect(() => {
    if (round > 5) {
      setIsGameOver(true);
    }
  }, [round]);

  // 4
  useEffect(() => {
    alert('Good game!');
  }, [isGameOver]);

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    } else {
      setCard(nextCard);
    }
  }

  // ...
```

위 코드의 흐름을 보겠습니다.

1. handlePlaceCard 함수가 호출됐을 때, 게임이 끝나지 않았다면 card 상태를 업데이트 합니다. (리렌더링 발생)
2. 리렌더링이 끝난 후, card가 변경되었기 때문에 1번 effect 콜백이 실행됩니다.
   1. card가 null이 아니고 gold라면 goldCardCount 상태를 업데이트 합니다. (리렌더링 발생)
3. 리렌더링이 끝난 후, goldCardCount가 변경되었기 때문에 2번 effect 콜백이 실행됩니다.
   1. goldCardCount가 3보다 크면 round와 goldCardCount 상태를 업데이트 합니다. (리렌더링 발생)
4. 리렌더링이 끝난 후, goldCardCount와 round가 변경되었기 때문에 2번 3번 effect 콜백이 실행됩니다.
   1. goldCardCount가 3보다 크면 round와 goldCardCount 상태를 업데이트 합니다. (리렌더링 발생)
   2. round가 5보다 크면 gameover 상태를 업데이트 합니다. (리렌더링 발생)

이 모든 과정은 setCard가 호출되었기 때문에 비롯된 코드이고,

setCard를 호출하는 코드 블럭 안에서는 업데이트된 상태를 바로 사용할 수 없기 때문에, 최신의 값을 사용하기 위해 이를 effect로 다룬 것으로 보입니다.

<br />

예상대로 동작할지라도, 매우 비효율적인 코드입니다.

코드의 흐름을 알아보기 어려울 뿐더러 effect 내부에서 호출한 setState가 불필요한 리렌더링을 일으키기 때문입니다.

<br />

이전 예시를 통해 알아본 것처럼, 어떤 이벤트가 발생했기 때문에 실행돼야 하는 코드는 effect가 아닌 **이벤트 핸들러에서 처리할 수 있는지**, **더 나아가 렌더링 중에 계산될 수 있는지** 생각하는 것이 좋습니다.

예시 코드를 다음처럼 리팩토링할 수 있습니다.

```tsx
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);

  // 🟢 good : game over 여부는 렌더링 중에 계산할 수 있습니다. (상태로 관리할 필요가 없습니다.)
  const isGameOver = round > 5;

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    }

    // 🟢 good : 다음 상태를 가지고 핸들러 안에서 처리합니다.
    setCard(nextCard);
    if (nextCard.gold) {
      if (goldCardCount <= 3) {
        setGoldCardCount(goldCardCount + 1);
      } else {
        setGoldCardCount(0);
        setRound(round + 1);
        if (round === 5) {
          alert('Good game!');
        }
      }
    }
  }

  // ...
```

새로운 상태값을 활용해 코드를 실행해야 한다면, state가 업데이트된 다음에 실행하려 하지 말고 state를 업데이트할 때 넣어준 그 값 자체를 활용하면 됩니다!

불필요한 리렌더링을 확 줄여주고 좀 더 이해하기 쉬운 코드를 작성할 수 있습니다.

## 정리

effect는 외부 시스템과 리액트 컴포넌트를 동기화할 수 있게 도와줍니다.

그런데 잘못된 상황에서 사용할 경우 불필요한 리렌더링을 초래하고 코드를 이해하기 어렵게 만들기 때문에 올바르게 사용하는 것이 중요합니다.

다음 내용을 고민해보고 effect를 사용하기 적절하다고 판단되면 그 때 사용하시길 바라겠습니다.

1. **렌더링 중에 계산**하세요.
   1. 이 state가 반드시 필요한가 생각해 보면 도움이 됩니다.
2. 특정 데이터가 변경되었을 때 실행하고 싶다면 **useMemo**를 사용하세요.
3. 어떤 props나 state에 따라 전체 상태를 리셋하고 싶다면 **key**를 사용하세요.
4. 일부 state를 변경하고 싶다면 **렌더링 중에 셋팅**하세요.
5. 코드를 실행해야 하는 이유가 **컴포넌트가 화면에 보여졌기 때문이라면 effect**를 사용하고 **사용자와의 상호작용이 발생했기 때문이라면 이벤트 핸들러**에서 처리하세요.
6. 여러 상태를 업데이트해야 한다면, **단일 이벤트에서 업데이트**해 보세요.
7. effect로 데이터를 fetch한다면 경쟁 조건을 피하기 위해 cleanup을 구현하세요.

### 출처

[https://react.dev/learn/you-might-not-need-an-effect](https://react.dev/learn/you-might-not-need-an-effect)
