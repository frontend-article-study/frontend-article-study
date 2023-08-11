# what is compound?

compound는 복합체(명사) , 합성의(형용사)로 정의되어있습니다.



# What is Compound Component?

즉 compound component는 합성할 수 있는 컴포넌트

혹은 복합체 컴포넌트 정도로 번역할 수 있습니다.

이러한 compound component 패턴을 따르는

대표적인 사례로 radix 라이브러리의 코드를 들 수 있습니다.

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

컴파운드 컴포넌트 패턴은 복잡한 컴포넌트를 만들기 위해

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


compound component 패턴을 사용하는 것은 즉 변경에 유연한 컴포넌트를 만들어야한다.

라는 가치를 따르기 위해서라고 볼 수 있습니다.


# what is weakness?

그러나 compound component 패턴은 코드와 구조의 복잡도를 올리는 문제가 있습니다.

즉 풀어야할 문제 , 요구되는 기능이 복잡하지 않다면

평범한 props 넘겨주기 패턴으로 구현하는 것이 더 좋은 경우가 있습니다.

따라서 compound component 패턴은 복잡한 기능을 구현해야 하는 경우

혹은 관심사를 좀 더 엄격하게 분리 하고 싶은 경우에 유용합니다.


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


# context api와 함께 사용

```tsx
const ModalContext = createContext({});

const Modal = ({ children }) => {
  const [isOpened, setIsOpened] = useState(false)

  const childrenArray = React.Children.toArray(children);
  const header = childrenArray.find(child => child.type === Modal.Header);
  const body = childrenArray.find(child => child.type === Modal.Body);
  const footer = childrenArray.find(child => child.type === Modal.Footer);

  return (
    <ModalContext.Provider value={{ isOpened, setIsOpened }}>
      <div className="modal">
        <div className="modal-header">{header}</div>
        <div className="modal-body">{body}</div>
        <div className="modal-footer">{footer}</div>
      </div>
    </ModalContext.Provider>
  );
};

Modal.Header = ({ children }) => {
  const {setIsOpened} = useContext(ModalContext)
  return <div className="modal-header">{children}<button onClick={() => setIsOpened(false)}>X</button></div>;
};

Modal.Body = ({ children }) => {
  return <div className="modal-body">{children}</div>;
};

Modal.Footer = ({ children }) => {
  const {setIsOpened} = useContext(ModalContext)
  return <div className="modal-footer">{children}<button onClick={() => setIsOpened(false)}>Close</button></div>;
};​
```

compound component 패턴을 context api와 함께 사용하는 것을 통하여 props 전달 없이 상태, 정보를 쉽게 공유할 수 있습니다.



# context api 사용 시 DX 개선

관용적으로 `createContext()`에 전달하는 초기값을 null로 설정하곤 합니다.

이렇게 null로 초기값을 설정해준 경우 `useContext()`를 사용할 때마다

조건문을 이용하여 타입가드를 해주어야만 하는 불편함이 있습니다.

```tsx
const context = useContext(TabsContext)
if(context === null) return <></>
```

이러한 형태로 불필요한 타입가드 코드가 생기게 됩니다.

이를 커스텀훅을 통하여 null 체킹을 수행하는 부분을 분리하는 방법이 있습니다.

```tsx
const useCustomContext = () => {
    const context = useContext(TabsContext)
    if(context === null) {
        throw new Error('useTabs should be used within Tabs')
    }
    return context
}

```

커스텀 훅을 통하여 이미 널체킹이 끝났기 때문에

커스텀훅을 사용하는 컴포넌트에서는 널체킹 없이 바로 상태를 사용할 수 있습니다.



# 마치며

compound component는 장점이 명확한 패턴입니다.

그래서 모든 상황에 적용하고 싶어지기 쉽습니다.

`망치를 들면 모든게 못으로 보인다.` 는 말에 주의할 필요가 있습니다.

상황에 따라 적절한 디자인 패턴을 선택하기 위한 노력과 고민이 필요합니다.

물론 적절한 상황에서 compound component 패턴을 활용하는 것은 너무나도 좋은 선택입니다.


# 레퍼런스

https://fe-developers.kakaoent.com/2022/220731-composition-component/

https://fe-developers.kakaoent.com/2021/211022-react-children-tip/

https://itchallenger.tistory.com/928

https://react.dev/reference/react/Children#alternatives

https://iyu88.github.io//react/2023/03/25/react-compound-component-pattern.html
