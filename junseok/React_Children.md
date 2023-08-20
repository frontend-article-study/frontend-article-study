# React Children

## 1. React Children 

리엑트에서 컴포넌트 간의 연결은 prop을 통해 이루어집니다. 이는 단방향 바인딩을 특징으로 합니다. prop을 통한 컴포넌트의 수직적 연결은 두 가지 방법이 있습니다. 

1. 일반적인 prop 전달하기
- 일반적으로 가장 많이 사용하는 방법이라고 생각합니다.
```ts
function Avatar() {
  return (
    <img
      className="avatar"
      src="https://i.imgur.com/1bX5QH6.jpg"
      alt="Lin Lanying"
      width={100}
      height={100}
    />
  );
}

export default function App() {
  return (
    <Avatar />
  );
}

```

2. React Children 
- React Children은 HTML의 빌트인 태그 특징을 그대로 도입하였습니다. 
```html
<select>
    <options></options>
</select>
```

이러한 특징을 바탕으로 직접 작성한 컴포넌트들끼리 중첩할 수 있습니다. jsx 태그 내에 컴포넌트를 중첩하면 리엑트에서 부모 컴포넌트는 `children`이라는 prop을 받게 됩니다. 
```ts
import Avatar from './Avatar.js';

function Card({ children }: {children: React.ReactNode}) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

export default function Profile() {
  return (
    <Card>
      <Avatar
        size={100}
        person={{ 
          name: 'Katsuko Saruhashi',
          imageId: 'YfeOqp2'
        }}
      />
    </Card>
  );
}
```

## 2. 모든 것은 child가 될 수 있습니다. 

recat의 children에는 원시 값, 참조 값과 jsx 값 전달이 모두 가능합니다. 이는 `React.ReactNode`의 타입을 확인해 보면 명확하게 알 수 있습니다.
```ts
    type ReactNode =
        | ReactElement
        | string
        | number
        | ReactFragment
        | ReactPortal
        | boolean
        | null
        | undefined
        | DO_NOT_USE_OR_YOU_WILL_BE_FIRED_EXPERIMENTAL_REACT_NODES[keyof DO_NOT_USE_OR_YOU_WILL_BE_FIRED_EXPERIMENTAL_REACT_NODES];

```

이를 바탕으로 children에는 어떤 타입이든 올 수 있다는 것을 확인할 수 있습니다.

```tsx
function Grid({children}: {children:React.ReactNode}){
  return <div>{children}</div>
}


export default function App() {
  const foods = ['라면','밥','김치','김']

  return (
    <div>
      {/* 문자 가능 */}
      <Grid>Hello</Grid>

      {/* 배열 가능 */}
      <Grid>{foods.map(item => <div key={item}>{item}</div>)}</Grid>

      {/* jsx 값 가능, 및 혼합해서 전달 가능 */}
      <Grid><div style={{width:'100px', height:"100px", color:"white", backgroundColor: "black"}}>It's a jsx</div>
          it's a string
      </Grid>
    </div>
  );
}
```

### 2-1. children에 함수 전달하기 
이러한 방법 뿐만 아니라 children에 특정 값을 반환하는 함수 자체를 전달해줄 수도 있습니다. 이는 특정한 디자인 패턴으로 정리해서 각 컴포넌트의 역할을 명확하게 분리할 수 있습니다. Function as Child 패턴으로 불리기도 합니다. 

**react children은 자식에 어떤 것이 들어올지 모른다**고 가정을 합니다. `FunctionComponent`는 데이터 로직만을 갖습니다. 해당 데이터 로직을 자식 함수에 주입합니다.

children에서 필요한 데이터를 전달 받아서 같은 데이터를 활용하는 다른 jsx component에 동일한 로직을 적용할 수 있습니다. 

고양이 랜덤 이미지를 가지고 오는 API를 예시로 활용해 보았습니다. 



