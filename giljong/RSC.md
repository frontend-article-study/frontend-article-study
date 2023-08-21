
# React Server Component

해당 자료에 더한 더욱 자세한 내용은 아래 블로그에서 확인할 수 있습니다.

본 문서는 간략하게 요약한 내용입니다.

https://xionwcfm.tistory.com/393



# What is React Server Component?

React Server Component는 react 18 버전부터 도입된 개념으로서

서버에서만 렌더링 되어 전달되는 컴포넌트를 의미합니다.

리액트 서버 컴포넌트는 HTML이 아닌 "특별한 형태로" 서버에서 렌더링 되어 클라이언트에 전달됩니다.

유의해야할 것은 **RSC !== SSR** 이라는 것입니다.


<br/>

서버 컴포넌트가 있으면 클라이언트 컴포넌트도 있지 않을까?

생각할 수 있습니다.

클라이언트 컴포넌트는 기존 리액트에서 사용하던 컴포넌트를 의미하게됩니다.



# RSC는 왜 필요한가요?

RSC가 해결하는 기존의 문제점은 크게 두가지로 정리할 수 있습니다.

1. 기존 리액트의 데이터페칭 워터폴 문제

2. 라이브러리 번들 사이즈 문제



# 데이터 페칭 워터폴 문제

```tsx
function KakaopayHome({ memberId }) {
  return (
    <MemberDetails memberId={memberId}>
      <MoneyBalance memberId={memberId}></MoneyBalance>
      <PaymentHistory memberId={memberId}></PaymentHistory>
    </MemberDetails>
  );
}
```

위 코드는 memberid를 prop으로 전달받고 자식 컴포넌트들에게

아이디를 뿌려주는 KakaopayHome 컴포넌트에 대한 코드입니다.

각 컴포넌트는 멤버아이디에 기반한 데이터를 그려주는 역할을 수행해야합니다.

이러한 상황에서 개발자는 두가지 선택지를 고려할 수 있습니다.
<br/>

1. 부모 컴포넌트에서 하나의 거대한 API를 호출하고 데이터를 자식에게 전달하기

2. 각각의 컴포넌트에서 필요한 API를 호출하고 데이터를 사용하기

<br/>

그러나 첫번째 방법은 네트워크 요청에 워터폴이 일어날 수 있습니다.

부모 컴포넌트의 데이터페칭 -> 렌더링이 완료되기 전까지는

자식 컴포넌트의 데이터페칭이 이루어질 수 없기 때문입니다.

<br/>

두번째 방법은 네트워크 요청에서는 효율적이어지지만

컴포넌트를 재사용하기 어려워지고 구성이 바뀌게 되면 `over-fetching`이 생기기 쉽습니다.

<br/>

기존에는 `relay` , `graphql`을 통하여 `over-fetching` 문제를 해결해왔지만

이러한 솔루션은 모든 리액트 프로젝트에 범용적으로 적용될 수 있지는 않았습니다.

따라서 새로운 해결책이 필요했습니다.


# 라이브러리 번들 사이즈 문제

```tsx
// *Before* Server Components
import marked from "marked"; // 35.9K (11.2K gzipped)
import sanitizeHtml from "sanitize-html"; // 206K (63.3K gzipped)

function NoteWithMarkdown({text}) {
  const html = sanitizeHtml(marked(text));
  return (/* render */);
}
```

최근에는 트리쉐이킹이 대부분 지원되지만 그럼에도 불구하고

라이브러리 사용으로 인한 번들사이즈 증가는 피할 수 없는 문제입니다.

리액트 등장 이후 개발자들은 자바스크립트 번들의 크기를 줄이기 위해 노력해왔지만

이를 근본적으로 해결하지는 못하였습니다.





# RSC는 어떻게 해결할까?


데이터 페칭과 렌더링을 서버에서 해결하기 때문에

불필요한 렌더링을 줄일 수 있어집니다.

(네트워크 워터폴 문제는... 해결중이라고하네요 ㅎ;ㅎ;)

<br/>


리액트 서버 컴포넌트는 서버측에서 컴포넌트를 렌더링하며

새로운 데이터가 있을 때에도 서버측에서 렌더링을 하며 일부 컴포넌트만 다시 받아올 수 있는 점을 활용하여

클라이언트측에 전송해야하는 코드량을 제한할 수 있도록 도와줍니다.


<br/>


# (adv) RSC의 동작 원리

리액트 서버 컴포넌트는 앞서 특별한 형태로 클라이언트에게 렌더링 정보를 전달한다고 했습니다.

그리고 RSC는 SSR이 아니라고도 하였습니다.

왜 그런 것일 까요?

이는 RSC의 동작을 살펴보면 알 수 있습니다.


RSC가 반환하는 것은 HTML이 아닙니다.
```
M1:{"id":"./src/ClientComponent.client.js","chunks":["client1"],"name":""}
J0:["$","@1",null,{"children":["$","span",null,{"children":"Hello from server land"}]}]
```

RSC는 위와 같은 JSON 데이터를 스트림의 형태로 반환합니다.

M으로 시작하는 라인은 클라이언트 번들에서 컴포넌트 함수를 조회하는 데 필요한 정보와

클라이언트 컴포넌트 module reference를 정의합니다.

J로 시작하는 라인은 M 라인에서 정의된 클라이언트 컴포넌트를 참조하는 것으로

실제 리액트 컴포넌트 트리를 정의합니다.

그리고 @로 작성된 부분들은 아직 데이터 페칭이 완료되지 않았기에

임시로 보여지는 placeholder 입니다.

<br/>

이러한 형식으로 작성된 포맷은 스트림의 형태로 전송이 가능합니다.

즉 한행이 완성되면 그 행을 즉각적으로 작업에 반영할 수 있습니다.

이러한 형태로 스트림된 렌더링 데이터들은

`react-server-dom-webpack`과 같은 플러그인들에 의해

리액트 엘리먼트 트리로 재구성 됩니다.

```tsx

import { createFromFetch } from 'react-server-dom-webpack'
function ClientRootComponent() {
  // fetch() from our RSC API endpoint.  react-server-dom-webpack
  // can then take the fetch result and reconstruct the React
  // element tree
  const response = createFromFetch(fetch('/rsc?...'))
  return (
    <Suspense fallback={null}>
      {response.readRoot() /* Returns a React element! */}
    </Suspense>
  )
}

```

response.readRoot()를 통해 응답 스트림이 처리 될 때 업데이트 되는 리액트 엘리먼트를 반환합니다.

준비되지 않은 @ 참조들을 발견할 때마다 promise를 throw 하는 것을 통해

suspense는 fallback을 표시하며 모든 promise가 resolve 될 때 까지 반복합니다.