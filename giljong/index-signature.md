# 인덱스 시그니처

타입스크립트를 시작하고 마주치는 가장 큰 벽이라고도 할 수 있을 것 같습니다.

만약.. 이 문제를 겪어보지 않은 분이라면 타입을 느슨하게 작성하고 있을 확률이 높습니다.

왜냐하면 타입을 엄격하게 작성하면 작성할수록 인덱스시그니처는 항상 문제가 되고

느슨한 타입 하나가 모든 인덱스 시그니처를 망치는 결과를 낳기 때문입니다.

# 객체의 키를 좁히는 방법

레코드 타입을 사용하기

```tsx

type Hello = Record<'hello'|'hi" , string>

const hello:Hello = {
  hello:'hello',
  hi:'hi'
}

type Hello = {[key in 'hello' | 'hi']:string}
```

이와 같이 사용할 수 있습니다.

이렇게 인덱스시그니처를 이용하여 객체를 원하는 형태로 최대한 좁힐 수 있는데요



문제는 이런 상황에서도 발생합니다.

# Object.keys와 같은 객체 정적 메서드 사용시 타입이 넓혀지는 문제

```tsx
object.keys(hello).map((item) => {
    const value = hello[item]
    return value
})

```

이 코드는 개발자가 생각하기에는 매우 정상적으로 동작할 수 밖에 없는 코드입니다.

하지만 ojbect.keys의 반환 타입은 string[]이기 때문에

내부 요소의 타입들은 hello의 인덱스 시그니처가 아니게됩니다.

따라서 타입스크립트는 에러를 반환하는 것입니다.

이것을 해결하기 위해서 타입 단언을 사용할 수도 있습니다.


```tsx
(Object.keys(hello) as Array<keyof Hello>).map(item => {
    const value = hello[item]
    return value
})
```

정적메서드를 한번 래핑한 커스텀 함수를 만드는것으로 해결할수도 있습니다.

```tsx
function keysOf<T extends Object>(obj:T):Array<typeof T> {
    return Array.from(Object.keys(obj) as Array<keyof T>)
}

keysOf(hello).map(item => {
    const value = hello[item]
    return value
})
```

제네릭을 이용하면 적절히 추론시킬 수 있겠죠?

# 왜 이따구로 만들어놓은거임??

타입스크립트는 덕 타이핑 (구조적 서브타이핑) 구조를 채택하고 있기 때문입니다.

타입 체크를 할 때에 그 값이 가진 형태에 집중한다는 것인데요

자바스크립트는 객체의 타입에 구애받지 않고 객체의 타입에 열려있기 떄문에

타입스크립트 역시도 이런 자바스크립트의 특성에 맞춰서 타입을 추론해줘야했어서

가장 열린타입인 string[]을 반환하게 된 것으로... 보여집니다..