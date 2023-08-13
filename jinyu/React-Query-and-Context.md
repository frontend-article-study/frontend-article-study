# React-Query와 Context API

리액트 쿼리의 최고의 특징 중 하나는 컴포넌트 트리에서 원하는 곳 어디서나 쿼리를 사용할 수 있습니다. 리액트 쿼리를 사용하면 아래의 코드와 같이 컴포넌트에서 필요한 데이터를 서버로부터 fecth 할 수 있습니다.

```tsx
function ProductTable() {
  const { data, isError } = useProductQuery();

  if (data) {
    return <table>...</table>;
  }

  if (isError) {
    return <ErrorMessage error={productQuery.error} />;
  }

  return <SkeletonLoader />;
}
```

위의 코드처럼 컴포넌트 내부에서 데이터를 fetch 한다는 것은 매우 유용합니다. 애플리케이션 어디로든 컴포넌트를 옮길 수 있고 동작하기 때문에 독립적으로 컴포넌트 구성이 가능합니다. 또한 이런 독립적인 컴포넌트는 변화에 매우 유연합니다. 이런 점 때문에 최근 리액트 쿼리가 각광을 받는 이유 중 하나이죠.

이렇게 좋은 리액트 쿼리라는 툴이 있지만 결국 사용하는 것은 개발자입니다. 사람은 언제나 실수합니다. 이번 주제는 Context API를 이용해 어떻게 개발자의 실수를 줄일 수 있는지에 대해 알아보고자 합니다.

다음 코드를 보겠습니다.

```tsx
export const useUserQuery = (id: number) => {
  return useQuery({
    queryKey: ['user', id],
    queryFn: () => fetchUserById(id),
  });
};
export const useCurrentUserQuery = () => {
  const id = useCurrentUserId();

  return useUserQuery(id);
};
```

이 쿼리는 로그인한 사용자가 어떤 권한을 가졌는지 확인하고 실제로 페이지를 볼 수 있는지 결정하기 위한 컴포넌트로 페이지의 모든 곳에서 필요한 필수적인 정보입니다.

만약 이 쿼리를 사용해 받아온 데이터에서 `userName`을 보여주길 원하는 컴포넌트가 있을 때 `userCurrentUserQuery` 훅으로 가져올 수 있다고 가정합시다.

```tsx
function UserNameDisplay() {
  const { data } = useCurrentUserQuery();
  return <div>User: {data.userName}</div>;
}
```

이와 같은 코드에서 `data`가 `undefined`가 될 가능성이 있으므로 타입스크립트는 이걸 허용하지 않습니다.

그럼 여기서 타입스크립트의 불평을 없앨 방법이 무엇이 있을까요?

`data!.userName`을 사용할 수 있고, `data?.userName`을 사용할 수도 있고, 그냥 타입가드로 `if(!data) return null`을 추가 할 수도 있습니다. 아니면 `useCurrentUserQuery()`를 호출하는 모든 컴포넌트에서 적절한 로딩과 에러를 추가해주는 방법도 있습니다.

만약 이 `data`가 타입스크립트가 불평하는 것과 달리 개발자가 생각했을 때 `undefined`가 될 수가 없습니다. 그럼 개발자는 “절대 일어날 수 없는” 상황에 대해 타입 가드와 같은 처리를 통해 타입스크립트의 불평을 침묵시켜야 하므로 불필요한 코드를 작성한다고 느낄 수 있으며 코드가 길어질 수 있습니다.

하지만 타입스크립트는 거의 항상 옳기 때문에 무시하기 쉽지 않습니다.

이러한 문제는 **“암시적 종속성”** 때문에 발생합니다. 암시적 종속성이란 애플리케이션 구조에 대한 개발자의 지식과 머릿속에만 존재할 뿐 코드 자체에서는 보이지 않는 종속성을 의미한다고 합니다.

데이터가 정의되었는지 확인하지 않아도 안전하게 `useCurrentUserQuery`를 호출할 수 있다는 것을 알고 있지만, 정적 분석으로는 이것을 확인할 수 없습니다. 동료들도 이 사실을 모를 수 있고 코드를 작성한 개발자도 3개월 후에는 이 사실을 모를 수 있습니다.

가장 위험한 것은 “지금은 맞을지 몰라도 미래에는 더 이상 아닐 수 있다”라는 것입니다. 캐시에 사용자 데이터가 없을 수도 있고, 이전에 다른 페이지를 방문한 경우와 같이 조건부로 있을 수도 있습니다.

