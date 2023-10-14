# getServerSideProps

앱라우터가 더 익숙하신 분들도 많겠지만 app router의 경우

아직은 프로덕션에서 사용하기 꺼려지는 것이 사실입니다.

따라서 앱라우터에서 해결된 문제들을 앱라우터에서는 쉽게 사용할 수 있지만

페이지 라우터에서는 조금 어려움이 있곤 합니다.

<br/>

페이지 라우터에 익숙하지 않은 분들을 위해

getServerSideProps에 대해 간략히 설명하면

이것은 서버사이드에서 데이터를 페칭하도록 도와주는 함수라고 생각할 수 있습니다.

서버사이드에서 비동기 작업을 수행하면 좋은 점은

네트워크 워터폴을 최소화할 수 있으며 서버에서만 가능한 로직들을 수행할수도 있다는 것입니다.

그뿐만이아니라 불필요한 로딩 , 에러처리에 대한 로직을 컴포넌트에서 작성하지 않아도된다는 점과

기존 useEffect를 이용한 데이터페칭의 약점인

컴포넌트 렌더링 -> 비동기 데이터 요청 -> 비동기 요청 완료 -> 컴포넌트 리렌더링

과 같은 리렌더링 이슈도 해소할 수 있다는 장점이 있습니다.

그런데 이러한 getServerSideProps에서 아쉬운 점이 하나 있는데요

# 흰 화면을 보여주어야 한다.

getServerSideProps의 문제점은 getServerSideProps의 동작원리와 큰 연관이 있습니다.

getServerSideProps는 어떻게 동작할까요?

# getServerSideProps의 동작

getServerSideProps로 작성된 페이지에 이동할 때에 일어나는 일련의 과정은 다음과 같습니다.

/ 에서 getServerSideProps를 이용하는 /eunhe 경로로 이동한다고 가정해봅시다.

1. 이용자는 Next/link로 작성된 링크 컴포넌트를 클릭하여 /eunhe 경로로 이동을 요청합니다.

2. eunhe 경로는 getServerSideProps를 이용하기 때문에

서버리스 함수 getServerSideProps 를 이용하여 서버에

eunhue 경로에 대한 getServerSideProps 함수를 실행할 것을 요청합니다.

3. 서버는 getServerSideProps를 실행하고 비동기 작업이 완수되면 결과를 클라이언트에게 전달합니다.

4. 클라이언트는 3번 과정이 완료되기까지 그저 기다립니다.

여기에서 기다린다는 것은 서버로부터의 응답을 기다리는 행위를 한다는 것입니다.

<br/>

이제 눈치채셨나요?

getServerSideProps를 이용하는 동안 우리는 로딩 상태를 보여줄 수 없습니다.

우리의 데이터 페칭은 서버에서 이루어지기때문입니다.

서버에서 비동기 요청에 대한 로딩을 모두 수행하고 결과를 전달하는 동안

클라이언트에서는 아무것도 할 수 없습니다.

왜냐하면 우린 클라이언트에서 "지금은 데이터를 가져오는 중이다"라는 상태를 만들어놓을 수 없었기 떄문입니다.

이것이 getServerSideProps 를 이용하면 로딩 컴포넌트를 보여줄 수 없는 이유입니다.

# 그러면 어떻게 해결할 수 있을까요?

사실 위에서도 살짝 힌트를 드렸는데요

next.js에게 getServerSideProps와 같은 함수로 인하여

페이지 이동에 로딩이 필요한 경우 이러한 컴포넌트를 보여주어라.

라는 코드를 넣어주는 것을 통해 문제를 해결할 수 있습니다.

그리고 이 일은 next가 제공하는 next/router의 Router 를 통해서 수행할 수 있어요

nextjs는 Router 내부에서 발생하는 특정 이벤트들을 핸들링할 수 있도록

이벤트 처리를 커스텀하는 기능을 제공해주는데요

기본적으로 모든 이벤트들은 url과 {shallow}를 파라미터로 받는답니다.

만약 /A => /B로 이동한다면 이벤트는 다음과 같이 발생해요

1. routeChangeStart

2. beforeHistoryChange

3. routeChangeComplete

그러면 이제 할일은 간단합니다.

로딩 상태를 만들어주고

routeChangeStart 이벤트가 발생하면 로딩 상태를 true로

routeChangeComplete 이벤트가 발생하면

라우트 이동이 끝났다는 것이니 로딩 상태를 false로 바꿔주는 코드를 작성하고

그 로딩 상태에 따라 보여줄 로딩 컴포넌트를 제작해주면 되는것입니다!

먼저 이벤트를 감지하고 그 이벤트에 대해 로딩 상태를 조작하는 로직을 작성해주겠습니다.

훅으로 작성하면 좋겠네요

```tsx
import { Router } from 'next/router';
import React from 'react';

const useSSRLoading = () => {
  const [loading, setLoading] = React.useState(false);
  React.useEffect(() => {
    const start = () => {
      console.log('start');
      setLoading(true);
    };
    const end = () => {
      console.log('findished');
      setLoading(false);
    };
    Router.events.on('routeChangeStart', start);
    Router.events.on('routeChangeComplete', end);
    Router.events.on('routeChangeError', end);
    return () => {
      Router.events.off('routeChangeStart', start);
      Router.events.off('routeChangeComplete', end);
      Router.events.off('routeChangeError', end);
    };
  }, []);
  return loading;
};

export default useSSRLoading;
```

이제 이 useSSRLoading 훅을 \_app.tsx에서 호출해줍니다.

```tsx
const loading = useSSRLoading();
```

그리고 이 로딩 상태에 대해서 보여줄 로딩 컴포넌트를

삼항연산자를 통해서 보여주면 끝이죠!

# 마치며

앱 라우터의 loading.tsx 파일이 정말 그리운 순간이었습니다.

페이지에서도 쓰게해줘잉~~~~~~~
