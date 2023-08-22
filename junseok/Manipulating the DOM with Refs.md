# Manipulating the DOM with Refs

#### 작성 : 손준석
- 참고사이트 
   - https://react-ko.dev/learn/manipulating-the-dom-with-refs
   - https://react.dev/learn/manipulating-the-dom-with-refs


## DOM ref
React에서는 DOM을 자동으로 업데이트 하지만, 상황에 따라 React가 관리하는 DOM 요소에 직접 접근해서 작업을 해야 합니다. (ex. 스크롤, 크기&위치 측정) React에서는 이를 위해 ref를 제공합니다. 

```tsx
# 텍스트 input에 초점 맞추기
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

> ### How to manage a list of refs using a ref callback?
> - map을 통해 jsx를 렌더링 할떄, 각각의 jsx마다 개별 ref를 정의하는 것은 비효율적입니다. 하지만  훅은 컴포넌트의 최상위 레벨에서만 호출해야 하기 >때문입니다. 반복문 또는 map() 내부에서는 useRef를 >호출할 수 없습니다.
> - 이를 위한 해결책으로 ref Callback을 소개합니다. current 값으로 Map 자료를 할당하고 map 자료 내부에서 데이터를 관리하는 방식입니다.
> ```ts
>
>import { useRef } from 'react';
>
>export default function App() {
>  const itemsRef = useRef<Map<any, HTMLLIElement> | null>(null);
>
>  function scrollToId(itemId:number) {
>    const map = getMap();
>    const node = map.get(itemId); 
>    if(node){
>      node.scrollIntoView({
>        behavior: 'smooth',
>        block: 'nearest',
>        inline: 'center'
>      });
>    }
>
>  }
>
>  function getMap() {
>    if (!itemsRef.current) {
>      // Initialize the Map on first usage.
>      itemsRef.current = new Map();
>    }
>    return itemsRef.current;
>  }
>
>  return (
>    <>
>      <nav>
>        <button onClick={() => scrollToId(0)}>
>          Tom
>        </button>
>        <button onClick={() => scrollToId(5)}>
>          Maru
>        </button>
>        <button onClick={() => scrollToId(9)}>
>          Jellylorum
>        </button>
>      </nav>
>      <div>
>        <ul>
>          {catList.map(cat => (
>            <li
>              key={cat.id}
>              ref={(node) => {
>                const map = getMap();
>                if (node) {
>                  map.set(cat.id, node);
>                } else {
>                  map.delete(cat.id);
>                }
>              }}
>            >
>              <img
>                src={cat.imageUrl}
>                alt={'Cat #' + cat.id}
>              />
>            </li>
>          ))}
>        </ul>
>      </div>
>    </>
>  );
>}
>
>type Cat = {
>  id: number
>  imageUrl: string
>}
>
>const catList: Cat[] = [];
>for (let i = 0; i < 10; i++) {
>  catList.push({
>    id: i,
>    imageUrl: 'https://placekitten.com/250/200?image=' + i
>  });
>}
>```

## Accessing another component’s DOM nodes
- 컴포넌트 분리를 통해 렌더링을 진행할 때, prop으로 ref를 전달해야 할 때가 있을 수 있습니다. 가령 input tag를 커스텀 하게 디자인 한 `<MyInput />`이 있고 해당 컴포넌트에 prop으로 ref 값을 전달하고자 합니다. 이때 기본적으로 `null`이 반환됩니다. 
- 이는 기본적으로 React가 다른 컴포넌트의 DOM 노드에 접근하는 것을 허용하지 않기 떄문입니다. 
- 이를 해결하기 위해서 별도의 설정이 필요합니다. 이는 `fowardRef` API를 통해 해결할 수 있습니다. 

```ts
import { forwardRef, useRef } from 'react';

const MyInput = forwardRef<HTMLInputElement>((props, ref) => {
  return <input {...props} ref={ref} />;
});