이 때 우리는 Context API를 사용해 의존성을 명시적으로 만들어 해결할 수 있습니다.

앞서 `useCurrentUserQuery` 훅을 이용해 사용자의 데이터를 가져왔습니다. 그리고 이 데이터를 통해 사용자 인증을 위한 데이터의 사용 가능성을 확인하는 검사를 수행해야 합니다. 근데 만약 이 훅을 사용하는 컴포넌트가 20개 정도 있다고 한다면, 이 모든 20개 컴포넌트에서 사용 가능성을 확인해야 합니다.

이러한 검사를 Context API를 사용해 아래의 코드와 같이 context에서 사용 가능성을 확인하는 코드를 작성한다면 20개의 모든 컴포넌트에서 확인할 필요 없이 한 번만 작성하면 됩니다.

```tsx
const CurrentUserContext = React.CreateContext<User | null>(null);

export const useCurrentUserContext = () => {
  return React.useContext(CurrentUserContext);
};

export const CurrentUserContextProvider = ({
  children,
}: {
  children: React.ReactNode;
}) => {
  const currentUserQuery = useCurrentUserQuery();

  if (currentUserQuery.isLoading) {
    return <SkeletonLoader />;
  }

  if (currentUserQuery.isError) {
    return <ErrorMessage error={currentUserQuery.error} />;
  }

  return (
    <CurrentUserContext.Provider value={currentUserQuery.data}>
      {children}
    </CurrentUserContext.Provider>
  );
};
```

이렇게 Context를 만듦으로서 `UserNameDisplay` 컴포넌트 같은 자식 컴포넌트에서 동료 혹은 다른 누군가가 `UserNameDisplay`를 볼 때마다 `CurrentUserContextProvider`에서 제공된 데이터가 필요하다는 것을 명시적으로 알려줄 수 있습니다.

```tsx
function UserNameDisplay() {
  const data = useCurrentUserContext();
  return <div>User: {data.username}</div>;
}
```

이렇게 명시적으로 알려줌으로서 암시적 종속성의 문제를 해결할 수 있습니다. Provider가 렌더링 되는 위치를 변경하면 해당 context를 사용하는 모든 자식에도 영향을 미치며 어떤 리액트 쿼리의 훅이 어떤 컴포넌트에서 호출되는지 범위도 알 수 있으므로 리팩터링할 때 유용하게 사용할 수 있습니다.

### 인사이트

솔직히 이번 주의 경우 이미 선약된 약속들과 가족 휴가로 인해 시간을 많이 투자하지 못했습니다. 이번 주 스터디 주제는 Context API에 대해 조금 더 깊은 내용을 찾아보다가 발견한 아래의 레퍼런스 사이트의 번역글을 읽고 제가 이해한 대로 다시 정리하였습니다.

저한테는 번역글이 뭔가 매끄럽지 않게 느껴져 한 번에 이해가 되지 않았어요. 지금 글을 쓰는 시점에서도 아리송하네요.

글의 전반적인 내용은 “암시적 종속성”의 문제에 대해 Context API를 이용해 보완하는 방법에 대해 설명하는 것 같았어요. 그리고 Context API는 상태 관리를 위한 도구로 사용하기 보다는 “의존성 주입” 도구로 활용하라는 것 같았어요.

즉, “의존성 주입”을 통해 컴포넌트를 유연하고 확장성 있게 설계하고 명시적으로 만듦으로서 암시적 종속성을 보완하고 팀의 협업 기여할 수도 있고, 이후 리팩토링에서도 매우 유용하다라는 것을 알려주는 글인 것 같았습니다.

레퍼런스 글을 읽으면서 나왔던 “상태 동기화”, “요청 폭포” 라는 용어들이 저한테는 생소하네요. 이런 부분에 대한 추가 학습과 Context API와 의존성 주입의 관계에 대해서 더 공부를 해봐야 될 것 같아요.

유명한 블로그 같아 이미 알고 계시는 분들도 있겠지만 레퍼런스의 다른 글들을 보니 React-Query에 대해 이것저것 많은 팁을 전수해주고 있더군요. 관련 한글 번역글도 많고요. 꼭 정독을 해봐야겠습니다.

### 레퍼런스

https://tkdodo.eu/blog/react-query-and-react-context
