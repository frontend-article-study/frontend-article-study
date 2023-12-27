# React Design Pettern
### 디자인 패턴?
디자인 패턴은 소프트웨어 개발에서 흔히 발생하는 문제에 대한 반복 가능한 솔루션입니다.

디자인 패턴은 개발 프로세스의 속도를 높일 뿐만 아니라 코드를 더 쉽게 읽고 유지 관리할 수 있게 해줍니다.

Front End에서도 이러한 디자인 패턴을 많이 사용하고 있습니다. 특히 UI 중점인 리엑트에서 활용되는 디자인 패턴은 주로 UI와 Data logic을 분리해서 재활용성과 디자인 변경에 쉽게 대처할 수 있는 방향으로 설계가 되었습니다. 

## React 디자인 패턴 종류

### 1. Compounds Component
컴파운드 컴포넌트는 React에서 그룹으로 함께 작동하는 컴포넌트를 만드는 데 사용되는 패턴으로, 사용자가 여러 관련 컴포넌트의 렌더링을 사용자 정의하고 제어할 수 있습니다.

이를 통해 자식 컴포넌트의 동작과 상태를 캡슐화하는 부모 컴포넌트를 정의할 수 있으며, 동시에 사용자가 자식 컴포넌트의 구성과 모양을 유연하게 결정할 수 있습니다.
[예제](https://codesandbox.io/s/flexible-compound-component-pattern-typescript-j3xqde?from-embed)

### 2. High Order Components
HOC는 하나의 컴포넌트를 입력으로 받아 추가 기능을 가진 새로 생성된 컴포넌트를 출력으로 제공합니다. 이를 통해 컴포넌트를 추가 기능으로 래핑하여 컴포넌트 로직을 재사용할 수 있습니다.

HOC를 사용하면 코드 중복 없이 여러 컴포넌트에 공통 기능을 쉽게 추가할 수 있습니다.

```ts
import React from 'react';
import { useState } from 'react';

function withLogger(WrappedComponent) {
  return function WithLogging(props: {url: string}) {
   const [data, setData] = useState('')

   React.useEffect(()=>{
      (async()=>{
        const json = await fetch(props.url)
        const data = await json.json()
        setData(data[0])
      })()
   }, [])

    return <WrappedComponent data={data}  />;
  };
}

// Usage
const MyComponent = ({data}) => {
  return (<div >{data.id}
            <img src={data.url} alt='image'/>
          </div >);
};

const EnhancedComponent = withLogger(MyComponent);


export default function App(){
  return <EnhancedComponent url={'https://api.thecatapi.com/v1/images/search'}/>
}
```
위의 컴포넌트는 withLogger에서 매개변수로 전달된 컴포넌트(`WrappedComponent`)에 새로운 기능을 추가한 컴포넌트 ( `WithLogger`)를 반환하는 HOC 입니다. 해당 반환 컴포넌트는 prop으로 전달된 url에 맞추어 데이터를 받아옵니다. 


HOC를 사용하기 위해 본래의 컴포넌트인 MyComponent를 WithLogger로 래핑하여 새 컴포넌트인 EnhancedComponent를 만듭니다. 이를 통해 data를 받아오는 기능을 추가하게 됩니다.


### 3. Render Prop(Function as Child)
이 패턴에서 컴포넌트는 함수를 prop으로 받아 해당 함수를 사용하여 콘텐츠를 렌더링합니다.

```ts
import React from 'react';

const MouseTracker = (props) => {
  const [position, setPosition] = React.useState({ x: 0, y: 0 });

  const handleMouseMove = (event) => {
    setPosition({ x: event.clientX, y: event.clientY });
  };

  return <div onMouseMove={handleMouseMove}>{props.renderPosition(position)}</div>;
};

// Usage
const DisplayMousePosition = () => (
  <MouseTracker
    renderPosition={({ x, y }) => (
      <div>
        Mouse position: {x}, {y}
      </div>
    )}
  />
);

```
renderPosition 함수를 prop으로 받아서 활용합니다. 내부 로직은 MouseTracker 컴포넌트에서 이미 작성이 되어 있으며, UI는 디자인에 맞추어서 자유롭게 작성할 수 있습니다.

### 4. Presentational & Container Component
 Container Component는 로직과 상태 관리를 처리하고 Presentational components는 제공된 prop을 기반으로 UI를 렌더링하는 데 중점을 둡니다.
```ts
import React from 'react';
import { useState } from 'react';

function ContainerComponent({url}: {url:string}){
  const [data, setData] = useState('')
  console.log(data)
 React.useEffect(()=>{
    (async()=>{
      const json = await fetch(url)
      const data = await json.json()
      setData(data[0])
    })()
 }, [url])
  return <PrasentationalComponent data = {data}/>
}

const PrasentationalComponent = ({data}) =>{
  return <img src={data.url} />
}


export default function App(){
  return <ContainerComponent url={'https://api.thecatapi.com/v1/images/search'}/>
}

```
Container Component는 상태와 필요한 콜백을 Presentational component에 prop으로 전달합니다. Presentational components는 데이터 가져오기나 비즈니스 로직에 대한 걱정 없이 제공된 prop을 기반으로 UI를 렌더링할 책임이 있습니다.


### 5. Custom Hook
커스텀 훅 또한 위의 4가지 디자인 패턴과 같습니다. state 관리 로직을 분리하여 UI와 데이터를 각각 독립적으로 관리하고 재활용성을 높이는 방향과 같습니다. 
