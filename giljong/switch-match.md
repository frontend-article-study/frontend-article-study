# 스위치문을 대체하는 또 다른 방법

클린 코드를 위해 코딩 방법론을 찾아보다보면

쉽게 이런 이야기들을 찾아볼 수 있습니다.

1. else if 를 사용하지마세요.

2. switch문을 사용하지마세요

아니 그러면 도대체 어떻게 코딩을 하라는걸까요?

우리는 스위치문이 필요한 로직이 있다는 것을 알고있습니다.

하지만 스위치문 없이 어떻게 구현을 하라는것일까요?

그냥 얼리리턴문을 사용하라는걸까요?



```js
const matched = (x) => ({
  on: () => matched(x),
  otherwise: () => x,
});
const match = (x) => ({
  on: (pred, fn) => (pred(x) ? matched(fn(x)) : match(x)),
  otherwise: (fn) => fn(x),
});

const a = match(50)
  .on(
    (x) => x > 0,
    () => console.log('0보다 커요'),
  )
  .on(
    (x) => x < 55,
    () => console.log('55보다 작아요'),
  )
  .otherwise((x) => x * 10); // 50 * 10을 하고 함수진행을 끝냅니다.


```