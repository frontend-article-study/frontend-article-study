## **😙 HighOrderComponent란?**

고차 컴포넌트란 컴포넌트 로직을 재사용하기 위한 기술입니다.

리액트 문서의 정의에 따르면 고차 컴포넌트는

**"컴포넌트를 가져와 새 컴포넌트를 반환하는 함수"** 라고 해요

```tsx
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```

예제를 보면 다음과 같은데 구조만 보면 꽤나 익숙합니다.

React에서 제공하는 forwardRef , memo와 같은 함수들의 사용형태와 같은데

redux의 connect 함수도 생각이 나네요

---

## **😇 횡단 관심사(cross cutting concerns)**

이러한 고차컴포넌트 패턴은 리액트에서 횡단관심사를 통하여 로직을 바라볼 수 있게 도와줍니다.

이전에는 mixin을 사용하여 구현하였으나 mixin이 가지는 문제가 있어

고차컴포넌트 패턴을 권장하는 형태로 바뀌었다고 해요

서로 다른 일을 수행하는 객체,로직들 사이에서 공통적으로 적용되는 관심사들을

횡단하는 것을 통하여 코드를 깨끗하게 분리해낼 수 있으며 + 재사용할 수 있어집니다.

따라서 고차 컴포넌트 패턴을 적용하면 좋을 상황도 쉽게 추론할 수 있습니다.

많은 컴포넌트에서 공통적으로 사용되는 로직을 분리하고 싶은 경우 유용하겠죠?

---

## **😌 나의 부끄러운 착각**

```tsx
<Suspense fallback={<div>loading</div>}>
  <Fetcher/>
<Suspense/>
```

저는 이 코드에서 서스펜스와 같이 자식 컴포넌트를 감싸주는 부모 컴포넌트를

고차컴포넌트라고 부르는 줄 알았는데 아니더라구욤..

위와 같은 형태의 컴포넌트들은 컨테이너 / 래퍼 컴포넌트라고 부르는 듯 합니다.

---

## **🥰 직접 구현해보기**

프론트엔드에서 이러한 고차컴포넌트 패턴을 잘 활용할 수 있는 대표적인 예시는

인증 / 인가 여부에 따라 접근할 수 있는 컴포넌트, 화면을 다르게 설정하는 것을 떠올릴 수 있습니다.

이러한 예시를 직접 간단하게 구현해봅시다.

실습환경은 next.js pages router/ jotai이지만 두 라이브러리, 프레임워크에 대한 지식이 없어도

상관없을만큼 간단한 예시를 준비했으니 이런식으로 하면 되겠구나 감만 잡으시면 되겠습니다.

```tsx
import { atom } from 'jotai';

type CertificationType = {
  isLogin: boolean;
  accessToken: string | null;
};

export const certificationAtom = atom<CertificationType>({
  isLogin: false,
  accessToken: null,
});
```

먼저 인증을 구현하기 위해 로그인을 했는지 구분하는 전역 상태를 만들어주겠습니다.

여러가지 방법으로 구현할 수 있지만 간단하게 전역상태를 쓰기 위해 jotai를 써주겠습니다.

jotai는 provider를 감싸주지 않아도 사용할 수 있습니다.

```tsx
import { certificationAtom } from '@/src/store/certification';
import { useAtom } from 'jotai';
import React from 'react';

interface LoginPageProps {}

const LoginPage = ({}: LoginPageProps) => {
  const [state, setState] = useAtom(certificationAtom);
  return (
    <div>
      <button
        onClick={() => {
          setState((s) => ({
            isLogin: !s.isLogin,
            accessToken: s.accessToken ? null : 'dsalkfmdkfdkl',
          }));
        }}
      >
        로그인하기
      </button>
    </div>
  );
};

export default LoginPage;
```

useAtom안에 만들어둔 아톰을 넣어두면 간단하게 상태를 useState처럼 사용할 수 있습니다.

로그인 페이지 컴포넌트에는 로그인 상태를 반전시키는 간단한 버튼을 만들어주겠습니다.

