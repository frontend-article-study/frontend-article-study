# 서론

Funnel 기능을 수행하는 커스텀 훅을 만들고 있는데 next.js의 useRouter에 훅이 의존하게되는 것이 마음에 들지 않아서

history api를 사용하는 형태로 구현해보았습니다.

그런데 history api는 url을 바꿔주는 역할을 수행할 뿐 상태를 리렌더링시키거나 하는 책임은 당연히 없습니다.

따라서 이런 식으로 코드를 짜게됩니다.

1. 개발자는 history api를 통해 url을 바꿉니다.

2. 당연히 history api는 url만 바꾸기에 url의 상태에 따라 화면이 달라지지 않습니다.

3. 그래서 useEffect를 이용해 popState에 대한 이벤트 핸들러를 달아주는 것을 통해 브라우저 스택 변경을 감지하고 상태를 변경합니다.


```tsx
React.useEffect(() => {
  const handlePopState = () => {
    const nowStep = safeSearchParams(targetKey) ?? '';
    setCurrentStep(nowStep);
  };
  window.addEventListener('popstate', handlePopState);
  return () => {
    window.removeEventListener('popstate', handlePopState);
  };
}, []);
```

예컨대 이런 코드가 필요하다는 것입니다.

history가 스택이 변한다고 해서 상태가 변하는것이 아니기 때문에

popstate 이벤트가 발생하면 상태를 적절히 업데이트하는 형태로 spa를 구현하게된다는 것이에요

# 그러면 next.js는 어떻게 라우터를 구현해서 spa를 구현하고 있는걸까?

저는 사실 바텀업형태로 봤지만 간략하게 다루는데에는 탑다운 형태가 더 편할 것 같아서

거의 최상단의 코드부터 가져와봤습니다.

아래 코드는 app-router.tsx입니다.

packages/next/src/client/components/app-router.tsx

```tsx
return (
  <>
    <HistoryUpdater appRouterState={useUnwrapState(reducerState)} sync={sync} />
    <PathnameContext.Provider value={pathname}>
      <SearchParamsContext.Provider value={searchParams}>
        <GlobalLayoutRouterContext.Provider
          value={{
            buildId,
            changeByServerResponse,
            tree,
            focusAndScrollRef,
            nextUrl,
          }}
        >
          <AppRouterContext.Provider value={appRouter}>
            <LayoutRouterContext.Provider
              value={{
                childNodes: cache.parallelRoutes,
                tree,
                // Root node always has `url`
                // Provided in AppTreeContext to ensure it can be overwritten in layout-router
                url: canonicalUrl,
              }}
            >
              {content}
            </LayoutRouterContext.Provider>
          </AppRouterContext.Provider>
        </GlobalLayoutRouterContext.Provider>
      </SearchParamsContext.Provider>
    </PathnameContext.Provider>
  </>
);
```

리턴되는 JSX 코드만 보아도 next.js의 구현을 대략적으로 예측할 수 있습니다.

1. next.js가 제공하는 기능들은 그냥 ContextApi로 구현되어있다.

2. pathname, searchParams 와 같은 이름을 가진 컨텍스트들이 있는것으로 미루어보아

브라우저 히스토리 / url과 관련된 로직 역시 context api를 통하여 관리하고있다

3. 히스토리 / url을 리액트 상태와 동기화하고 있는 곳이 어딘가에는 존재할 것이다.

라는 것을 예상할 수 있습니다.

그 전에 AppRouterContext.Provider에 들어가는 appRouter 밸류가 뭔지 궁금하니까

그거만 살짝 봐보겠습니다.

```tsx
const appRouter = useMemo<AppRouterInstance>(() => {
  const routerInstance: AppRouterInstance = {
    back: () => window.history.back(),
    forward: () => window.history.forward(),
    prefetch: (href, options) => {
      // Don't prefetch for bots as they don't navigate.
      // Don't prefetch during development (improves compilation performance)
      if (
        isBot(window.navigator.userAgent) ||
        process.env.NODE_ENV === 'development'
      ) {
        return;
      }
      const url = new URL(addBasePath(href), window.location.href);
      // External urls can't be prefetched in the same way.
      if (isExternalURL(url)) {
        return;
      }
      startTransition(() => {
        dispatch({
          type: ACTION_PREFETCH,
          url,
          kind: options?.kind ?? PrefetchKind.FULL,
        });
      });
    },
    replace: (href, options = {}) => {
      startTransition(() => {
        navigate(href, 'replace', options.scroll ?? true);
      });
    },
    push: (href, options = {}) => {
      startTransition(() => {
        navigate(href, 'push', options.scroll ?? true);
      });
    },
    refresh: () => {
      startTransition(() => {
        dispatch({
          type: ACTION_REFRESH,
          origin: window.location.origin,
        });
      });
    },
    // @ts-ignore we don't want to expose this method at all
    fastRefresh: () => {
      if (process.env.NODE_ENV !== 'development') {
        throw new Error(
          'fastRefresh can only be used in development mode. Please use refresh instead.',
        );
      } else {
        startTransition(() => {
          dispatch({
            type: ACTION_FAST_REFRESH,
            origin: window.location.origin,
          });
        });
      }
    },
  };

  return routerInstance;
}, [dispatch, navigate]);
```

