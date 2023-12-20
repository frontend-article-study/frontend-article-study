# 타입스크립트

타입스크립트는 개인적으로 시기에 따라 이런 느낌이 든다고 생각해요

1. 타입스크립트를 처음 접했을 때

-> 너무 불편해.. 이걸 왜 쓰는거야..

2. 어느정도 익숙해졌을 때

-> 완전 편한데?? 지금까지 어떻게 타입스크립트 없이 코딩했지?

3. 지금까지 알고있던 타입스크립트 지식으로 해결불가능 한 문제를 마주침

-> 절망

4. 3을 극복하고 좁혀진 타입이 주는 DX와 편안함을 느낌

-> 다시 기분 굿

5. 3,4를 반복하다보니 내가 타입스크립트의 기능들을 제대로 활용 못하고 있다는 걸 꺠닫게됨

-> 절망

# 템플릿 리터럴 타입

템플릿 리터럴 타입은 자바스크립트 개발자에게는 익숙할 `` 백틱을 통하여 표기하는 문법을 타입에서도 사용하는 것입니다.

이 멋진 기능은 TypeScript 4.1에서 릴리즈되었는데 사용법은 자바스크립트의 템플릿 리터럴 문법과 완전히 동일합니다.

```tsx
type FrontendArticleStudyMemeber =
  | 'wooeunhe'
  | 'giljong'
  | 'chaeun'
  | 'joonseo'
  | 'youngha'
  | 'joonseock'
  | 'woohyun';

type FrontendDeveloper = `${FrontendArticleStudyMember}Developer`;
```

이런 형태의 작업이 가능해집니다.

이런 템플릿 리터럴 타입은 여러가지 경우의수가 발생하는 경우 매우 유용하게 사용할 수 있습니다.

여러가지 타입이 중첩되어 필요한 경우를 생각해볼까요?

상상력이 풍부하실거라 생각하니 굳이 예제를 보여드리진 않겠습니다.

# 제네릭에는 조건식을 사용할 수 있다.

제네릭에는 조건식을 사용할 수 있습니다.

이것을 타입스크립트 진영에서는 Conditional Type 이라고 부릅니다.

예컨대 이런 식이지요

```tsx
type TrueType = 'hey';
type FalseType = string;
type GenericType<S extends string> = S extends 'hey' ? TrueType : FalseType;

const exampleString: GenericType<'hey'> = 'hey';
// 제네릭의 값이 hey 타입으로 extends가 가능하기 때문에 타입은 hey로 좁혀집니다.

const exampleString2: GenericType<'sdfs'> = 'hsfdl';
// 제네릭에 대한 삼항연산이 false이기 때문에 string 타입으로 추론됩니다.
```

# 조건식이 True일 때에는 infer 키워드를 사용할 수 있다.

조건식이 참으로 평가되는 경우에는 infer 키워드를 통하여 타입을 추론할 수 있습니다.

infer 키워드의 대표적인 유스케이스는 다음과 같은데요

```tsx
type UnWrappedPromise<T> = T extends Promise<infer U> ? U : never;

type A = UnWrappedPromise<Promise<'hey'>>; // A === "hey"
```

위 예제와 같이 T가 만약 Promise인 경우라면 프로미스의 반환타입을 infer 한뒤

반환 타입으로 사용하는 형식으로 프로미스 타입에서 반환 타입만을 추출할 수 있습니다.

만약 이것을 다른 형태로 작성해보고싶다면 어떻게할 수 있었을까요?

```tsx
type UnWrappedButNoInfer<T> = T extends Promise<T> ? T : never;

type B = UnWrappedButNoInfer<Promise<'hey'>>;
```

이렇게 작성하면 되지 않을까? 라고 생각할 수도 있습니다.

실제로 다음과같이 타이핑을 하면 never 타입으로 추론되게 되는데요

그 이유는 T 제네릭은 Promise<T>로 이미 확장이 가능하다는 것을 본 상태이기 때문입니다.

따라서 이 예제는 성립될 수 없으며 이를 infer 키워드 없이 해결하기 위해서는 한개의 제네릭이 더 필요합니다.

```tsx
type UnWrappedButNoInfer<T, U> = T extends Promise<U> ? U : never;

type B = UnWrappedButNoInfer<Promise<'hey'>, 'hey'>;
```

제네릭을 한개 더두는 것을 통해 동작하도록 할 수 있는데요

이 방법의 경우에는 추론이 이루어지지 않기때문에 개발자가 일일히 타입을 지정해야한다는 문제가 발생하겠죠?

infer 키워드는 이런 불편을 쉽게 해소시켜줍니다.

그럼 이제 이것을 활용해서 심화예제를 만들어보겠습니다.

# 심화 예제

```tsx
type TextColor = 'red' | 'blue' | 'wooeunhe' | 'primary';
type TextColorSaturation = '100' | '200' | '300' | '400' | '500' | '600';

type ExtractTextColor<WideS extends string> =
  WideS extends `text-${infer TColor}-${TextColorSaturation}` ? TColor : WideS;

type ExtractTextSaturation<Satu extends string> =
  Satu extends `text-${TextColor}-${infer TSatu}` ? TSatu : Satu;

const createTextColor = (
  color: ExtractTextColor<TextColor>,
  saturation: ExtractTextSaturation<TextColorSaturation>,
) => {
  return `text-${color}-${saturation}` as const;
};

const result = createTextColor('red', '500');
```

이제 간단하게 TextColor, TextColorSaturation 타입의 유니온만 추가해주면

타입세이프하게 css 클래스를 생성해주는 생성함수를 얻을 수 있습니다.

# 마치며

이처럼 템플릿 리터럴 타입은 타입 추가, 삭제를 위해 여러 파일을 건드려야하는 수고를 없애고

코어가 되는 하나의 타입만 관리하면 되는 형태로 타입을 만들 수 있다는 장점을 제공하는데요

기억해두시고 잘 활용해보시면 좋을 것 같습니다.

# 레퍼런스

https://toss.tech/article/template-literal-types

https://www.youtube.com/watch?v=TsYeBS6v4r8

냠냠맨 머리
