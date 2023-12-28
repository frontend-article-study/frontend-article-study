# 타입 가드

태그: typescript
날짜: 2023년 12월 22일

## 타입 좁히기 (타입 가드)

타입스크립트에서 타입 좁히기는 변수 또는 표현식의 범위를 좁혀나가는 것을 의미합니다. 이를 통해 더 정확하고 명시적인 타입 추론을 할 수 있게 되고, 복잡한 타입을 작은 범위로 축소해서 타입 안전성을 높일 수 있습니다. 

기본적으로 자바스크립트 연산자를 활용한 타입 가드는 `**typeof**`, `**instanceof**`, `**in**`과 같은 연산자를 사용해서 제어문으로 특정 타입 값을 가질 수 없에 없는 상황을 유도하는 방법을 의미합니다.  이러한 자바스크립트 연산자를 사용한 이유는 런타임에 유효한 타입 가드를 만들기 위해서 입니다. 

### typeof

- 원시 타입(`number, bigint, string, object, function, boolean, symbol, null, undefined`)을 추론 할 때 사용합니다.
    
    ```tsx
    const checkLength = (arg: unknown): number | void => {
      if (typeof arg === 'string' || Array.isArray(arg)) return arg.length;
      alert('배열과 문자열만 입력해주세요');
    };
    ```
    

### instanceof

- 해당 연산자는 인스턴스화된 객체 타입을 판별하는 타입 가드로 사용할 수 있습니다. **`A instanceof B`** 형태로 사용이 됩니다.  A에는 타입을 검사할 대상 변수, B에는 특정 객체의 생성자가 들어갑니다.
    
    프로토타입 체인 상에 B 생성자가 존재한다면 true, 그렇지 않다면 false를 반환합니다. 해당 예시는 Enter를 눌렀을 때와, Shift + Enter를 눌렀을 경우를 분리해서 Enter를 눌렀을 경우만을 제어합니다. 
    
    ```tsx
    export default function App() {
      const handleInput = (e: React.KeyboardEvent) => {
        const { target } = e;
        const { key } = e;
    
        if (target instanceof HTMLInputElement && key === 'Enter' && !e.shiftKey) {
          alert('Enter');
        }
      };
    
      return <input type="text" onKeyUp={handleInput} />;
    }
    ```
    

### in

in 연산자는 객체에 속성이 있는지 확인하고 true 또는  false를 반환합니다. in 연산자를 통해 객체를 구분할 수 있습니다. in 연산자는 `**A in B`** 라는 형태로 사용합니다. A 객체 내부에 B라는 속성이 있는지 확인하게 됩니다. 

**자바스크립트의 in 연산자는 런타임의 값 만을 검사하지만, 타입스크립트에서는 객체 타입에 속성이 존재하는지를 검사합니다.** 

아래의 예시는 날짜 표기 시, 시간을 함께 표기하는 여부를 time 속성을 통해서 분기 처리하는 예시입니다. 

```tsx
interface Date {
  dateId: number;
  date: string;
}

interface Time extends Date {
  time: number;
}

type TimeAndDate = Time | Date;

function TimeDialog(props: TimeAndDate) {
  if ('time' in props) return <DateWithTime {...props} />;
  return <Date {...porps} />;
}
```

## is 연산자로 사용자 정의 타입 가드 활용

is 연산자는 직접 타입 가드 함수를 만들어 적용할 때 활용합니다. 이러한 방식의 타입 가드는 반환 타입이 타입 명제(type predicates)인 함수를 정의해서 사용할 수 있습니다. 

타입 명제는 `**A is B`** 형식으로 작성합니다. A는 매개변수의 이름, B는 타입입니다. 참/거짓을 반환하면서 **반환 타입을 타입 명제로 지정하게 되면 반환 값이 참일 때 A 매개변수의 타입을 B 타입으로 취급**하게 됩니다. 

