# Promise.allSettled

Promise.allSettled는 다음과 같은 메서드입니다.

간단하게 타입으로 표현하면 다음과 같을 수 있습니다.

```tsx
type PromiseAllSettledArgument = Promise<any>[];

type PromiseAllSettled = (
  param: PromiseAllSettledArgument,
) => (PromiseRejectedResult | PromiseFulfilledResult<any>)[];
```

async await 문법은 개발자가 간편하고 직관적이게 비동기 동작을 이해할 수 있고

비동기 작업의 순차 처리를 수행하는데에 큰 도움을 주는 문법적 설탕입니다.

하지만 프론트엔드 개발을 수행하다보면 순차처리가 필요하지 않은 상황이 여럿 존재합니다.

예컨대 병렬처리가 더욱 효율적인 경우가 있을 수 있습니다.

# 병렬처리가 효율적인 이유

수행하는 데에 각각 1초의 시간이 걸리는 비동기 작업 5개가 있다고 가정해보겠습니다.

이 비동기 작업을 async await을 통해 표현하면 다음과 같습니다.

```tsx
const awaitPromise = async () => {
  await doAsyncSomeThing(); // 1초
  await doAsyncSomeThing(); // 2초
  await doAsyncSomeThing(); // 3초
  await doAsyncSomeThing(); // 4초
  await doAsyncSomeThing(); // 5초
  return;
};
```

이러한 작업은 서로간의 순서가 중요하지 않은 작업에서는 매우 비효율적입니다.

그러나 여러개의 비동기 작업을 수행할때에 중요한 것은 어떤 비동기 작업이

성공하고 실패했는지를 트래킹하는 것입니다.

Promise.allSettled는 이를 아주 간편하게 지원합니다.

위 코드를 Promise.allSettled로 치환하면 다음과 같습니다.

# 병렬 처리를 수행하기

```tsx
const allPromise = async () => {
  const promiseList = [
    doAsyncSomething(),
    doAsyncSomething(),
    doAsyncSomething(),
    doAsyncSomething(),
    doAsyncSomething(),
  ]; 

  return Promise.allSettled(promiseList);
};
```

이 코드는 병렬적으로 비동기 작업이 수행되기 떄문에

수행 완료까지 1초의 시간밖에 걸리지않습니다.

또한 allSettled의 반환값은 다음과 같은 형태인데요

```tsx
type returnType = {
    status:'fulfilled'|'rejected'
    value?:any
    reason?:any
}[]
```

따라서 어떤 비동기 작업이 실패/ 성공했는지를 status 프로퍼티를 통해 트래킹할 수 있습니다.

그런데 이렇게 유용한 Promise.allSettled도 문제가 하나 존재하는데요..

바로 타입스크립트와 사용하기 힘들다는 것입니다.

타입추론이 적절하게 수행되지 못하거든요

이를 해결하기 위해서 두가지 방법을 고려해볼 수 있는데요

# 타입스크립트에서 Promise.allSettled 사용하기

1. 유형 어설션

2. 사용자 커스텀 타입가드

두가지 방법이 존재합니다.

개인적으로는 커스텀 타입가드를 만드는게 더욱 간결하게 느껴져서 추천드리고 싶은데요

우선 유형 어설션의 사례부터 보겠습니다.

유형 어설션은 개발자가 타입스크립트에게 이 값은 이거다 라고 알려주는 행위에 가깝습니다.

예시를 보겠습니다.

```tsx
const isFulfilledPromise = async () => Promise.resolve('hi' as const);

Promise.allSettled([isFulfilledPromise]).then((d) => {
  const fulfilledList = d.filter(
    (item) => item.status === 'fulfilled',
  ) as PromiseFulfilledResult<any>[];

  const getFulfilledValue = fulfilledList.map(
    (item) => item.value,
  ) as PromiseFulfilledResult<'hi'>[];
});

```

제가 간단하게 작성한 예시입니다.

유형 어설션을 통해 개발자가 직접 반환타입을 알려줍니다.

이렇게 작성한 경우 이제 타입스크립트는 getFulfilledValue가

'hi' 타입의 요소만을 가진 배열로 인식하게됩니다.

```tsx
const isRejected = (
  input: PromiseSettledResult<unknown>,
): input is PromiseRejectedResult => input.status === 'rejected';

const isFulfilled = <T>(
  input: PromiseSettledResult<T>,
): input is PromiseFulfilledResult<T> => input.status === 'fulfilled';

const myPromise = async () => Promise.resolve('hello world');

const someThingDifficult = async () =>
  Promise.resolve({ hard: 123, soft: 'dsad' });

const typeSafe = async () => {
  const data = await Promise.allSettled([someThingDifficult()]);

  const response = data.filter(isFulfilled);
  const error = data.filter(isRejected);
  return {
    response,
    error,
  };
};

typeSafe().then(console.log);

typeSafe().then((data) => data.response.map((item) => item.value.hard));

```

다음은 사용자 지정 타입가드를 이용한 해결방법입니다.

이 방식의 단점은 같은 반환값을 가지는 프로미스끼리만 사용하는게 가능하다는 것인데요

제네릭을 늘려주면 해결이 될수도 있겠지만.. 애매하네요
