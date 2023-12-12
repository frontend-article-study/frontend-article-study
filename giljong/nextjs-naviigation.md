# 서론

Funnel 기능을 수행하는 커스텀 훅을 만들고 있는데 next.js의 useRouter에 훅이 의존하게되는 것이 마음에 들지 않아서

history api를 사용하는 형태로 구현해보았습니다.

그런데 history api는 url을 바꿔주는 역할을 수행할 뿐 상태를 리렌더링시키거나 하는 책임은 당연히 없습니다.

따라서 이런 현상이 발생합니다.

1. 개발자는 history api를 통해 url을 바꿉니다.

2. 당연히 history api는 url만 바꾸기에 url의 상태에 따라 화면이 달라지지 않습니다.

3. 그래서 useEffect를 이용해 popState에 대한 이벤트 핸들러를 달아주는 것을 통해 브라우저 스택 변경을 감지하고 상태를 변경합니다.

4. 잘 동작하지만 클린업함수에 의해 리스너가 제거된 뒤로 useEffect가 안붙어주면 동작하지않습니다.

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

저는 사실 바텀업형태로

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

# next.js navigation

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

    // __NA is used to identify if the history entry can be handled by the app-router.
    if (state.__NA) {
      window.location.reload()
      return
    }

    if (!state.__N) {
      return
    }

    // Safari fires popstateevent when reopening the browser.
    if (
      isFirstPopStateEvent &&
      this.locale === state.options.locale &&
      state.as === this.asPath
    ) {
      return
    }

    let forcedScroll: { x: number; y: number } | undefined
    const { url, as, options, key } = state
    if (process.env.__NEXT_SCROLL_RESTORATION) {
      if (manualScrollRestoration) {
        if (this._key !== key) {
          // Snapshot current scroll position:
          try {
            sessionStorage.setItem(
              '__next_scroll_' + this._key,
              JSON.stringify({ x: self.pageXOffset, y: self.pageYOffset })
            )
          } catch {}

          // Restore old scroll position:
          try {
            const v = sessionStorage.getItem('__next_scroll_' + key)
            forcedScroll = JSON.parse(v!)
          } catch {
            forcedScroll = { x: 0, y: 0 }
          }
        }
      }
    }
    this._key = key

    const { pathname } = parseRelativeUrl(url)

    // Make sure we don't re-render on initial load,
    // can be caused by navigating back from an external site
    if (
      this.isSsr &&
      as === addBasePath(this.asPath) &&
      pathname === addBasePath(this.pathname)
    ) {
      return
    }

    // If the downstream application returns falsy, return.
    // They will then be responsible for handling the event.
    if (this._bps && !this._bps(state)) {
      return
    }

    this.change(
      'replaceState',
      url,
      as,
      Object.assign<{}, TransitionOptions, TransitionOptions>({}, options, {
        shallow: options.shallow && this._shallow,
        locale: options.locale || this.defaultLocale,
        // @ts-ignore internal value not exposed on types
        _h: 0,
      }),
      forcedScroll
    )
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

packages.next/src/client

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

https://github.com/vercel/next.js/blob/canary/packages/next/src/client/components/navigation.ts

# 참고

https://developer.mozilla.org/en-US/docs/Web/API/History_API // mdn의 historyapi

https://developer.mozilla.org/en-US/docs/Web/API/History // mdn의 history

https://www.daleseo.com/js-history-api/ // 달레님의 history api 설명

https://github.com/vercel/next.js // next.js github repository
