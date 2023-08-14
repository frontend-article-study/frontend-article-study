#📌 useTransition
`useTransition`은 UI를 차단하지 않고 state를 업데이트할 수 있는 React 훅입니다.

###목차

1. [리액트에 도입된 fiber 엔진](#리액트에-도입된-fiber-엔진)
2. [참조](#reference-참조)
3. [개요](#개요)
4. [매개변수](#매개변수)
5. [사용법](#사용법)

### 리액트에 도입된 fiber 엔진

- 리액트 18버전은 fiber라는 엔진을 개선하여 자체적인 스케쥴러를 갖게 되었습니다.
- 작업의 우선순위를 정하고, 우선순위 높은 작업이 들어오면 먼저 처리하는 기능이 구현되었습니다.
- 무겁고 유저 경험에 중요하지 않은 작업은 우선순위를 낮춰 프레임률을 유지할 수 있습니다.
  <br>

### Reference 참조

상태 변화의 우선순위를 지정하기 위해 useTransition 훅이 새로 도입되었습니다. useTransition은 `isPending`, `startTransition` 을 반환하는데, `isPending` 은 작업이 지연되고 있음을 알리는 boolean 이며, `startTransition`은 낮은 우선순위로 실행할 함수를 인자로 받는다. 다음과 같이 사용할 수 있습니다.

```JSX
function App() {
  const [isPending, startTransition] = useTransition();
  const [count, setCount] = useState(0);

  function handleClick() {
    startTransition(() => {
      setCount(c => c + 1);
    })
  }


  return (
    <div>
      {isPending && <Spinner />}
      <button onClick={handleClick}>{count}</button>
    </div>
  );
}
```

- 클릭할 때마다 일어나는 count 에 대한 상태 업데이트를 낮은 우선순위로 실행합니다.
- 그래서 더 중요한 이벤트가 있는 경우 count의 업데이트를 지연시키고 대신 이전의 값을 보여줍니다.
- isPending을 이용하여 업데이트가 지연된 동안 스피너를 보여줄 수도 있습니다.

```TSX
export default function Home() {
  const [text, setText] = useState("");
  const [value, setValue] = useState("");
  const [isPending, startTransition] = useTransition();

  const onChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    startTransition(() => {
      setText(e.target.value);
    });
    setValue(e.target.value);
  };

  console.log({ text, isPending });
  console.log({ value });

  return <input type="text" onChange={onChange} />;
}
```

- onChange 함수가 실행되면 setText와 setValue가 실행되면서 상태가 변하게 됩니다.
- 하지만 setText는 startTransition 함수로 래핑 되어있습니다.
- 이렇게 래핑 되면 상태변화의 우선순위가 낮아지고, 다른 상태변화가 전부 일어난 후 setText가 실행되어 text 상태가 변하게 됩니다.

<br>

### Usage 사용법

```JSX
function App() {
  const [isPending, startTransition] = useTransition();
  const [filterTerm, setFilterTerm] = useState('');

  const filteredProducts = filterProducts(filterTerm);

  function updateFilterHandler(event) {
    startTransition(() => {
      setFilterTerm(event.target.value);
    });
  }

  return (
    <div id="app">
      <input type="text" onChange={updateFilterHandler} />
      {isPending && <p>Updating List...</p>}
      {!isPending && <ProductList products={filteredProducts} />}
    </div>
  );
}
```
