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

type FrontendDeveloper = `${FrontendArticleStudyMember}Developer`
```

이런 형태의 작업이 가능해집니다.

이런 템플릿 리터럴 타입은 여러가지 경우의수가 발생하는 경우 매우 유용하게 사용할 수 있습니다.

여러가지 타입이 중첩되어 필요한 경우를 생각해볼까요?

상상력이 풍부하실거라 생각하니 굳이 예제를 보여드리진 않겠습니다.


# infer 키워드

제네릭에는 조건식을 사용할 수 있습니다.


# 마치며

# 레퍼런스

https://toss.tech/article/template-literal-types