로그인이 필요한 경우에는 로그인 페이지 컴포넌트가 렌더링되며

버튼을 누르는것을 통해 로그인을 할 수 있게 하는 코드입니다.

```tsx
import LoginPage from '@/src/routes/login/login.page';
import { certificationAtom } from '@/src/store/certification';
import { useAtom } from 'jotai';
import React from 'react';

export const withCertified = <P extends object>(
  Component: React.FunctionComponent<P>,
) => {
  const WrappingComponent = (prop: P) => {
    const [certification, setCertification] = useAtom(certificationAtom);
    if (certification.isLogin) {
      return <Component {...prop} />;
    }
    return <LoginPage />;
  };
  const displayName = Component.displayName || Component.name || 'Component';
  WrappingComponent.displayName = `Certified(${displayName})`;
  return WrappingComponent;
};
```

이제 본격적으로 고차컴포넌트를 구현해봅시다.

고차컴포넌트 패턴은 그 특성상 상용구가 조금 필요하다는 점을 기억해야하는데요

개인적으로는 어쩔 수 없는것이 아닌가 싶습니다.

컴포넌트를 반환하는 함수를 만들어야하는 것이니까요

고차컴포넌트는 훅과 비슷하게 네이밍 규칙이 존재합니다.

훅과 달리 반드시 지켜야만 하는 것은 아니지만 개발자들 사이에서 통상적으로 쓰이는 네이밍 규칙이니

기억해두시면 좋을 듯 합니다. 커스텀 고차컴포넌트의 네이밍은 with로 시작해야합니다.

고차컴포넌트는 컴포넌트를 인수로 받고 필요한 동작을 수행한 뒤 인수로 받은 컴포넌트를 반환하면 됩니다.

다만 필요한 부분이 있는데 바로 대부분의 컴포넌트는 props를 받는다는 것입니다.

그렇다보니 인수로 받은 컴포넌트가 받아야할 props를 적절히 추론시킬 수 있어야할 것입니다.

그것을 하기위해 제네릭을 통해 인수로 받은 컴포넌트가 가져야하는 props를 추론합니다.

displayName의 경우에는 필수적인 부분은 아니지만 해주면 좋으니까

(eslint의 불평을 잠재울수있고 / 리액트 디버거의 도움을 받기 쉬워집니다.)

```
export default withCertified(HighOrderConsumer);
```

이렇게 제작한 고차컴포넌트를 컴포넌트에 래핑해주는 작업을 통해 사용합니다.

---

## **🙃 highordercomponent vs wrapper**

사실 위에서 고차컴포넌트로 구현한 작업은 wrapper를 만드는 것으로도 비슷하게 수행할 수 있습니다.

```tsx
import LoginPage from '@/src/routes/login/login.page';
import { certificationAtom } from '@/src/store/certification';
import { useAtomValue } from 'jotai';

export const CertificationWrapper = ({
  children,
  fallback,
}: {
  children?: React.ReactNode;
  fallback?: React.ReactNode;
}) => {
  const certificate = useAtomValue(certificationAtom);
  if (certificate.isLogin) {
    return children;
  }
  return fallback ?? <LoginPage />;
};
```

이와 같이 적절한 값일때에만 children을 렌더하고 그렇지 않은 경우에는 login을 렌더하는 전략을 통해서 말이에요

그렇다면 어떤 방법을 사용하면 좋을까요?

두 방식을 모두 사용해본 결과 저는 이런 결론을 내릴 수 있었습니다.

#### **HighorderComponent**

| 장점 | 1\. 컴포넌트를 사용하는 입장에서 고차컴포넌트의 존재를 전혀 알지 못한다.                                                    |
| ---- | --------------------------------------------------------------------------------------------------------------------------- |
| 단점 | 1\. 상황에 따라 다른 일을 수행해야하는 경우 외부에서 값을 inject해주기가 힘들다. 2\. 이해해야하는 상용구와 규칙이 존재한다. |

