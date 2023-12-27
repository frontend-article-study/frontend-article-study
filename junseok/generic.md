## 제네릭 톺아보기

타입스크립트를 학습할 때 가장 처음으로 마주치는 난관이 제네릭이라고 생각합니다. 저는 처음 제네릭이라는 개념을 학습할 때, 해당 개념을 어떻게 정의해야 하는가?에서 부터 막혔던 경험이 듭니다. 그리고 이를 활용해야 할 경우 직접 고민하기 보다는 검색을 통해 대체를 했던 경험이 많았습니다. 그만큼 친해지는데 많은 시간이 걸린 녀석입니다. 

제네릭의 사전적인 의미로 “**일반적인**”이라는 뜻을 가지고 있습니다. 즉 뚜렷한 특징이 없는 일반적인 경우를 의미하는데, 왜 타입스크립트는 해당 녀석의 이름을 제네릭이라 부르는 걸까요? 

저는 “**일반적인 것**”을 타입에서 미리 정해두지 않고 외부에서 받은 녀석으로 대체한다. 즉 **유형이 어느 하나에 국한되지 않고 다양한 유형을 처리할 수 있는 유형이라는 컴퓨터 과학 및 프로그래밍 언어의 개념에서 파생** 되었다고 할 수 있습니다. 

이러한 제네릭의 장점은 **재사용성**입니다. 함수, 클래스 등의 여러 타입에 대해 따로따로 정의하지 않아도 되기 때문입니다. 

일반적으로 타입스크립트에서 제네릭은 `<T>` 와 같이 꺾쇠괄호 내부에 정의됩니다. **사용 시에는 함수에 매개 변수를 넣는 것과 같이 꺾쇠괄호 안에 사용할 타입을 넣어줍니다**. 보통 타입 변수 명으로 한 글자를 많이 사용합니다.

```
T(Type), E(Element), K(key), V(Value)
```

## 제네릭 함수

만약 제네릭 함수 호출시 타입을 명시하지 않을 경우 타입스크립트에서 인수를 통해 타입을 추론합니다.  혹은 제네릭 타입에 특정 값을 할당할 수도 있습니다.

```tsx
// 자동 추론 예시
[1, 2, 3, 4, 5].map(item => `${item}`);

// Array<number>.map<string> : string[]

// type map<U> = (callbackfn:(value: T, index:number, array: T[]), thisArg? : any) => U[]
```

```tsx
// 제네릭에 값을 할당하는 경우 
type ArrayNumber<T = number | string> = T[];
const arr: ArrayNumber = ['1'];

arr.map(item => `${item}`);
```

### extends (제한된 제네릭)

  **제네릭의 근본 의미는 일반화된 데이터 타입**입니다. 따라서 외부에서 제네릭 타입을 사용할 경우 어떤 타입이든 들어올 수 있다는 것을 알고 이를 적절히 활용할 수 있어야 합니다. 

  따라서 제네릭을 통해 특정 속성 Array.length 혹은 String.length 와 같은 값에 접근하려고 하면 에러가 발생합니다.  왜냐하면 어떤 타입에서 `length` 를 전달하는지 제네릭을 통해서는 알 수 없기 때문입니다. 

  ```tsx
  const getLength: <T>(arg: T) => number = arg => arg.length;
  // Property 'length' does not exist on type 'T'.ts(2339)
  ```

  따라서 제네릭의 값을 적절하게 좁혀주는 작업이 필요합니다. 이는 **`extends`** 를 통해 가능합니다. **`extends` 키워드를 사용하면 컴파일러에게 특정 타입의 하위 타입만 올수 있음을 확실하게 알려주게 됩니다. 이는 제네릭에 특정 타입을 상속하는 것을 의미합니다.**

  ```tsx
  interface LengthProperty {
    length: number;
  }

  const getLength: <T extends LengthProperty>(arg: T) => number = arg => arg.length;

  getLength(123);
  /* Argument of type 'number' is not assignable to parameter of type 
  'LengthProperty'.ts(2345) */

  getLength('123');

  getLength([1]);
  ```

  혹은 다음과 같이 사용할 수도 있습니다. 

  ```tsx
  interface LengthProperty {
    length: number;
  }
  type LengthFn<T extends LengthProperty> = (arg: T) => number;
  const getLength: LengthFn<string | any[]> = arg => arg.length;

  getLength(123);

  getLength('123');

  getLength([1]);
  ```

  또한 `**extends` 키워드를 통해 제네릭 값을 분기 처리**할 수 있습니다. 

  ```tsx
  // 배열의 첫 번째 값을 꺼내
  type First<T extends any[]> = T extends [] ? never : T[0];
  type OneToFour = [1, 2, 3, 4];
  const arr: OneToFour = [1, 2, 3, 4];

  const firstValue: First<OneToFour> = arr[0];
  const secondValue: First<OneToFour> = arr[1];
  // Type '2' is not assignable to type '1'.ts(2322)
  ```

