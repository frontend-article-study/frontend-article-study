# 정말 많이 일어나는 상황

같은 타입안에서도 특정 프로퍼티의 값에 따라

가져야할 프로퍼티들이 달라지는 경우가 존재합니다.

이런 경우 타입스크립트를 만족시키는 타입을 작성하기가 꽤 까다로워지곤하는데요

type의 특성 중 하나인 intersectionType을 이용하면

아주 유용하게 해당 사용 사례를 구현할 수 있습니다.

# type의 & 연산자를 이용하여 구현하기

```tsx
type Props = {
  name: string;
} & (MaleProps | FemaleProps);

type MaleProps = {
  gender: 'male';
  salary: number;
};

type FemaleProps = {
  gender: 'female';
  weight: number;
};

const Test: Props = {
  name: 'hi',
  gender: 'male',
  salary: 123,
};

const Female: Props = {
  name: 'hi',
  gender: 'female',
  weight: 412,
};
```

이렇게 구현된 Props 타입은 gender의 값에 따라

salary / weight이 선택적으로 require 됩니다.

gender가 female일 때에는 weight이

gender가 male일 때에는 salary가 require 되는 식이지요

이러한 기법을

"tagged unions" 혹은 "discriminated unions"라고 부릅니다.

이 기법은 성별과 같이 특정 프로퍼티에 따라 차별적인 프로퍼티를 가져야하는 경우에 매우 유용합니다.

잘 기억해두시면 나중에 분명히 잘 쓰실거에요

# 번외

위에서 type으로 보여드린 기법은 인터페이스로도 비슷하게 구현할 수 있는데요

두 방식 모두 장단점이 있는 방식이니 상황에 맞는것을 취사선택하시면 되겠습니다.

아래에서 보여드릴 것은 인터페이스의 상속을 활용한 기법입니다.

제가 첫 면접을 봤던 회사에서 과제로 나왔던 타입기법이어서 머리에 아주...잘 남아있네요..

```tsx
interface IProps {
  gender: 'male' | 'female';
}

interface IMaleProps extends IProps {
  gender: 'male';
  salary: number;
}

interface IfFemaleProps extends IProps {
  gender: 'female';
  weight: number;
}
```

이렇게 상속으로 구현된 타입들에 유니온을 적용하면 위와 같은 동작을 할 수 있는데요

```tsx
const IFemale: IMaleProps | IFemaleProps = {
  gender: 'female',
  weight: 123,
};
const IMale: IMaleProps | IFemaleProps = {
  gender: 'male',
  salary: 123,
};
```

이렇게 작성할 수 있습니다.

# 마치며

타입의 세계는... 심오합니다...

# 레퍼런스

https://www.youtube.com/watch?v=9i38FPugxB8
