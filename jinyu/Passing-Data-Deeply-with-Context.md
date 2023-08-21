# Passing Data Deeply with Context

> author: 이진유<br/>date: 2023/08/07 (sun)

일반적으로 리액트에서는 부모 컴포넌트에서 자식 컴포넌트로 `props`를 통해 정보를 전달합니다. 하지만 중간에 여러 컴포넌트를 거쳐야 하거나 앱의 여러 컴포넌트가 동일한 정보를 필요로 하는 경우 `props`를 전달하면 가독성이 떨어지며 불편해질 수 있습니다.

<figure style="background-color:#23272f">
<figcaption style="text-align:center; padding: 1rem; color:#d9d9d9">Props Drilling</figcaption>
<img src="https://react.dev/_next/image?url=%2Fimages%2Fdocs%2Fdiagrams%2Fpassing_data_prop_drilling.dark.png&w=640&q=75" alt="Props drilling">
</figure>

위의 그림처럼 최상단 부모의 컴포넌트에서 최하위 자식 컴포넌트로 `props`를 전달해줄 때 여러 컴포넌트를 거치면서 전달해줍니다. 그 과정에서 해당 `props`가 필요없는 컴포넌트도 있어 불필요한 전달 과정도 있고, 전달되어야할 `props`가 많고 깊이가 깊어질수록 가독성도 떨어지며 코드를 작성하는 개발자 입장에서도 많은 불편함을 경험하게 됩니다.

여러 컴포넌트를 거치지 않고 딱 데이터가 필요한 컴포넌트에서 데이터를 받아올 수 있는 방법이 있다면 좋지 않을까요?

그래서 React에서는 `Context` 기능을 제공하고 있습니다.

`Context`를 사용하면 부모 컴포넌트가 `props`를 통해 명시적으로 전달하지 않고도 깊이 여부와 무관하게 그 아래 트리의 모든 컴포넌트에서 일부 정보를 사용할 수 있습니다.

Context를 사용하는 방법은 다음과 같습니다:

1. context를 생성합니다.
2. 데이터가 필요한 컴포넌트에서 해당 context를 사용합니다.
3. 데이터를 지정하는 컴포넌트에서 해당 context를 제공합니다.

예제 코드로 살펴볼까요? 예제 코드는 리액트의 공식문서의 코드를 가져왔습니다.

## Step 1: Context 만들기

살펴볼 예제는 컴포넌트의 깊이에 따라 Heading 값을(`h1`~`h6`) 동적으로 변경하는 예제입니다.

먼저 데이터를 담을 context를 만들어야 합니다. 그리고 해당 데이터가 필요한 컴포넌트가 사용할 수 있도록 `export` 해줍니다.

```jsx
// LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

## Step 2: Context 사용하기

우선 깊이 별 `level`값을 담을 `Section` 컴포넌트를 선언하겠습니다.

```jsx
// Section.js
export default function Section({ children }) {
  return <section className="section">{children}</section>;
}
```

그리고나서 해당 `level`값을 사용할 `Heading` 컴포넌트를 선언하겠습니다.

```jsx
// Heading.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Unknown level: ' + level);
  }
}
```

그리고 가장 최상단 컴포넌트인 `App.js`에서 다음과 같이 코드를 작성해줍니다.

```jsx
// App.js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section level={1}>
      <Heading>Title</Heading>
      <Section level={2}>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section level={3}>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section level={4}>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

아직 context를 제공하지 않았으므로 위의 코드는 동작하지 않습니다.

## Step 3: Context 제공하기

기존에 선언했던 `Section` 컴포넌트에 context provider로 감싸 `LevelContext`를 제공하겠습니다.

```jsx
// Section.js
import { LevelContext } from './LevelContext.js';

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext.Provider value={level}>{children}</LevelContext.Provider>
    </section>
  );
}
```

여기까지 봤을 때 `App.js`에서 `Section` 컴포넌트가 가지고 있는 `level` 데이터를 필요한 곳에 적재적소로 사용하고 있습니다.

