## 📌 어떤 타입을 써야할까?
타입스크립트와 리액트로 프로젝트를 개발하다 보면 컴포넌트에서 `children` `prop`으로 받아서 사용하고 이에 대한 인터페이스를 정해줘야하는 경우가 있습니다.

```tsx
export default function Component({children}){
  return (
    <div>
      {children}
    </div>
  )
}
```

이럴때마다 항상 children에 어떤 타입을 명시해야하는 고민이 있었어서 이에 대한 것을 알아보고자 합니다.
<br />

## ✏️ React.ReactElement
```tsx
function Parent({children: React.ReactElement}){
  return children
}

// 에러 발생
function Test(){
  return <Parent>{null}</Parent>
}
```
`React.ReactElement`는 `React` 함수형 컴포넌트의 반환타입입니다. 따라서 해당 타입은 오직 `JSX`요소만 받도록 강제할 수 있기 때문에 위와같이 원시타입을 넣게 된다면 에러를 내뱉게 됩니다.
<br />
따라서 함수형 컴포넌트와 같이 `JSX` 요소만을 `children`으로 받고 싶을 때 사용하는 것이 좋을 것 같습니다.

<br />
## ✏️ JSX.Element
```tsx
function Parent({children: React.ReactElement}){
  return children
}

// 에러 발생
function Test(){
  return <Parent>{null}</Parent>
}
```
`JSX.Element`는 `ReactElement`의 특정 타입이라고 생각하면 됩니다. `JSX.Element`라고 자식을 명시한다면 `ReactElement`와 같이 원시타입은 자식에 할당 될 수 없습니다.

```jsx
const element = <div>Hello, React!</div>;
const element = React.createElement("div", null, "Hello, React!");
```
위와 같은 JSX도 결국 `React.createElement` 함수 호출로 변환되기때문에 `JSX.Element`는 `ReactElement`의  부분집합이라고 생각할 수 있습니다.

<br />

## ✏️ React.FC (!! Not to use)
```typescript
const Example: React.FC = ({ children }) => <div>{children}</div>;
```
18버전 이전에는 FC를 사용해서 함수형 컴포넌트를 선언한다면 `children`을 암시적으로 사용할 수 있었습니다. 하지만 18 버전 이후에는 `children`관련 선언이 모두 제거되었습니다.

- `children`이라는 암묵적이었던 일종의 키워드는 자식 노드가 필요한지 아닌지의 의도를 명확하게 드러낼 수 없다는 단점이 있었습니다. (FC으로 선언해버리는 경우, `children`이 필요하든 안필요하든 암시적으로 선언이 되어버리기 때문에... )
- 이런점은 타입의 관점에서 크리티컬한 문제였습니다. 타입에 자식 노드가 꼭 필요한 컴포넌트에 자식 노드를 넣지 않았을 경우 혹은 그 반대의 경우를 타입으로 잡아내지 못한다는 단점이 있었습니다.

```typescript
type Props = {
  children: React.ReactNode;
};

const Example: React.FC<Props> = ({ children }) => <div>{children}</div>;
```
따라서 앞으로 FC를 사용하게 된다면 위와 같이 명시적으로 선언해주어야합니다.
<br />


## ✏️ React.PropsWithChildren
```tsx
import { PropsWithChildren } from 'react'

type FooProps = {
  name: 'foo'
}

export const Foo = (props: PropsWithChildren<FooProps>) => {
    return props.children
}
```
`PropsWithChildren`을 `type`으로 명시해 제네릭으로 사용한다면, 내부적으로 children이 명시되어 사용할 수 있습니다. 보일러플레이트가 덜 필요하고, 프로퍼티가 암시적으로 입력되는 장점이 있습니다.

```typescript
type PropsWithChildren<P> = P & { children?: ReactNode };
```
하지만 단점으로는 `children`이 옵셔널해서 컴포넌트에서 `children`이 필수인 값이라면 적합하지 않을 수도 있습니다.
또한 `children`이 `ReactNode` 타입으로 지정되므로 `JSX` 요소만이 오게 강제하고 싶을 때는 유용하지 않습니다.
<br />

## ✏️ ReactNode
```tsx
import { ReactNode } from 'react'

type FooProps = {
  name: 'foo'
  // look here 👇
  children: ReactNode
}

export const Foo = (props: FooProps) => {
    return props.children
}
```

가장 광범위하게 사용할 수 있는 React 컴포넌트 타입입니다. 또한 클래스형 리액트 컴포넌트의 반환 타입이기도 합니다.
```typescript
type ReactNode = ReactElement | string | number | ReactFragment | ReactPortal | Boolean | null | undefined;
```
`ReactNode`의 타입명세를 보면 `string`, `number`, `null`, `undefined` 와 같은 원시타입과 `ReactElement`, `ReactFragment`등의 타입을 모두 포함하고 있습니다.
또한 `children`에 대한 타입을 `ReactNode`임을 매번 명시해야합니다. 
<br />

## ✏️ React.ReactChild (‼️deprecated)
```typescript
type ReactChild = ReactElement | string | number;
```
React 18부터 사용을 지양하는 타입입니다. 제 개인적인 의견으로는 타입 지정이 중구난방으로 되어 있어 지양하는 것 같습니다. (ReactNode처럼 다양한 타입을 받는 것도 아니고...)




## 참고 아티클
https://handhand.tistory.com/279#6%EF%B8%8F%E2%83%A3-react.fc-%F0%9F%9A%A8-consider-not-to-use
https://itchallenger.tistory.com/641
https://blog.shiren.dev/2022-04-28/