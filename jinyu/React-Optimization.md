# 리액트 최적화

### 리액트의 성능 및 최적화

리액트에서 성능은 다음 두 가지 방식으로 생각할 수 있습니다.

1. Loading Performance - Compressing & loading code/assets (mostly *Non-React* things)
2. Runtime Performance - CPU & rendering issues (mostly *React-specific* things)

### 로딩과 런타임

로딩 성능

로딩 성능은 사용자가 사이트에 접속할 때 콘텐츠가 로드되는 속도를 측정한 것입니다. 몇몇 특정 측정 항목으로는 First Contentful Paint (FCP), Largest Contentful Paint (LCP), First Input Delay (FID), TTI (Time to Interactive), Speed Index(속도 지수)이 있습니다.

런타임 성능

초기 로드 후 애플리케이션이 얼마나 "부드럽게" 실행되고 작동하는지를 측정한 것입니다. 이에 대한 일부 특정 측정항목으로는 "프레임 속도", "CPU" 및 "메모리 사용량"이 있습니다.

### 로드 시간 측정 (Measuring Loadtimes)

아래의 웹 사이트들과 도구들을 이용해 웹 페이지의 로드 시간을 측정할 수 있습니다.

- **[PageSpeed Insights](https://pagespeed.web.dev/)**
- **[WebPageTest.org](https://webpagetest.org/)**
- **[Lighthouse](https://developer.chrome.com/docs/lighthouse/overview/)**
- **[UnLightHouse](https://unlighthouse.dev/)**

### 로드 시간 최적화 (\***\*Optimizing Loadtimes)\*\***

네트워크를 통해 가능한 한 적은 코드/미디어를 전송하고 모든 것을 최적화하세요.

- 서버 측에서 GZip 압축을 활용하여 모든 인플라이트(in-flight) HTTP 요청을 압축합니다.
- 번들에 포함된 모든 이미지와 동영상 최적화
  - 이미지 지연 로드
- 모든 프로젝트 자산(assets)을 CSS / 자바스크립트로 빌드(Building) 및 축소(Minifying)
- 그 외 다양한 방법
  - 코드 분할 - Vite 번들에서는 기본적으로 일부 코드 분할이 수행됩니다.
  - 지연 로딩과 Suspense를 사용해 React 앱에서 컴포넌트나 라우트를 지연 로드하는 방법
- 서버측 렌더링(SSR)과 정적 사이트 생성(SSG)은 브라우저가 페이지를 더 빠르게 렌더링하고 검색 엔진에서 접근할 수 있도록 해주므로 퍼스트 콘텐츠 페인트와 인터랙티브 시간 메트릭을 개선할 수 있습니다.
  - 기존 프로젝트에 SSR/SSG를 추가하는 것은 쉬운 일이 아닙니다. 애플리케이션이 이러한 기술을 통해 이점을 얻을 수 있는지 미리 평가하여 처음부터 올바른 방식으로 프로젝트를 구성할 수 있도록 하세요.

### 성능 측정 ( Measuring Runtimes)

모던 리액트는 기본적으로 빠른 편입니다. 복잡한 컴포넌트 혹은 기능을 작성해 애플리케이션이 느리게 동작하는 것을 발견한 것이 아니라면 최적화 전략을 사용할 필요가 없습니다. 만약 React 앱에서 성능 문제가 발생한다면 렌더링 문제(너무 느리거나 너무 많음)일 가능성이 큽니다.

리액트 렌더링은 재귀적입니다. 렌더링 성능 최적화는 현재 병목 현상을 찾아서 해결하는 것으로 귀결됩니다. 더 빠르게 만들거나 메모를 사용하여 렌더링을 건너뛰는 방법이 있습니다.

다음은 React 앱에서 런타임 성능을 측정하는 방법을 이해하는 데 도움이 되는 기타 유용한 레퍼런스입니다.

- **[Measuring Performance in Chrome](https://developer.chrome.com/docs/devtools/performance/)**
- **[Debugging in the Browser](https://javascript.info/debugging-chrome)**
- **[Devtools Coverage](https://developer.chrome.com/docs/devtools/coverage/)** - shows how much of your code is being shipped but then NOT running (while session is recording)
- Use the **[Profiler](https://react.dev/reference/react/Profiler)** to measure rendering performance of a React tree programatically
- **[How to Detect Slow Renders in React?](https://alexsidorenko.com/blog/react-performance-slow-renders/)**
- **[How many re-renders are too many?](https://alexsidorenko.com/blog/react-how-many-rerenders/)**

### 런타임 최적화 (Optimizing Runtimes)

런타임 성능 문제는 일반적으로 두 가지 유형의 문제로 요약됩니다.

1. Fixing Slow Renders
   - 리렌더링 현상으로 인해 성능이 떨어지는 것을 고치기 전에 렌더링이 늦게 되는 컴포넌트가 없는지를 먼저 찾고 느린 렌더링을 빠르게 고치는 것이 좋습니다.
   - ES6는 단일 스레드이므로 지연 동작이 쉽게 발생할 수 있음을 이해해야 합니다.
     - 렌더링을 차단(block)해서는 안 되는 고비용(expensive) 작업을 웹 워커로 오프로드하세요. (?)
2. Fixing Profuse Re-Renders
   - 느린 렌더링 현상을 수정하고나서 과도하게 리렌더링되는 현상을 수정합니다.
   - 리렌더링을 유발해서는 안 되는(또는 전혀 렌더링되지 않는) 상태 및 값에는 Ref를 사용하세요.
   - Effect, Event Handlers, conditions 외부에서 상태를 설정할 때 주의하세요.

메모이제이션(Memoization) 및 가상화(Virtualization)는 느린 렌더링이나 많은 양의 리렌더링을 다양한 방식으로 해결할 수 있습니다:

- 메모(memo)를 사용하여 컴포넌트를 메모하고 Prop이 변경되지 않을 때 하위 컴포넌트의 불필요한 재렌더링을 줄입니다.
  - Context Provider 아래의 모든 컴포넌트에 대해서는 `memo()` 를 사용하는 것이 좋습니다.
  - [시각적 예제](https://codesandbox.io/s/a-visual-guide-to-react-rendering-sandbox-td70u?file=/src/sandbox.jsx)를 통해 memo()가 실제로 작동하는 모습 보기
- `useMemo()` 훅을 이용해 컴포넌트 내부에서 일어나는 특정한 작업을 메모하고, 해당 작업이 다시 수행될 필요가 없다면 건너뛰어 렌더링하는 시간을 단축할 수 있습니다.
  - **[A Visual Guide to React Rendering - useMemo](https://alexsidorenko.com/blog/react-render-usememo/)**
- 컴포넌트에서 사용하기 위해 생성된 **함수**를 메모화(캐시)하고 함수를 다시 생성할 필요가 없는 경우 건너뛰어 렌더링 시간을 단축하는 `useCallback()`을 사용합니다.
  - **[A Visual Guide to React Rendering - useCallback](https://alexsidorenko.com/blog/react-render-usecallback/)**
- [가상화](https://www.patterns.dev/posts/virtual-lists)를 사용하여 대규모 데이터 세트 목록(large lists of datasets)을 효율적으로 렌더링하고 렌더링 시간을 단축합니다.
  - 가상화?
    리스트 가상화(List Virtualization) 또는 윈도잉이라 부르기도 하며 전체 리스트 중에서 보이는 엘리먼트만 렌더링을 하고, 보이지 않는 엘리먼트는 DOM 트리에서 빼버리는 기법입니다.
    ![[리디 - 예상보다 24배 많은 콘텐츠에 프론트가 대처하는 방법](https://ridicorp.com/story/ridi-markdown-improvements/#03_1_gzip)](./assets/react-optimization-ridi.png)
    [리디 - 예상보다 24배 많은 콘텐츠에 프론트가 대처하는 방법](https://ridicorp.com/story/ridi-markdown-improvements/#03_1_gzip)
- [Million.js](https://million.dev/)라는 라이브러리가 최근 유행하고 있는데, 이 라이브러리는 React의 VDOM을 약 70% 더 빠른 버전으로 대체할 수 있다고 합니다.
  - Million.js?
    리액트에서 상태를 가진 UI 컴포넌트를 변경을 할 때, 새 상태와 이전 상태의 차이(”diffing” 프로세스)를 계산하고 업데이트된 가상 DOM과 동기화하므로서 실제 DOM에 변경 사항을 적용합니다.
    이 “diffing”은 트리 크기에 따라 달라지며 궁극적으로 가상 DOM의 병목 현상이 발생합니다. 즉, 노드가 많을수록 비교하는데 더 많은 시간이 걸립니다.
    ### The Block Virtual DOM
    2022년에 [Blockdom](https://github.com/ged-odoo/blockdom)이 출시되었습니다. 블록 가상 DOM은 diffing에 대해 다른 접근 방식을 취하며 두 부분으로 나눌 수 있습니다:
    1. 정적분석(Static Analysis): 가상 DOM은 트리의 동적 부분을 “Edit Map” 또는 가상 DOM의 동적 부분을 상태에 대한 “편집(edit)” 목록으로 추출합니다.
    2. 더티 체킹(Dirty Checking): 변경된 내용을 확인하기 위해 가상 DOM 트리가 아닌 상태가 달라집니다. 상태가 변경된 경우 Edit Map을 통해 DOM이 직접 업데이트 됩니다.
       Million.js는 Blockdom과 유사한 접근 방식으로 가상 DOM을 최적화합니다.
       [예제로 직접 살펴보기](https://million.dev/blog/virtual-dom#counter-example)
    ### The Block Virtual DOM 단점
    - 동적 컨텐츠가 적고 정적 컨텐츠가 많을 때 적합합니다. → 많이 변경되지 않는 UI 트리에 적합합니다.
    - 정적 컨텐츠가 많은 사이트를 구축하는 경우에 적합 → 많지 않을 경우 큰 차이를 못 느낀다고 합니다.

### 요약

리액트에서 성능을 최적화하는 방법은 크게 2가지가 있습니다.

- 로드 시간 최적화
- 런타임 시간 최적화

로드 시간 최적화에서는 코드 스플리팅, 지연 로딩, 번들 사이즈 축소, 이미지 및 동영상 최적화로 네트워크를 통해서 전송되는 코드/미디어 데이터를 최적화합니다. 또한 SSR, SSG를 사용해 TTV(Time to View)를 개선해 사용자에게 더 빨리 페이지를 보여주는 방법도 있습니다.

런타임 시간 최적화에서는 컴포넌트의 느린 렌더링 속도를 먼저 개선하고 과도하게 리렌더링이 발생하는 컴포넌트를 수정해 최적화합니다. 이 과정에서 메모이제이션 & 가상화를 사용하면 다양한 방법으로 애플리케이션을 최적화할 수 있습니다.

### 레퍼런스

https://reacthandbook.dev/react-performance-optimization

https://vercel.com/blog/how-react-18-improves-application-performance

[https://velog.io/@shin6403/React-렌더링-성능-최적화하는-7가지-방법-Hooks-기준](https://velog.io/@shin6403/React-%EB%A0%8C%EB%8D%94%EB%A7%81-%EC%84%B1%EB%8A%A5-%EC%B5%9C%EC%A0%81%ED%99%94%ED%95%98%EB%8A%94-7%EA%B0%80%EC%A7%80-%EB%B0%A9%EB%B2%95-Hooks-%EA%B8%B0%EC%A4%80)

https://wikidocs.net/197788

https://cocoder16.tistory.com/36

https://uzihoon.com/post/ef453fd0-ab14-11ea-98ac-61734eebc216

[가상화 한글번역](https://patterns-dev-kr.github.io/performance-patterns/list-virtualization/)

[리디 - 예상보다 24배 많은 콘텐츠에 프론트가 대처하는 방법](https://ridicorp.com/story/ridi-markdown-improvements/#03_1_gzip)

공식문서

https://react.dev/blog/2022/06/15/react-labs-what-we-have-been-working-on-june-2022#static-server-rendering-optimizations

https://react.dev/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-optimizing-compiler

https://react.dev/reference/react/useContext#optimizing-re-renders-when-passing-objects-and-functions

https://react.dev/reference/react/useCallback#optimizing-a-custom-hook