이처럼 context는 프로젝트 전역에서 사용하는 것보다 부분적으로 필요한 곳에서 스코프를 정해서 **부분전역**처럼 사용하는 것이 좋습니다.

하지만 위의 코드를 봤을 때 `level`를 수동으로 값을 할당하고 있습니다. 그렇다면 처음에 말했듯이 동적으로 깊이에 따라 `level`을 적용하고 싶을 때는 어떻게 해야 할까요?

`Context`를 제공하는 `Section` 컴포넌트에서 부모 `Section` 컴포넌트의 `level` 데이터를 가지고와서 `1`을 더해주면 되지 않을까요?

```jsx
// Section.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext.Provider value={level + 1}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

이제 `Section` 컴포넌트에서 `level` 데이터를 받을 필요가 없으니 `App.js`에서 `level`을 모두 지워주고, `LevelContext`의 초기값을 `0`으로 설정해줍니다.

```jsx
// App.js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading>Title</Heading>
      <Section>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

```jsx
// LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(0);
```

아직까지 Context의 동작 방식에 대해 어려움이 느껴지시나요? 리액트 공식문서에는 CSS 속성 상속을 떠올리면 이해하기 더욱 쉽다고 합니다.

> CSS에서는 `<div>`에 color: blue을 지정할 수 있으며, 중간에 다른 DOM 노드가 color: green으로 재정의하지 않는 한 그 안에 있는 모든 DOM 노드는 아무리 깊어도 그 색을 상속받습니다. 마찬가지로 React에서 위에서 오는 context를 재정의하는 유일한 방법은 children을 다른 값으로 context provider로 감싸는 것입니다.<br /><br /> CSS에서는 color 및 background-color와 같은 서로 다른 속성이 서로 재정의되지 않습니다. background-color에 영향을 주지 않고 모든 `<div>`의 color를 빨간색으로 설정할 수 있습니다. 마찬가지로 서로 다른 React context도 서로 재정의하지 않습니다. createContext()로 만드는 각 context는 다른 context와 완전히 분리되어 있으며, 특정 context를 사용하거나 제공하는 컴포넌트를 함께 묶습니다. 하나의 컴포넌트가 문제없이 다양한 context를 사용하거나 제공할 수 있습니다.

## Context를 사용하기 전에

1. `props` 전달로 시작하세요. `props drilling`이 꼭 나쁜 것으로 인식될 수 있는데 전혀 그렇지 않습니다. 리액트를 사용하다보면 `props drilling`은 당연히 일어나는 것입니다. `props`를 전달할 깊이가 깊지가 않다면 어떤 컴포넌트가 어떤 데이터를 사용하는지 명확해지기 때문에 오히려 `props drilling`이 가독성에 도움이 될 경우도 있습니다.
2. 컴포넌트를 추출하고 JSX를 `children`으로 전달하세요. 예를 들어, posts과 같은 데이터 props를 직접 사용하지 않는 시각적 컴포넌트에 `<Layout posts={posts} />` 와 같은 방법 대신, Layout이 children을 prop으로 사용하도록 만들고 `<Layout><Posts posts={posts} /></Layout>`를 렌더링합니다. 이렇게 하면 데이터를 지정하는 컴포넌트와 데이터를 필요로 하는 컴포넌트 사이의 레이어 수가 줄어듭니다.

## Context 사용 사례

다크모드 등과 같은 테마, 로그인 유지, 라우팅, 상태 관리 등에 사용

## 요약

- 멀리 떨어진 컴포넌트에서 일부 정보가 필요한 경우 context가 도움이 된다.
- Context를 너무 남용하지 말라. Context를 Context의 데이터가 변경되었을 때 의존하고 있는 모든 컴포넌트가 리렌더링된다. 그러므로 다크테마, 로그인 유지 등과 같은 변경 사항이 거의 일어나지 않는 경우에만 전역 상태로 관리해라.
- 부분 전역을 사용할 때 매우 유용하다.
