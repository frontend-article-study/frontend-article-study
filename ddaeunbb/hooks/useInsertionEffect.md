#📌 useInsertionEffect

- `useInsertionEffect`는 CSS-in-JS 라이브러리 작성자를 위한 훅입니다. 
- `useInsertionEffect`는 useEffect의 버전 중 하나로, DOM 변이 전에 실행됩니다.

### 개요
`useInsertionEffect(setup, dependencies?)`

### 참조
- `useInsertionEffect`를 호출하여 DOM 변이 전에 스타일을 주입합니다.

```JSX
import { useInsertionEffect } from 'react';

// Inside your CSS-in-JS library
// CSS-in-JS 라이브러리 내부에서
function useCSS(rule) {
  useInsertionEffect(() => {
    // ... inject <style> tags here ...
    // ... <style> 태그를 여기에 주입합니다.
  });
  return rule;
}
```

### 매개변수
- `setup` : Effect의 로직이 포함된 함수입니다. 셋업 함수는 클린업 함수를 선택적으로 반환할 수도 있습니다. 컴포넌트가 DOM에 추가되기 전에 React는 셋업 함수를 실행합니다. 변경된 의존성으로 다시 렌더링할 때마다 React는 먼저 이전 값으로 셋업 함수를 실행한 다음(제공한 경우) 새 값으로 셋업 함수를 실행합니다. 컴포넌트가 DOM에서 제거되기 전에 React는 클린업 함수를 한 번 더 실행합니다.
- `dependencies` : 의존성 목록은 일정한 수의 항목을 가져야 하며 [dep1, dep2, dep3]와 같이 인라인으로 작성해야 합니다. React는 Object.is 비교 알고리즘을 사용하여 각 의존성을 이전 값과 비교합니다. 의존성을 전혀 지정하지 않으면 컴포넌트를 다시 렌더링할 때마다 Effect가 다시 실행됩니다.

### 반환값
- undefined

### 주의사항
- `useInsertionEffect`가 실행될 때까지는 refs가 아직 첨부되지 않았고, DOM이 아직 업데이트되지 않았습니다.
- 위의 대한 이해가 되지 않는다면, 리액트 공식문서의 Synchronizing with Effects <a href="https://react.dev/learn/synchronizing-with-effects">Synchronizing with Effects</a>을 참고해주세요.

### 사용법
일부 팀은 CSS 파일을 작성하는 대신 JavsScript 코드에서 직접 스타일을 작성하는 것을 선호합니다. 이 경우 일반적으로 CSS-in-JS 라이브러리 또는 도구를 사용해야 합니다. CSS-in-JS에는 세 가지 일반적인 접근 방식이 있습니다.

- 컴파일러를 사용하여 CSS 파일로 정적 추출
- 인라인 스타일. 예: `<div style={{ opacity: 1 }}`>
- 런타임에 head에 `<style>` 태그 삽입

CSS-in-JS를 사용하는 경우 처음 두 가지 접근 방식(정적 스타일의 경우 CSS 파일, 동적 스타일의 경우 인라인 스타일)을 조합하여 사용하는 것이 좋습니다. 런타임 환경에서 `<style>` 태그 주입은 다음 두 가지 이유로 권장하지 않습니다.

1) 런타임 주입은 브라우저에서 스타일을 훨씬 자주 계산하도록 합니다.
2) 런타임 주입이 React 라이프사이클에서 잘못된 시점에 발생하면 속도가 매우 느려질 수 있습니다. (잦은 렌더링으로 인해)
<br/>

```JSX
// Inside your CSS-in-JS library
// CSS-in-JS 라이브러리 코드 내부에
let isInserted = new Set();
function useCSS(rule) {
  useInsertionEffect(() => {
    // As explained earlier, we don't recommend runtime injection of <style> tags.
    // 앞서 설명한 것처럼 <style> 태그의 런타임 주입은 권장하지 않습니다.
    // But if you have to do it, then it's important to do in useInsertionEffect.
    // 하지만 런타임 주입을 해야한다면, useInsertionEffect에서 수행하는 것이 중요합니다.
    if (!isInserted.has(rule)) {
      isInserted.add(rule);
      // head 안에 어떠한 룰을 추가하는 예시
      document.head.appendChild(getStyleForRule(rule));
    }
  });
  return rule;
}

function Button() {
  const className = useCSS('...');
  return <div className={className} />;
}
```

#### 좀더 쉬운 이해..
즉, 위의 예시는 렌더링이 되기 이전에 `useInsertionEffect` hook을 사용하는 것입니다.

아래는 이해를 돕기 위한 쉬운 예시입니다.

1. 이미 렌더링을 마친 컴포넌트가 있다.
2. style 태그를 강제로 head에 추가했다.
3. head에 추가한 스타일에 1번에 대한 높이값을 변화시키는 문법이 있다.
4. 1번 컴포넌트는 높이 값이 변화할때마다 state가 변화한다면?

-> 렌더링 되고 난 이후에 높이가 변화했으므로 또 렌더링하게 됩니다.

따라서, 렌더링 되기 이전에 style 태그를 추가하는 것이 좋습니다. 즉, 1번 이전에 style 태그를 추가하는 것이 좋습니다. ( =`useInsertionEffect` hook을 사용하면 됩니다.)