1. **반환 값을 true, false로 처리하는 경우**
    
    ```tsx
    type FoodKey = 'pizza' | 'chicken' | 'hamburger';
    
    const food: Record<FoodKey, number> = {
      pizza: 3,
      chicken: 3,
      hamburger: 4,
    };
    
    const isFoodKey = (menu: string) =>
      menu === 'pizza' || menu === 'chicken' || menu === 'hamburger';
    
    const calculateFood = (
      name: string,
      count: number,
      method: 'plus' | 'minus',
    ) => {
      if (isFoodKey(name)) {
        method === 'plus' ? (food[name] += count) : (food[name] -= count);
        food[name] < 0 ? (food[name] = 0) : null;
    		/*
    			Element implicitly has an 'any' type 
    			because expression of type 'string' 
    			can't be used to index type 'Record<FoodKey, number>'.
    		*/
      } else {
        alert('치킨, 피자, 햄버거만 수량 변경할 수 있습니다.');
      }
    };
    ```
    
2. 반환 값을 타입 명제를 통해 타입을 명시하는 경우
    
    ```tsx
    type FoodKey = 'pizza' | 'chicken' | 'hamburger';
    
    const food: Record<FoodKey, number> = {
      pizza: 3,
      chicken: 3,
      hamburger: 4,
    };
    
    const isFoodKey = (menu: string): menu is FoodKey =>
      menu === 'pizza' || menu === 'chicken' || menu === 'hamburger';
    
    const calculateFood = (
      name: string,
      count: number,
      method: 'plus' | 'minus',
    ) => {
      if (isFoodKey(name)) {
        method === 'plus' ? (food[name] += count) : (food[name] -= count);
        food[name] < 0 ? (food[name] = 0) : null;
      } else {
        alert('치킨, 피자, 햄버거만 수량 변경할 수 있습니다.');
      }
    };
    ```
    
    타입 명제 is를 통해 타입을 명시할 경우, 인자로 전달된 name이 true일 경우 타입스크립트에서 해당 name 인자의 type을 FoodKey로 인식을 하게 됩니다. 
    

## 식별할 수 있는 유니온

식별할 수 있는 유니온은 유사한 역할을 하는 객체 타입을 하나로 관리할 때 유용합니다. 이는 각 객체에 고유한 값을 할당해서, 유니온 타입으로 다양한 객체를 관리하는 경우 해당 객체의 범위를 좁히는 역할을 할 수 있습니다. 

가령 다음과 같은 대화창을 표시해야 합니다. 

<img src="./type_guard1.png" width="300" height="500">

이때 이미지일 경우 `<img>` 태그를, text일 경우 `<span>` 태그를 사용해야 합니다. 이 외에 추가적으로 날짜 표시, 내가 작성한 답변등을 표시해야 할 수 있습니다. 이를 type으로 다음과 같이 관리하고 있습니다. 

```tsx
type YourMessage = {
  type: 'your';
	id: 'message' | 'image'
  user: string;
  time: string;
  profile: string;
};

type MyMessage = {
  type: 'my';
	id: 'message' | 'image'
  user: string;
  time: string;
  profile: string;
};

type Date = {
  type: 'date';
  date: string;
};

export type MessageOrDate = YourMessage | Date | MyMessage;
```

이때 각각의 타입을 유니온으로 묶어 `MessageOrDate` 값을 활용하고자 합니다. 

이때 해당 유니온이 어디 값에 속하는지를 type과 id 값으로 파악할 수 있습니다. 

```tsx
if (item.type === 'my') {
    return item.id === "message" ? <MyMessage /> : <MyImage />
  }
if (item.type === 'your') {
		return item.id === "message" ? <YourMessage /> : <YourImage />
}
if (item.type === 'date') {
		return <ChatDate />
}
```

이처럼 식별할 수 있는 판별자를 만들기 위해서는 다음과 같은 규칙을 지켜야 합니다. 

- **리터럴 타입이어야 합니다.**
- **판별자로 선정한 값에 적어도 하나 이상의 유닛 타입이 포함되어야 하며, 인스턴스화 할 수 있는 타읍은 포함되지 않아야 합니다.**