export default function App() {
  const inputRef = useRef<HTMLInputElement>(null);

  function handleClick() {
    if(inputRef?.current){
      inputRef.current.focus();
    }

  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}

```

1.  `<MyInput ref={inputRef} />` 를 통해 ref를 prop으로 전달합니다. 
2. `MyInput`은 prop으로 전달된 ref 값을 허용하기 위해서 `forwardRef`를 사용해 선언됩니다. `props` 다음에 선언되는 두 번째 `ref` 인수로 인해서 `inputRef`를 받을 수 있습니다.
3. `MyInput`은 prop을 통해 받은 ref를 내부의 `<input>`으로 전달합니다.


## When React attaches the refs

- 일반적으로 렌더링 주에는 ref에 접근하는 것을 원하지 않습니다. 따라서 첫 번째 렌더링 중네는 DOM 노드가 생성되지 않았으므로 `ref.current` 는 `null`이 됩니다. 
- 이후 React는 커밋하는 동안에 `ref.current`를 설정합니다. React는 DOM이 업데이트 되기 전에는 `ref.current`의 값을 `null`로 설정하였다가, DOM이 업데이트된 직후 해당 DOM 노드로 다시 설정합니다.

> ### Flushing state updates synchronously with flushSync
> - `react-dom` 의 `flushSync`를 사용하면 React 가 DOM을 동기적으로 업데이트 하도록 강제할 수 있습니다. 
> -  `flushSync`로 감싼 코드가 실행된 직후 React가 DOM을 동기적으로 업데이트하도록 지시합니다.
>```ts
> import { useState, useRef } from 'react';
>import { flushSync } from 'react-dom';
>
>export default function TodoList() {
>  const listRef = useRef(null);
>  const [text, setText] = useState('');
>  const [todos, setTodos] = useState(
>    initialTodos
>  );
>
>  function handleAdd() {
>    const newTodo = { id: nextId++, text: text };
>    flushSync(() => {
>      setText('');
>      setTodos([ ...todos, newTodo]);      
>    });
>    listRef.current.lastChild.scrollIntoView({
>      behavior: 'smooth',
>      block: 'nearest'
>    });
>  }
>
>  return (
>    <>
>      <button onClick={handleAdd}>
>        Add
>      </button>
>      <input
>        value={text}
>        onChange={e => setText(e.target.value)}
>      />
>      <ul ref={listRef}>
>        {todos.map(todo => (
>          <li key={todo.id}>{todo.text}</li>
>        ))}
>      </ul>
>    </>
>  );
>}
>
>let nextId = 0;
>let initialTodos = [];
>for (let i = 0; i < 20; i++) {
>  initialTodos.push({
>    id: nextId++,
>    text: 'Todo #' + (i + 1)
>  });
>}
>```

## Best practices for DOM manipulation with refs
- state를 통해 관리하는 DOM node와 ref를 통해 인위적으로 관리하는 DOM node를 구분해야 합니다. 
- 만약 state와 ref로 관리하는 DOM node가 동일하다면, react에서는 ref를 통해 관리하는 DOM node가 무엇인지 알지 못하기 때문에 오류가 발생할 수 있습니다. 


## Recap
- Ref는 일반적인 개념이긴 하지만, 대부분 DOM 엘리먼트를 보관할 때 사용합니다.
- `<div ref={myRef}>`를 전달해 DOM 노드를 myRef.current에 넣으라고 React에 지시합니다.
- 보통은 포커스, 스크롤, DOM 엘리먼트 측정과 같은 비파괴적인 동작에 ref를 사용합니다.
- 컴포넌트는 기본적으로 DOM 노드를 노출하지 않습니다. forwardRef를 사용하고 두 번째 ref 인수를 특정 노드에 전달하여 DOM 노드를 노출하도록 설정할 수 있습니다.
- React가 관리하는 DOM 노드를 변경하지 마세요.
- React가 관리하는 DOM 노드를 수정해야 한다면 React가 업데이트할 이유가 없는 부분을 수정하세요.
