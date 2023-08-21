# Render and Commit

> React는 고객의 주문을 받고 음식을 가져다주는 웨이터!

우리가 흔히 생각하는 웨이터가 하는 역할은 주문을 받고 주방에 주문을 전달하고 테이블에 주문한 요리를 서빙한다. 이와 비슷하게 리액트는 렌더링을 발생시키고 컴포넌트를 렌더링한 후 DOM에 커밋한다.

## Step 1: Trigger a render

컴포넌트 렌더링이 일어나는 두 가지 이유

1. 초기 렌더링인 경우 (Initial render)
2. state가 업데이트된 경우

### Initial render

앱을 시작하기 위해선 처음 렌더링이 일어나야 한다. 그러기 위해선 `createRoot`를 호출한 다음 render 메서드를 사용한다.

```jsx
import Image from './Image.js';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(<Image />);
```

### Re-renders when state updates

state를 업데이트해서 렌더링을 트리거시킬 수 있다. 컴포넌트의 state를 업데이트하면 렌더링이 대기열에 추가된다.

## Step 2: React renders your components

렌더링을 트리거시키면 React는 컴포넌트를 호출하여 화면에 표시할 내용을 확인한다.

> "Rendering"은 React에서 컴포넌트를 호출하는 것

- 처음 렌더링에서는 root 컴포넌트를 호출한다.
- 추가 렌더링에서는 state에 업데이트에 의해 렌더링이 발동된 **함수 컴포넌트**를 호출한다. 👉🏻 이 과정은 재귀적으로 동작

```jsx
import Image from './Image.js';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(<Gallery />);

function Gallery() {
  return (
    <section>
      <h1>Inspiring Sculptures</h1>
      <Image />
      <Image />
      <Image />
    </section>
  );
}
```

- 첫 렌더링을 하는 동안 React는는 section, h1, image 태그에 대한 DOM 노드를 생성한다.
- 리렌더링하는 동안 React는 이전 렌더링 이후 변경된 속성을 계산한다.

## Step 3: React commits changes to the DOM

컴포넌트를 렌더링한 후에는 DOM을 수정한다.

- 초기 렌더링의 경우 React는 `appendChild()` DOM API를 사용하여 생성한 모든 DOM 노드를 화면에 표시한다.
- 리렌더링의 경우 React는 필요한 최소한의 작업을 적용하여 DOM이 최신 렌더링 출력과 일치하도록 한다.
