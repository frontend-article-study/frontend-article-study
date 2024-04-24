# typed routes 

nextjs 공식문서를 살펴보면 다음과 같이 [next.js typed routes](https://nextjs.org/docs/app/api-reference/next-config-js/typedRoutes) typed routes에 대한 experimental 을 제공하고 있습니다.

해당 작업은 [FIX NEXT.JS ROUTING TO HAVE FULL TYPE-SAFETY](https://www.flightcontrol.dev/blog/fix-nextjs-routing-to-have-full-type-safety)라는 아티클에서 영감을 받았습니다.

그러나 해당 코드가 바로 동작하지 않는 문제가 있었고 TypedRoutes의 경우에는 상황에 따라 느슨하게 타입을 써야하는 경우가 있음에도 불구하고 너무 타입을 좁혀버리는 문제가 있었습니다.

또한 app router로 들어오면서 기존 useRouter가 제공하던 편의 기능들이 많이 사라지고 대신 searchParams , Params와 같은 형태로 router의 기능이 파편화되는 구조적인 변경이 일어났습니다.

그래서 해당 변화가 상당히 마음에 들지 않는다고 생각하여 useRouter에 편의기능들을 넣고 추상화하는 작업도 덤으로 수행했습니다.


<img src="../asset/full-type-routes-1.jpeg">

```tsx
import * as qs from "qs";
import { useInternalRouter } from "./use-internal-router";

type DefaultQuery = Record<string, any>;
type DefaultPathname = Array<[string, any]>;
type DefaultRouterType = {
  query?: DefaultQuery;
  pathname?: DefaultPathname;
  catchAll?: string;
};
type TupleArrayToRecord<T extends Array<[string, any]>> = {
  [K in T[number][0]]: Extract<T[number], [K, any]>[1];
};
type ExtractPathnameValue<T extends DefaultPathname> = {
  [K in keyof T]: T[K] extends [any, infer V] ? V : never;
};

type RoutesString = `/${string}`;
export type RoutesQueryAndPath<T extends DefaultRouterType = DefaultRouterType> = {
  query: T["query"] extends DefaultQuery ? Partial<T["query"]> : DefaultQuery;
  pathname: T["pathname"] extends DefaultPathname ? TupleArrayToRecord<T["pathname"]> : Record<string, any>;
  pathnameValue: T["pathname"] extends DefaultPathname ? ExtractPathnameValue<T["pathname"]> : string[];
  pathnameTuple: T["pathname"] extends DefaultPathname ? T["pathname"] : DefaultPathname;
  pathnameCatchAll: T["catchAll"] extends string ? Record<T["catchAll"], string[]> : Record<string, string[]>;
  arg: {
    query?: T["query"] extends DefaultQuery ? Partial<T["query"]> : DefaultQuery;
    pathname?: T["pathname"] extends DefaultPathname ? ExtractPathnameValue<T["pathname"]> : string[];
  };
};

export const parseSearchParams = <T extends Record<string, unknown>>(param?: string): T => {
  return qs.parse(param ? param.replace(/^\?/, "") : "") as T;
};

export const stringfySearchParams = (obj?: Record<string, unknown>): string => {
  return qs.stringify(obj ?? {}, { arrayFormat: "repeat" });
};

export const stringfyPathname = (str?: string[]) => {
  return str?.join("/") ?? "";
};

export const createInternalPath = <
  E extends Pick<RoutesQueryAndPath, "pathnameValue" | "query">,
  T extends RoutesString = RoutesString,
>(
  basePath: T,
  option?: { query?: E["query"]; pathname?: E["pathnameValue"] },
) => {
  const path = stringfyPathname(option?.pathname);
  const query = stringfySearchParams(option?.query);
  const pathSlash = path.length > 0 ? "/" : "";
  const questionmark = query.length > 0 ? "?" : "";
  return `${basePath}${pathSlash}${path}${questionmark}${query}`;
};

export const createRoutes = <T extends RoutesQueryAndPath = RoutesQueryAndPath>(basePath: RoutesString) => ({
  path: (arg?: T["arg"]) => createInternalPath(basePath, arg),
  useRouter: () => useInternalRouter<T>(),
});

```

먼저 queryString을 편리하게 선언적으로 다루고 싶었습니다.

살짝 짚고 갈 지점이 있는데 queryString을 stringify, parse하는 작업은 상당히 만만치않은 작업입니다.

queryString의 밸류로 중첩객체, 배열이 들어가게되는 경우를 고려해야하기 때문이에요

이런 경우를 모두 고려하는 유틸함수를 짜는건 매우 힘듭니다.

그래서 qs라는 라이브러리를 통해 파싱을 수행하면서 라이브러리에 직접 의존하기보다는 한번의 추상화를 거치겠습니다.

또한 이후 qs라는 라이브러리를 query-string 라이브러리로 전환하거나 또는 모종의 코드변경이 필요할 수 있으니

지금 시점에서 잘 동작하는 지를 검증하는 테스트코드를 작성하면 좋을 것입니다.

```tsx
import { parseSearchParams, stringfySearchParams, stringfyPathname, createInternalPath } from "./router.util";
describe("parseSearchParams 테스트", () => {
  it("쿼리스트링 문자열을 객체로 변환할 수 있습니다.", () => {
    const parse = parseSearchParams<{ test: string }>("test=hello");
    expect(parse).toEqual({ test: "hello" });
  });

  it("물음표를 제거합니다.", () => {
    const parse = parseSearchParams<{ test: string }>("?test=hello");
    expect(parse).toEqual({ test: "hello" });
  });
  it("물음표는 단 한번만 제거합니다..", () => {
    const parse = parseSearchParams<{ "?test": string }>("??test=hello");
    expect(parse).toEqual({ "?test": "hello" });
  });

  it("물음표 문자열의 맨 앞에 온 경우만 삭제합니다.", () => {
    const parse = parseSearchParams<{ test: string }>("test=hello?");
    expect(parse).toEqual({ test: "hello?" });
  });

  it("&를 통해 구분된 쿼리스트링을 적절하게 파싱합니다.", () => {
    const parse = parseSearchParams<{ test: string; hi: string }>("test=hello&hi=val");
    expect(parse).toEqual({ test: "hello", hi: "val" });
  });

  it("?가 앞에 붙은 쿼리스트링도 적절하게 &를 기준으로 파싱합니다.", () => {
    const parse = parseSearchParams<{ test: string; hi: string }>("?test=hello&hi=val");
    expect(parse).toEqual({ test: "hello", hi: "val" });
  });
  it("동일한 키가 들어오는 경우를 잘 파싱합니다.", () => {
    expect(parseSearchParams("he=hello&he=world")).toEqual({ he: ["hello", "world"] });
  });
  it("중첩 객체여야하는 경우를 잘 파싱합니다.", () => {
    expect(parseSearchParams("he%5Bshe%5D=hello")).toEqual({ he: { she: "hello" } });
  });
});

describe("objToSearchParams Test", () => {
  it("평범한 문자열 밸류를 잘 변환합니다.", () => {
    expect(stringfySearchParams({ he: "hello" })).toBe("he=hello");
  });

  it("평범한 문자열 밸류를 잘 변환합니다.", () => {
    expect(stringfySearchParams({ he: ["hello", "world"] })).toBe("he=hello&he=world");
  });

  it("중첩객체도 잘 변환합니다.", () => {
    expect(stringfySearchParams({ he: { she: "hello" } })).toBe("he%5Bshe%5D=hello");
  });
});

describe("stringfy pathname test", () => {
  it("아무것도 입력받지 않으면 빈 문자열을 리턴합니다.", () => {
    expect(stringfyPathname([])).toBe("");
    expect(stringfyPathname()).toBe("");
  });
  it("입력이 있으면 path를 반환합니다.", () => {
    expect(stringfyPathname(["hello"])).toBe("hello");
  });
  it("길이가 긴 path를 입력받으면 이를 반영합니다.", () => {
    expect(stringfyPathname(["hello", "world"])).toBe("hello/world");
    expect(stringfyPathname(["hello", "world", "hi"])).toBe("hello/world/hi");
  });
});

describe("createInternalPath test", () => {
  it("평범한 입력에 대해 평범하게 대응합니다.", () => {
    expect(createInternalPath("/hello")).toBe("/hello");
  });
  it("쿼리스트링이 있는 경우 입력에 대해 쿼리스트링도 대응합니다.", () => {
    expect(createInternalPath("/hello", { query: { hello: "world" } })).toBe("/hello?hello=world");
  });
  it("쿼리스트링 여러개 있는 경우 입력에 대해 여러개의 쿼리스트링도 대응합니다.", () => {
    expect(createInternalPath("/hello", { query: { hello: "world", hi: "add" } })).toBe("/hello?hello=world&hi=add");
  });
  it("path가 있는 경우 이에 대해 대응합니다.", () => {
    expect(createInternalPath("/hello", { pathname: ["hello", "world"] })).toBe("/hello/hello/world");
  });
});

```

이제 테스트코드가 우리를 지켜주고 있으니 마음껏 코드를 건드려도 안심할 수 있습니다.

이제 해당 타입을 이용하면서 useRouter를 한번 추상화해봅시다

```tsx
"use client";
import { useParams, usePathname, useRouter, useSearchParams } from "next/navigation";
import { useMemo } from "react";
import { RoutesQueryAndPath, parseSearchParams } from "./router.util";

export const useInternalRouter = <T extends Pick<RoutesQueryAndPath, "query" | "pathname"> = RoutesQueryAndPath>() => {
  const router = useRouter();
  const pathname = usePathname();
  const serachParams = useSearchParams();
  const params = useParams<T["pathname"]>();
  const href = typeof window !== "undefined" ? window?.location?.href : "";
  const hostname = typeof window !== "undefined" ? window?.location?.hostname : "";
  const protocol = typeof window !== "undefined" ? window?.location?.protocol : "";
  const host = typeof window !== "undefined" ? window?.location?.host : "";
  const slash = typeof window !== "undefined" ? "//" : "";
  const basePath = `${protocol}${slash}${host}`;
  return useMemo(
    () => ({
      push: (href: string, option?: { scroll?: boolean }) => router.push(href, option),
      replace: (href: string, option?: { scroll?: boolean }) => router.replace(href, option),
      back: () => router.back(),
      refresh: () => router.refresh(),
      prefetch: (href: string) => router.prefetch(href),
      pathname: pathname,
      searchParams: serachParams.toString(),
      query: parseSearchParams<T["query"]>(serachParams.toString()),
      get: (qs: string) => serachParams.get(qs),
      params: params,
      href: href,
      hostname: hostname,
      protocol: protocol,
      host: host,
      basePath: basePath,
    }),
    [basePath, host, hostname, href, params, pathname, protocol, router, serachParams],
  );
};

```

next에 직접 의존하는게 아니라 인터페이스에 의존하는 코드를 만들기 위해 InternalRouter를 생성합니다.

이 과정에서 next가 뺏어간 쿼리스트링, 베이스패스, 로케이션 객체의 정보 등등을 가져와서 집어넣겠습니다.

```tsx
"use client";
import { RoutesQueryAndPath, createRoutes } from "./router.util";

type HomeRoutes = RoutesQueryAndPath<{ query: { type: "hello" | "world" } }>;
type LoginRoutes = RoutesQueryAndPath<{ query: { from: "landing" | "referrer" } }>;
type PostRoutes = RoutesQueryAndPath<{ pathname: [["slug", "react" | "typescript"]] }>;
export const Routes = {
  home: createRoutes<HomeRoutes>("/"),
  login: createRoutes<LoginRoutes>("/login"),
  post: createRoutes<PostRoutes>("/post"),
} as const;

```

이제 Routes를 정의하고 경로별로 Routes를 생성해줍니다.

이 Routes들은 내부적으로 이런 구조를 가집니다.

```tsx
export const createRoutes = <T extends RoutesQueryAndPath = RoutesQueryAndPath>(basePath: RoutesString) => ({
  path: (arg?: T["arg"]) => createInternalPath(basePath, arg),
  useRouter: () => useInternalRouter<T>(),
});

```
path는 완전히 순수한 함수이기때문에 테스트가 매우 쉬울거에요

그리고 테스트가 쉬운만큼 휴먼에러가 발생하면 안되는 곳이기도하고요!

쿼리스트링 처리, path 처리는 이전 유틸함수 테스팅에서 수행했으니 

여기서는 가장 먼저 해당 경로를 잘 생성하는지 테스트하겠습니다.

```tsx
import { Routes } from "./routes";

describe("Routes 테스트입니다.", () => {
  it("경로 매치 테스트", () => {
    expect(Routes.home.path()).toBe("/");
    expect(Routes.login.path()).toBe("/login");
    expect(Routes.post.path()).toBe("/post");
  });
});

```

테스트 코드를 작성할 때에는 기대하는 assertion값은 최대한 하드코딩하도록 합니다.

이렇게하면 변경사항이 생겼을때 코드를 수정하면서 테스트코드가 실패하게 될것이고

자연스럽게 실제 코드와 테스트코드의 싱크를 맞추는 작업도 수행하게될거에요

# 이렇게 했을때 장점은?

querystring, pathname을 완전히 선언적이게 관리할 수 있으면서 fully type하기때문에 인텔리센스의 추천도 최대한으로 활용할 수 있습니다.

```tsx
const Example = () => {
  const router = Routes.post.useRouter();
  router.push(Routes.login.path());
};
```

거기에 어디로 이동하려는지도 명확히 관리할 수 있죠

```tsx
type HelloWorldProps = { params: PostRoutes["pathname"] };

const HelloWorldpage = ({params:{slug}}:HelloWorldProps) => {}
```

next.js dynamicroutes의 타입을 가져오는것도 쉬워져요

# 마치며

라이브러리에 직접 의존하게되면 pages router -> app router 전환 등을 하려고할때 고생을 할 수 있다보니

항상 라이브러리,프레임워크에 깊게 의존하기보다는 한단계 추상화하여 인터페이스에 의존하는것을 지향하면 좋은 것 같아요