이때 모든 분기에 대한 타입 처리가 필요하다면 `**Exhaustiveness Checking`** 방법을 활용합니다.  **`Exhaustiveness`** 는 사전적으로 ‘철저함’, ‘완전함’을 의미합니다. 즉 모든 케이스에 대해 철저하게 타입을 검사하는 것을 의미합니다. 이는 만약 모든 분기에 대해 케이스 처리를 하지 않을 경우 에러를 만듭니다.

```tsx
type FoodKey = 'pizza' | 'chicken' | 'hamburger';

const food: Record<FoodKey, number> = {
  pizza: 3,
  chicken: 3,
  hamburger: 4,
};

const exhaustivenessCheck = (param: never) => {
  throw new Error('type Error');
};

const checkFood = (name: FoodKey): number => {
  if (name === 'chicken') {
    return food[name];
  }
  if (name === 'pizza') {
    return food[name];
  } 
	// 햄버거에 대한 분기를 제외한 상황입니다. 
	else {
    exhaustivenessCheck(name);
		/* name 인자에서 다음과 같은 에러가 발생합니다. 
			 Argument of type 'string' is not assignable to parameter of type 'never'.
			 name: "hamburger"
		*/	
    return 0;
  }
};
```

`**exhaustivenessCheck`**  함수는 매개변수를 nerver로 설정하고 있습니다. 즉 매개변수에서 어떤 값도 받아서는 안됩니다. 하지만 햄버거에 대한 분기처리를 제외해서 인자에 받을 수 있는 값이 있기 때문에 에러가 발생하게 됩니다. 

`**Exhaustiveness Checking`** 방법을 활용해서 예상치 못한 런타임 에러를 방지하고 요구사항 변경시 생길 수 있는 위험을 줄일 수 있습니다. 타입에 대한 철저한 분기에서 유용하게 활용할 수 있습니다.

## NonNullable 유틸리티 타입 활용한 타입 가드 구성하기

[Documentation - Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html#nonnullabletype)

is 키워드와 `NonNullable` 유틸리티 타입을 활용해서 undefined와 null 값을 처리할 수 있습니다. 

`NonNullable` 타입은 제네릭으로 받는 T가 null 또는 undefined 일때 never 또는 T를 반환합니다. 

```tsx
type NonNullable<T> = T extends null | undefined ? never : T;
```

해당 유틸리티 타입을 활용해서  해당 인자가 null 혹은 undefined인지 검사하는 타입 가드 함수를 만들 수 있습니다.

```tsx
function nonNullable<T>(value : T): value is NonNullable<T>{
	return value !== null && value !== undefined
}
```

해당 처리는 API 작업에서 활용할 수 있습니다.  아래의 api 처리 코드는 에러가 없을 시 해당 데이터를 그렇지 않은 경우 null을 반환합니다.

```tsx
async function getApiWhitToken<T>(method: ValueOf<Method>, jwt: string) {
  try {
    const data = await instance.get<T>(`${method}`, {
      headers: {
        '-user-token': jwt,
      },
    });
    return data.data;
  } catch () {
    return null
  }
}
```

이때 반환되는 데이터 형식은 `T | null` 입니다.  만약 해당 api 처리를 배열 내의 모든 데이터 `[{nickname: “son”, token: “1kfoq”}, {nickname: “sn”, token: “1kfoqc”}, {nickname: “yabi”, token: “1kfoqqw”}]` 에 대해서 일괄적으로 처리할 경우 반환된 배열에 대해서 모두 if 문을 통해 타입 가드를 해야합니다. 하지만 위의 noneNullable을 통해 효율적으로 데이터를 처리할 수 있습니다. 

```tsx
const data = [
    { nickname: 'son', token: '1kfoq' },
    { nickname: 'sn', token: '1kfoqc' },
    { nickname: 'yabi', token: '' },
  ];
  const result = await Promise.all(
    data.map(info =>
      getApiWhitToken<AnswerData>(END_POINT.getAnswerVisiting, info.token),
    ),
  );
  const sucessData = result.filter(nonNullable); // 데이터의 결과가 null인 값을 모두 제거
```


## Reference

우아한 타입스크립트