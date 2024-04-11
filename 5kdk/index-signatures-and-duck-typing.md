# 타입스크립트의 인덱스 시그니처와 Object.keys 그리고 덕 타이핑

_Element implicitly has an 'any' type because expression of type 'string'_  
_can't be used to index type 'data'_  
_No index signature with a parameter of type 'string' was found on type 'Data'_

객체의 키 배열을 뽑아냈지만, `string[]` 타입으로 강제되어 위와 같은 에러를 만난 적이 있을 것이다.

이를 이해하기 위해 타입스크립트 인덱스 시그니처와 자바스크립트 덕 타이핑 관계를 알아보자.

<br />

## 인덱스 시그니처

우선 인덱스 시그니처란 객체의 키를 정의하는 방식을 의미한다.

```ts
type Hello = {
  [key: string]: string;
};

const hello: Hello {
  hello: 'hello',
  hi: 'hi'
}
```

`[key: string]`을 시용한 부분이 바로 인덱스 시그니처다. 이는 동적인 객체를 정의할 때 유용하지만, 위의 예시와 같이 작성한다면 키의 범위가 너무 커지기 때문에 존재하지 않은 키로 접근하면 `undefined`를 반환할 수 도 있다.

```ts
hello['bye']; // undefined
```

따라서 객체의 키는 동적으로 선언되는 경우를 최대한 지양해야 하고, 객체의 타입도 필요에 따라 좁혀야 한다.

객체의 키를 좁히는 방법은 다음 두가지다.

1. 유틸리티 타입을 활용한 예시

   ```ts
   type Hello = Record<'hello' | 'hi', string>;
   ```

2. 타입을 사용한 인덱스 시그니처

   ```ts
   type Hello = { [key in 'hello' | 'hi']: string };
   ```

`Record<key, value>`를 사용하면 객체의 타입에 각각 원하는 키와 값을 넣을 수 있다. 그리고 인덱스 시그니처에 타입을 사용함으로써 객체를 원하는 형태로 최대한 좁힐 수 있다.

<br />

## 인덱스 시그니처를 사용할 때 마주하는 이슈

```ts
Object.keys(hello).map(key => {
  // Element implicitly has an 'any' type because expression of type 'string'
  // can't be used to index type 'hello'
  // No index signature with a parameter of type 'string' was found on type 'Hello'
  const value = hello[key];
  return value;
});
```

`hello` 객체의 key를 `Object.keys`로 잘 뽑아냈고, 그 키로 객체에 접근할 때 에러가 발생하는데 이러한 문제는 왜 발생할까?

우선 `Object.keys(hello)`가 반환하는 타입을 잘 살펴봐야한다.

```ts
// string[]

const temp = Object.keys(hello);
```

`Object.keys`는 `string[]`를 반환하고 있기 때문에 `Hello` 타입의 객체를 인덱스 키로 접근할 수 없기 때문이다.

이 문제를 해결하기 위한 방법은 다양하다.

1. `Obejct.keys(hello)`를 `as`로 타입 단언

   ```ts
   (Object.keys(hello) as Array<keyof Hello>).map(key => {
     const value = hello[key];
     return value;
   });
   ```

2. 타입스크립트의 `Object.keys`에 대한 반환 타입을 `string[]` 대신 개발자가 단언한 타입으로 강제하는 방법

   ```ts
   // Object.keys를 대신할 keyOf util 함수 생성
   function keysOf<T extends Object>(obj: T): Array<keyof T> {
     return Array.from(Object.keys(obj)) as Array<keyof T>;
   }

   keyOf(hello).map(key => {
     const value = hello[key];
     return value;
   });
   ```

   `keyOf` 함수는 객체의 키를 가지고 오면서 동시에 가져온 배열에 대해서도 마찬가지로 타입 단언으로 처리하는 과정을 거친다.

3. 가져온 key를 단언 하는 방법

   ```ts
   Object.keys(hello).map(key => {
     const value = hello[key as keyof Hello];

     return value;
   });
   ```

<br />

## Object.keys가 string[]으로 강제된 이유

결론부터 이야기하면 이는 자바스크립트의 특징과, 이를 구현하기 위한 타입스크립트의 구조적 타이핑(덕 타이핑)의 특징 때문이다.

자바스크립트는 다른 언어에 비해 객체가 열려 있는 구조로 만들어져 있으므로 덕 타이핑으로 객체를 비교해야 하는 특징이 있다.

> **덕 타이핑**
>
> 객체의 타입이 클래스 상속, 인터페이스 구현 등으로 결정되는 것이 아닌 어떤 > 객체가 필요한 변수와 메서드만 지니고 있다면 그냥 해당 타입에 속하도록 인정해 > 주는 것을 의미한다.
>
> [위키피디아 - 덕타이핑](https://ko.wikipedia.org/wiki/%EB%8D%95_%ED%83%80%EC%9D%B4%ED%95%91)

이러한 특징 때문에 타입스크립트 인터페이스 소개란에는 다음과 같은 문장이 등장한다.

> One of TypeScript's core principles is that type checking focuses on the shape that values have. This is sometimes called “duck typing” or “structural typing”.  
> 타입스크립트의 핵심 원칙 중 하나는 타입 검사가 값의 모양에 초점을 맞춘다는 것입니다. 이를 "덕 타이핑" 또는 "구조적 타이핑"이라고 부르기도 합니다.
>
> https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html

타입스크립트의 핵심 원칙은 타입 체크를 할 때 그 값이 가진 형태에 집중한다는 것이다. 이러한 것은 덕 타이핑 또는 구조적 타이핑이라고 한다.

<br />

## 덕 타이핑과 Object.keys의 상관관계

```ts
type Car = { name: string };
type Truck = Car & { power: number };

function horn(car: Car) {
  console.log(`${car.name}이 경적을 울립니다.`);
}

const truck: Truck = {
  name: '트럭',
  power: 100,
};

horn(truck); // Car에 필요한 속성은 모두 가지고 있기 때문에 Car처럼 name을 가지고 있으므로 정상 동작한다.
```

이처럼 자바스크립트는 객체의 타입에 구애받지 않으며 타입에 열려 있으므로 타입스크립트도 이러한 자바스크립트의 특징을 맞춰주어야 한다.

즉, 타입스크립트는 이렇게 모든 키가 들어올 가능성이 열려 있는 객체의 키에 포괄적으로 대응하기 위해 `string[]`으로 타입을 제공하는 것이다.

<br />

> **참고 자료:**
>
> - [위키피디아](https://ko.wikipedia.org/wiki/%EB%8D%95_%ED%83%80%EC%9D%B4%ED%95%91)
> - [모던 리액트 딥다이브](https://www.yes24.com/Product/Goods/123161563)
> - [타입스크립트 공식문서](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html)