특히 highordercomponent를 설계할 때에는 안티패턴을 유의하며 코드를 작성해야합니다.

예컨대 highordercomponent에서 props를 추가 / 제거 / 수정하는 경우는 좋지않습니다.

컴포넌트를 고치거나 고차컴포넌트를 사용할 때 항상 그 prop을 기억하고 있어야한다는

병목지점이 추가되기 때문입니다.

#### **WrapperComponent**

| 장점 | 1\. 상황에 따라 다른 값을 inject 해주는 것이 편리하다. (wrapper역시 컴포넌트이기 때문)                                                                          |
| ---- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 단점 | 1\. 컴포넌트를 사용하는 입장에서 WrapperComponent의 존재를 알아야만한다. 2\. 설계에 따라 컴포넌트를 사용하는 사람에게 너무 많은 통제권을 쥐어주게 될 수도 있다. |

눈치가 빠르신 분들이라면 이미 알아채셨겠지만 두 방식은 모두 장점과 단점이 하나를 가리킵니다.

또 한 방식의 단점이 다른 방식의 장점이되기도 합니다.

즉 고차컴포넌트 방식과 , 래퍼방식은 서로 수행하는 일은 비슷하지만 그것을 수행하는 방법이 다릅니다.

따라서 적절한 상황도 다를것이라고 유추할 수 있습니다.

저는 간단하게 이런 방식을 통해서 뭘 사용할지를 구분 짓곤해요

**1\. 고차컴포넌트를 쓰는 경우**

\-> 상황을 타지않는 공통된 하나의 동작을 처리할 필요성이 있을 때

\-> 해당 동작에 대한 맥락을 다른 컴포넌트들이 전혀 알지 못 하게 만들고 싶을 때

**2\. 래퍼를 쓰는 경우**

\-> 공통된 동작이 존재하면서도 특정 상태에 따라 다르게 동작해야하는 경우

\-> 해당 동작에 대한 맥락을 사용하는 곳에서 명확하게 드러내고싶을 때

\-> 공통되게 하나의 동작을 하는게 아니라 해당 동작을 수행해야할 때 / 수행하면 안될 때가 나눠져있는 경우

예를 들면 예시로 사용했던 인증 로직의 경우

특정 페이지는 무조건 인증이 필요하다는 것을 알고 있고 / 모든 경우에서 인증을 무조건 수행하게하고싶다면

고차컴포넌트 패턴을 통해 공통된 인증 동작을 정의할 수 있습니다.

반면 코드를 읽는 사람 입장에서 이 페이지가 인증이 필요한지를 deepdive없이도 알 수 있게하고싶다면

컴포넌트를 보는 입장에서 이 페이지는 인증이 필요하다는 것을 한눈에 알 수 있게

래퍼 컴포넌트를 통해서 맥락을 알려줄 수도 있을 것입니다.

이렇듯 큰 궤는 같아보이는 요구사항이더라도 내 프로젝트의 환경과 요구사항에 따라

정답은 언제든지 바뀔 수 있습니다.

프로그래밍에는 유명한 말이 있죠?

"No Silver Bullet – Essence and Accidents of Software Engineering"

---

## **😎** **마치며**

고차컴포넌트에 대해 알아보는 것을 넘어 다른 방식과의 차이점도 다루어보았습니다.

망치를 들면 모든게 못으로 보인다는 말이 있습니다.

망치를 손에 쥐고 있으면 모든 문제가 그 망치로 해결할 수 있는 "못"처럼 보인다는 말인데요

고차컴포넌트 패턴은 적절한 상황에서 큰 도움을 주는 좋은 "망치"일 수 있지만

모든 문제가 망치로 해결할 수 있는 "못"은 아니라는 것을 항상 기억해야할 것입니다.

읽어주셔서 감사합니다!

---

## **😎 레퍼런스**

[https://ko.legacy.reactjs.org/docs/higher-order-components.html](https://ko.legacy.reactjs.org/docs/higher-order-components.html)
