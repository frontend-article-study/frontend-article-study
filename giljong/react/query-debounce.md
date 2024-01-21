# 리액트 쿼리는 뮤테이션 디바운싱을 안줘요

진짜 어이없다 그죠

근데 해결할 방법이 있습니다.

라이브러리가 제공해주지 않는다면 직접 구현하면 되는 것이죠

# 왜 쿼리를 디바운싱해야하는데?

개발을 하다보면 굉장히 흔하게 생기는 니즈인데요

네트워크 요청이라는 행위 자체가 개발자 입장에서 가벼운 행위가 아닙니다.

또한 네트워크 요청 === 서버비용 인 경우도 있기 때문에

프론트엔드 입장에서 불필요한 api 호출은 줄이면 줄일수록 좋습니다.

꼭 필요할 때에만 요청을 보내는 것이지요

예컨대 사용자가 엄청나게 많이 상호작용을 시도한다면

그 시도에대해 모두 요청을 날리는 것이 아니라

사용자의 상호작용이 멈추기까지 기다렸다가 마지막 요청을 보내는 것이

효율적인 경우가 매우 많습니다.

# redux saga에서는 기본 제공해줍니다.

```tsx
import { takeLatest } from `redux-saga/effects`

function* fetchUser(action) {
  ...
}

function* watchLastFetchUser() {
  yield takeLatest('USER_REQUESTED', fetchUser)
}

const takeLatest = (patternOrChannel, saga, ...args) => fork(function*() {
  let lastTask
  while (true) {
    const action = yield take(patternOrChannel)
    if (lastTask) {
      yield cancel(lastTask) // cancel is no-op if the task has already terminated
    }
    lastTask = yield fork(saga, ...args.concat(action))
  }
})
```

takeLatest라는 함수를 제공하는 것을 통해 리덕스 사가는

사용자의 여러 연속 작업에 대하여 최신 작업으로만 결론을 내리는 기능을 제공합니다.

다만 리액트 쿼리는 이런 디바운싱에 관련된 기능을 제공해주지 않는데요

쿼리 작업의 경우에는 쿼리키가 변경되면 쿼리를 다시 트리거하는 성질을 이용하여

쿼리키에 들어가는 상태의 변경을 유예하는 것을 통해 구현할 수 있습니다.

# 뮤테이션은 어떻게 해야할까요?

여러가지 방법이 있을 수 있겠지만 저같은 경우에는 디바운스 훅을 구현하여

뮤테이션 요청 자체를 디바운싱 하는 기법을 사용합니다.

다만 아직 추상화할 방법이 애매해서 고민을 좀 하고있는 상태입니다.

```tsx
import React from 'react';

export const useDebounce = <T extends unknown[]>(
  callback: (...params: T) => void,
  time: number,
) => {
  const timer = React.useRef<ReturnType<typeof setTimeout> | null>(null);
  return (...params: T) => {
    if (timer.current) clearTimeout(timer.current);

    timer.current = setTimeout(() => {
      callback(...params);
      timer.current = null;
    }, time);
  };
};
```

ref와 타이머함수를 이용하여 간단한 디바운싱 커스텀훅을 구현합니다.

이후 이 커스텀훅에 뮤테이션 요청을 감싸고

디바운싱 된 뮤테이션을 이용하는것이 핵심 아이디어입니다.

```tsx
const useDebounceMutation = () => {
  const queryClient = useQueryClient();
  const debouceMutation = useMutation<MockApiType, Error, MutationArg>({
    mutationFn: async ({ bodyData }: MutationArg) => {
      const response = await fetch(`http://localhost:3000/api/debounce`, {
        method: 'POST',
        body: JSON.stringify({ test: bodyData }),
        headers: {
          'Content-Type': `application/json`,
        },
      });
      const data = await response.json();
      return data;
    },
    onSettled: () => {
      queryClient.invalidateQueries([DEBOUNCE_QUERY_KEY_FACTORY.ALL]);
    },
  });
  return debouceMutation;
};

const useDebouncingHandler = (debounceTime = 1000) => {
  const debounceMutation = useDebounceMutation();
  const requestDebouncing = useDebounce(({ bodyData }: MutationArg) => {
    debounceMutation.mutate({ bodyData });
  }, debounceTime);
  return requestDebouncing;
};

const useGetDebounceQuery = () => {
  const debounceQuery = useSuspenseQuery<MockApiType>(
    [DEBOUNCE_QUERY_KEY_FACTORY.ALL],
    async () => {
      const response = await fetch(`http://localhost:3000/api/debounce`);
      const data = response.json();
      return data;
    },
  );
  return debounceQuery;
};
```

이렇게 디바운싱을 적절히 먹여주면 되겠습니다.

만약 뮤테이션 이후 특정 쿼리키가 refetch 되어야한다면

mutation의 구현부에서 onSettled와 같은 옵션에 이와 관련된 처리를 해주는 것이 좋습니다.

```tsx
const [another, setAnother] = React.useState(['another working']);
const debounceState = useGetDebounceQuery();
const debounceHandler = useDebouncingHandler(1000);
const onClickHandler = () => {
  const date = new Date().getTime();
  const anotherContent = `it work for another${date}`;
  const debounceContent = `it work for debounce${date}`;
  setAnother([...another, anotherContent]);
  debounceHandler({ bodyData: debounceContent });
};
```

실제 사용은 다음과 같이 할 수 있습니다.

# 마치며

구현 난이도가 어렵다기보다는 코드를 깔끔하게 관리할 방법이 고민되는 것 같습니다.

그냥 라이브러리 차원에서 제공해주면 참 편리할텐데요

왜 그런지 모르곘읍니다 하하

그와 별개로 쿼리키를 적극적으로 활용하는 형태일 때에

쿼리키 팩토리 패턴을 이용하는 것이 아주 유용하다는 생각이 듭니다.
