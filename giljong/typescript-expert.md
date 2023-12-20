# 타입스크립트

타입스크립트는 개인적으로 시기에 따라 이런 느낌이 든다고 생각해요

1. 타입스크립트를 처음 접했을 때

-> 너무 불편해.. 이걸 왜 쓰는거야..

2. 어느정도 익숙해졌을 때

-> 완전 편한데?? 지금까지 어떻게 타입스크립트 없이 코딩했지?

3. 지금까지 알고있던 타입스크립트 지식으로 해결

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

# 마치며

# 레퍼런스

https://toss.tech/article/template-literal-types

https://www.youtube.com/watch?v=TsYeBS6v4r8