## Class에서 제네릭 사용하기

제네릭 클래스에서도 제네릭을 사용하여 유연하게 클래스에서 다룰 타입을 지정할 수 있습니다. 

> **단, 클래스를 제네릭으로 관리할 때, static 정적 멤버는 제네릭으로 관리할 수 없습니다.**
> 

```tsx
class GenericNumber<T> {
  zeroValue: T;
  add: (x: T, y: T) => T;

  constructor(v: T, cb: (x: T, y: T) => T) {
    this.zeroValue = v;
    this.add = cb;
  }
}

const myGenericNumber = new GenericNumber(0, (x, y) => {
  return x + y;
});

const myGenericString = new GenericNumber('0', (x, y) => {
  return x + y;
});

myGenericNumber.zeroValue; // 0
myGenericNumber.add(1, 2); // 3

myGenericString.zeroValue; // '0'
myGenericString.add('hello ', 'world'); // 'hello world'

```

## 제네릭 예시

일반적으로 제네릭을 많이 활용하는 경우는 **API를 처리하는 경우**입니다. 타입스크립트에서 Promise의 반환 값은 **`Promise<any>`**로 처리합니다. 이는 HTTP 요청에서 Promise 반환 값이 어떤 값이 들어올지 모르기 때문입니다. 따라서 적절한 API 응답 타입을 지정해서 해당 요청을 쉽게 처리할 수 있습니다. 

```tsx
type Method = {
    postStarting: '/user/starting';
    getMbtmi: '/user/mbtmi';
    getAnswerVisiting: '/answer/visiting';
    userNumber: '/user/number';
    postResult: '/user/answers';
}

type ValueOf<T> = T[keyof T]
```

```tsx
// api.ts
import { instance } from '@/api/axios';
import { Method, ValueOf } from '@/types/types';
async function getApiWhitToken<T>(method: ValueOf<Method>, jwt: string) {
  try {
    const data = await instance.get<T>(`${method}`, {
      headers: {
        '-user-token': jwt,
      },
    });
    return data.data;
  } catch (e) {
    throw new Error('token을 처리하는 과정에서 문제가 발생했습니다.');
  }
}

// 활용사례
await getApiWhitToken<AnswerData>(END_POINT.getAnswerVisiting, token!)
```

## 제네릭 사용시 고려할점

- 제네릭은 코드 길이를 늘어나게 하고 가독성을 헤칩니다. 따라서 반드시 필요한 경우에만 사용을 하는 것이 좋습니다.
- 제네릭의 타입을 any로 할당할 경우, 제네릭을 사용할 필요가 없습니다.
    
    ```tsx
    type ReturnType<T = any> = T
    ```
    
- 무분별 하게 제네릭 타입을 사용하기 보다는 복잡한 제네릭은 의미 다위로 분할해서 사용하는 것이 좋습니다. (마치 들여쓰기를 제한하는 것과 같습니다.)

## Reference

**우아한 타입스크립트**

[📘 타입스크립트 Generic 타입 정복하기](https://inpa.tistory.com/entry/TS-📘-타입스크립트-Generic-타입-정복하기)