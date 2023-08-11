# What is Compound Component?

대표적인 사례로 radix 라이브러리의 코드를 보면 알 수 있습니다.

```tsx
import * as React from 'react';
import * as Popover from '@radix-ui/react-popover';

const PopoverDemo = () => (
  <Popover.Root>
    <Popover.Trigger>More info</Popover.Trigger>
    <Popover.Portal>
      <Popover.Content>
        Some more info…
        <Popover.Arrow />
      </Popover.Content>
    </Popover.Portal>
  </Popover.Root>
);

export default PopoverDemo;
```

이러한 컴파운드 컴포넌트 패턴은 복잡한 컴포넌트를 만들기 위해

작은 컴포넌트들을 합성하는 구조로 이루어집니다.

html로 눈을 돌려보면 select와 option의 관계 역시도 compound component 패턴을 따랐다고 볼 수 있습니다.

```html
<select>
  <option value="1">Option 1</option>
  <option value="2">Option 2</option>
</select>
```

이 `select` 태그와 `option` 태그는 각각의 독립적인 태그로서는 큰 의미가 없지만

이를 조합하여 사용하는 것을 통해 화면에 의미 있는 요소가 됩니다.



# what is value of Compound Component?

컴파운드 컴포넌트 패턴의 아름다운 점은 두가지 정도로 요약할 수 있습니다.

1. SRP 충족이 쉬워진다.

컴파운드 컴포넌트 패턴을 사용하면 자연스럽게 각각의 컴포넌트들이 하나의 일만 수행하도록 구성됩니다.

2. 변경에서 자유로워진다.

컴파운드 컴포넌트 패턴을 채택한 경우 어떠한 상황에서든 필요에 따른 UI와 기능을 구현할 수 있습니다.


# 구현 방법

일반적으로 Compound Component 패턴은 컨텍스트를 통하여 구현됩니다.

주로 컨텍스트(상태)를 가진 부모 컴포넌트와

상태를 제공 받아 사용하고 업데이트하는 자식 컴포넌트로 구성됩니다.


# Compound Component를 잘 만들려면?

1. 컴포넌트 간의 관계와 결합 방법에 대해 신중하게 생각해야 합니다.

2. 컴포넌트의 기본 구조를 식별하고 더 작고 관리하기 쉬운 부분으로 잘게 쪼갭니다.

3. 사용자가 사용 방법과 옵션을 이해할 수 있도록 명확한 API를 제공하는 것이 중요합니다.

4. 일부 컴포넌트를 사용하지 않거나 잘못 합성하더라도 잘 동작할 수 있도록 합리적인 기본 값을 제공해야합니다.

5. props을 통해 너무 많은 데이터를 전달하지 않는 게 중요합니다.

6. 종종 컴포넌트 간 상태 공유를 위해 React.ContextAPI와 함께 사용할 수 있습니다.

7. shouldComponentUpdate 혹은 메모이제이션훅 등을 사용하여 불필요한 렌더링을 피합니다.

8. Render Props / High-Order Components 패턴 등을 결합하여 사용하는 것을 통해 더 강력하고 유연한 컴포넌트를 만들 수 있습니다.


# 실제 구현

```tsx
import React from 'react';
interface ChildrenProp {
  children: React.ReactNode;
}

export const Modal = ({ children }: ChildrenProp) => {
  const childrenArray = React.Children.toArray(
    children,
  ) as React.ReactElement<ChildrenProp>[];
  const header = childrenArray.find((child) => child.type === Modal.Header);
  const body = childrenArray.find((child) => child.type === Modal.Body);
  const footer = childrenArray.find((child) => child.type === Modal.Footer);

  console.log(childrenArray);
  return (
    <div className="modal">
      <div className="modal-header">{header}</div>
      <div className="modal-body">{body}</div>
      <div className="modal-footer">{footer}</div>
    </div>
  );
};

Modal.Header = ({ children }: ChildrenProp) => {
  return <div className="modal-header">{children}</div>;
};

Modal.Body = ({ children }: ChildrenProp) => {
  return <div className="modal-body">{children}</div>;
};

Modal.Footer = ({ children }: ChildrenProp) => {
  return <div className="modal-footer">{children}</div>;
};

```


주의 할 점은 타입 단언을 통하여 `React.ReactElement` 일 것을 단언해주어야 한다는 것입니다.

```tsx

(string | number | React.ReactElement<any, string | React.JSXElementConstructor<any>> | Iterable<React.ReactNode> | React.ReactPortal)[]

```

React.Children.toArray()의 반환값은 위와 같은 유니온 형태로 되어있기 때문입니다.



# 레퍼런스

https://fe-developers.kakaoent.com/2022/220731-composition-component/

https://fe-developers.kakaoent.com/2021/211022-react-children-tip/

https://itchallenger.tistory.com/928

https://react.dev/reference/react/Children#alternatives