```tsx
import {useEffect, useState} from 'react'

// API 타입 지정
type Api = {
  id: string;
  url: string;
  width: number;
  height: number;
}

// FunctionComponent prop 설정 
type Prop  = {
  url : string
  children: (image:string)=> JSX.Element
}

// API 불러오는 함수 설정
async function fetchUrl<T>(url:string){
  const json = await fetch(url)
  const data: Promise<T[]> = json.json()
  return data
}

// 데이터를 받아오는 로직 작성
function FunctionComponent({children, url}: Prop){
  const [image, setImage] = useState('')

  useEffect( () => {
    (async() => {
      const data = await fetchUrl<Api>(url)
      setImage(data[0].url)
    })()
  },[url])

  return children(image)
}

export default function App() {
  return (
      <FunctionComponent url={'https://api.thecatapi.com/v1/images/search'}>
        {/* 자유롭게 필요한 jsx Component 로직을 작성할 수 있습니다. */}
        {(image) => (<div style={{display: "flex", flexDirection: "column"}}>
          <img src={image} width={'500px'} height={'500px'} />
          <span>Image URL은 다음과 같습니다. ({image})</span>
        </div>)}
      </FunctionComponent>
  );
}

```
이 외에도 Compounds Components와 같은 패턴에서 children을 적극적으로 활용하고 있습니다. Compounds Components를 더 자세히 알고 싶다면 [길종님의 정리](https://github.com/XionWCFM/react-article-study/blob/main/giljong/CompoundComponent.md) 혹은 [해당 블로그](https://www.howdy-mj.me/design/headless-components)를 참조해주세요


## 3. Children API 활용하기 
react의 Children API를 통해 각 컴포넌트에 들어오는 children을 조작할 수 있습니다. **하지만 공식문서에서 추천하지 않는 기능입니다.** 공식 문서에는 다음과 같이 설명하고 있습니다. 
> **Children을 사용해야 하는 경우는 드물고, 코드가 취약해질 수 있습니다. 다른 일반적인 대안을 참조하세요.**
> - 명확한 이유는 알려주지 않고 있습니다. 🥲 이를 알기 위해서는 직접 라이브러리를 까보는 수 밖에 없는 것 같습니다. 

그래도 알면 나쁘지 않다고 생각하기에 기초적인 몇몇 method만을 소개합니다. 

### Children.toArray(children)
- children 데이터 구조에서 배열을 생성할 수 있습니다. 

### Children.forEach(children, fn, thisArg?) &&  Children.map(children, fn, thisArg?)
- 자바스크립트의 `Array.prototype.forEach()`, `Array.prototype.map()`과 사용법이 유사합니다.  

### 예시 첫 번째 children 요소 랜더링에서 배제하기
```ts
type PropChildren = {
  children : React.ReactNode
}

// Children.toArray(children)
function IgnoreFirstChild({children}: PropChildren): any{
  const childrenList = Children.toArray(children)
  return childrenList.map((item, index) => {
    if(index === 0) return null
    else return <div>{item}</div>
  } )
}

// Children.toArray(children)
function IgnoreFirstChildCase2({children}: PropChildren): any {
  return Children.map(children, (child, index)=> {
    if(index === 0) return null
    else return <div>{child}</div>
  })
}

export default function App() {
  return (
    <>
      <IgnoreFirstChild>
        <div >First</div> {/* 렌더링 제외*/}
        <div >Second</div>
      </IgnoreFirstChild>
      <IgnoreFirstChildCase2>
        <div>First</div> {/* 렌더링 제외*/}
        <div>Second</div>
      </IgnoreFirstChildCase2>
    </>
  );
}

```


## Reference
- [공식 문서 prop](https://react-ko.dev/learn/passing-props-to-a-component)
- [공식 문서 Children](https://react-ko.dev/reference/react/Children)
- [mxstbr.blog](https://mxstbr.blog/2017/02/react-children-deepdive/)
- [카카오 블로그](https://fe-developers.kakaoent.com/2021/211022-react-children-tip/)
- [우아한 테크코스](https://www.youtube.com/watch?v=aAs36UeLnTg)