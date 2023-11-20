# object.is 란?

object.is는 두 값이 같은 값인지 결정하는 ES6에 추가된 문법입니다.

NaN, +- 등 ===연산자로도 구분하기 어려운 미-묘한 녀석들을 잘 비교할 수 있게 해주는 메서드인데요

ojbect.is와 ===의 유일한 차이점은

앞서 소개한 미-묘한 녀석들의 처리에 있습니다.

```tsx
// Case 1: 평가 결과는 ===을 사용한 것과 동일합니다
Object.is(25, 25); // true
Object.is("foo", "foo"); // true
Object.is("foo", "bar"); // false
Object.is(null, null); // true
Object.is(undefined, undefined); // true
Object.is(window, window); // true
Object.is([], []); // false
const foo = { a: 1 };
const bar = { a: 1 };
const sameFoo = foo;
Object.is(foo, foo); // true
Object.is(foo, bar); // false
Object.is(foo, sameFoo); // true

// Case 2: 부호 있는 0
Object.is(0, -0); // false
Object.is(+0, -0); // false
Object.is(-0, -0); // true

// Case 3: NaN
Object.is(NaN, 0 / 0); // true
Object.is(NaN, Number.NaN); // true
```

예제로 보면 이해가 쉬운데 다음과 같습니다.

# 그러나 객체 비교는 여전하다.

```tsx
Object.is({},{}) // false
```

객체를 비교할 때에는 주소값을 기준으로 같은 객체인지 비교하기 때문에

비교가 미-묘합니다.


# 이거 왜 설명함?

리액트에서 동등 비교를 하는 방법이 바로 Object.is를 이용한 방법이기 때문입니다.

그런데 앞서 설명했듯이 object.is는 ES6 에 새로 등장한 문법입니다.

따라서 object.is를 사용할 수 없는 환경에서는 폴리필을 주입하는 식으로 사용하고있습니다.

다음은 리액트에서 사용하는 objectis 함수의 코드입니다.

react/packages/shared/objectIs.js

```tsx
/**
 * Copyright (c) Meta Platforms, Inc. and affiliates.
 *
 * This source code is licensed under the MIT license found in the
 * LICENSE file in the root directory of this source tree.
 *
 * @flow
 */

/**
 * inlined Object.is polyfill to avoid requiring consumers ship their own
 * https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is
 */
function is(x: any, y: any) {
  return (
    (x === y && (x !== 0 || 1 / x === 1 / y)) || (x !== x && y !== y) // eslint-disable-line no-self-compare
  );
}

const objectIs: (x: any, y: any) => boolean =
  // $FlowFixMe[method-unbinding]
  typeof Object.is === 'function' ? Object.is : is;

export default objectIs;
```

flow는 facebook에서 개발한 오픈소스로

타입스크립트와 비슷하게 자바스크립트에서 타입 시스템을 사용할 수 있게 해주는 프로젝트입니다.

만약 Object.is가 함수라면 es6를 사용할 수 있는 환경이라는 것이고

그렇지 않다고하면 es6를 사용할 수 없는 환경이기 때문에 폴리필을 주입하는 일을

objectIs 함수에서 수행하는것입니다.


react/packages/shared/shallowEqual.js

```tsx
/**
 * Copyright (c) Meta Platforms, Inc. and affiliates.
 *
 * This source code is licensed under the MIT license found in the
 * LICENSE file in the root directory of this source tree.
 *
 * @flow
 */

import is from './objectIs';
import hasOwnProperty from './hasOwnProperty';

/**
 * Performs equality by iterating through keys on an object and returning false
 * when any key has values which are not strictly equal between the arguments.
 * Returns true when the values of all keys are strictly equal.
 */
function shallowEqual(objA: mixed, objB: mixed): boolean {
  if (is(objA, objB)) {
    return true;
  }

  if (
    typeof objA !== 'object' ||
    objA === null ||
    typeof objB !== 'object' ||
    objB === null
  ) {
    return false;
  }

  const keysA = Object.keys(objA);
  const keysB = Object.keys(objB);

  if (keysA.length !== keysB.length) {
    return false;
  }

  // Test for A's keys different from B.
  for (let i = 0; i < keysA.length; i++) {
    const currentKey = keysA[i];
    if (
      !hasOwnProperty.call(objB, currentKey) ||
      // $FlowFixMe[incompatible-use] lost refinement of `objB`
      !is(objA[currentKey], objB[currentKey])
    ) {
      return false;
    }
  }

  return true;
}

export default shallowEqual;
```

이제 리액트는 위에서 만들어둔 Object.is를 기반으로 얕은 비교를 수행하는

shallowEqual 함수를 만들어 사용합니다.

그리고 이 shallowEqual을 통하여 의존성 비교와 같이 같은 값인지 비교할때 사용하게되는것이지요


# 근데 뭐하러 얕은 비교만 수행할 수 있게 만듬?

아니 근데 얕은 비교만 하면 무슨 의미가 있나요?

기왕 만들었으면 깊은 비교까지해서 완전히 똑같은지 체크하는게 당연히 더 좋은거아님??

이라고 생각할 수도 있는데요

내부에 있는 객체까지 완벽하게 비교하기 위해서는 우선 내부의 객체가 얼마나 중첩되어있을지

코드를 작성하는 입장에서 예측할수 없기 때문에 재귀문을 사용하게 됩니다.

그리고 아주 깊은 중첩의 객체를 props, 의존성배열 등에 넘겨주게 되면

깊이를 알수없는 객체의 심연을 향해 한무 비교를 수행할 것이고 이는 성능에 악영향을 미칠것입니다.

게다가 대부분의 경우에는 러프하게 얕은비교만해도 충분하니까

얕은 비교만 수행하게 짜는게 더 이득인것이지요

# 자바스크립트의 특성이라..

객체 비교가 불완전할 수 밖에 없는 것은 언어적인 한계이기 때문에

어쩔수가 없으니 그와중에 타협점을 찾아 구현하는 방식이 이런 얕은 비교라고 할 수 있습니다.