아하! appRouter 변수는 그저 히스토리 api를 리액트 상태와 엮어둔 것이라는것을 예측할 수 있습니다.

위에서 보이는 navigate() 함수역시 리액트 상태를 변경시키는 dispatch 함수를 반환하는 함수거든요

# 아하.. next.js 는 context api로 상태를 관리하는구나

라고 끝나면 시시할 것 같죠?

리액트 상태랑 url을 어떻게 동기화하고 있을지를 찾아가봅시다.

아래 코드는 넥스트가 사용하는 Router 클래스의 구현입니다.

constructor 부분을 보면 브라우저 뒤로가기 ,앞으로가기에 대응할 수 있는

popstate 이벤트 리스너를 달아주는 것을 볼 수 있습니다.

<br/>
** 토막상식 : 이벤트리스너는 여러개를 달면 단 순서대로 모든 리스너가 동작합니다. **

packages/next/src/client/router.ts

```tsx
export default class Router implements BaseRouter {
    ...대충 많은 코드
    constructor(...대충 매개변수){
        window.addEventListener('popstate', this.onPopState)
    }
    onPopState = (e: PopStateEvent): void => {
    const { isFirstPopStateEvent } = this
    this.isFirstPopStateEvent = false

    const state = e.state as HistoryState

    if (!state) {
      // We get state as undefined for two reasons.
      //  1. With older safari (< 8) and older chrome (< 34)
      //  2. When the URL changed with #
      //
      // In the both cases, we don't need to proceed and change the route.
      // (as it's already changed)
      // But we can simply replace the state with the new changes.
      // Actually, for (1) we don't need to nothing. But it's hard to detect that event.
      // So, doing the following for (1) does no harm.
      const { pathname, query } = this
      this.changeState(
        'replaceState',
        formatWithValidation({ pathname: addBasePath(pathname), query }),
        getURL()
      )
      return
    }
    // ... 대충 엄청 많은 구현이 포함되어있지만 분량 상 제거합니다.
    // 궁금하신분은 직접 해당 파일을 열어보세요
  }
}
```

packages/next/src/client/router.ts

```tsx
export function createRouter(
  ...args: ConstructorParameters<typeof Router>
): Router {
  singletonRouter.router = new Router(...args);
  singletonRouter.readyCallbacks.forEach((cb) => cb());
  singletonRouter.readyCallbacks = [];

  return singletonRouter.router;
}
```

그러면 저렇게 만들어진 라우터는 어떻게 쓰일까요?

동일한 파일에서 createRouter 함수를 정의하고 내보내는것을 확인할 수 있습니다.

ide 내부에서 이 Router 클래스의 참조 상태를 확인해보면

마치 싱글톤처럼 관리되는 것을 볼 수 있는데

오직 createRouter 함수에 의해서만 Router 클래스를 생성하는데

이 createRouter 함수는 전체 애플리케이션을 통틀어 단 한번만 호출되는 것을 확인할 수 있어요

당연히 한번만 써줘야할 것 같긴하지만... 아무튼 그렇습니다.

packages/next/src/client

```tsx
export let router: Router;
router = createRouter(initialData.page, initialData.query, asPath, {
  initialProps: initialData.props,
  pageLoader,
  App: CachedApp,
  Component: CachedComponent,
  wrapApp,
  err: initialErr,
  isFallback: Boolean(initialData.isFallback),
  subscription: (info, App, scroll) =>
    render(
      Object.assign<
        {},
        Omit<RenderRouteInfo, 'App' | 'scroll'>,
        Pick<RenderRouteInfo, 'App' | 'scroll'>
      >({}, info, {
        App,
        scroll,
      }) as RenderRouteInfo,
    ),
  locale: initialData.locale,
  locales: initialData.locales,
  defaultLocale,
  domainLocales: initialData.domainLocales,
  isPreview: initialData.isPreview,
});
```

앞서 이야기한바와 같이 단 한번만 호출하는 부분이 궁금하다면 다음 파일에서 확인할 수 있는데요

애플리케이션의 초기값과 앱시작을 위한 값들을 넘겨주어 router를 생성하는 것을 볼 수 있습니다.



# 결론

첫 시작은 그냥 funnel 훅이 만들고 싶었을 뿐이었는데

만드는 과정에서 쿼리스트링 조작이 필요했고.. 

근데 프레임워크에 종속되는건 싫어서 직접 history api를 쓰려고했고..

근데 바닥부터 spa를 만들어본 경험이 없다보니 시행착오를 겪었고

그 과정에서 ?? 아니 이거 이런 방식으론 구현 못하겠는데 next.js 같은것들은 어떻게 구현을 한거지??

라는 궁금증이 생겨서 가볍게 찾아본 결과 다음과 같은 인사이트를 얻을 수 있었습니다.



# 참고

https://developer.mozilla.org/en-US/docs/Web/API/History_API // mdn의 historyapi

https://developer.mozilla.org/en-US/docs/Web/API/History // mdn의 history

https://www.daleseo.com/js-history-api/ // 달레님의 history api 설명

https://github.com/vercel/next.js // next.js github repository

https://github.com/vercel/next.js/blob/canary/packages/next/src/client/components/navigation.ts
