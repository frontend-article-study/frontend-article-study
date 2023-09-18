# 선언형 프로그래밍으로서의 react

## 선언형 프로그래밍과 React
- 선언형 프로그래밍은 방식을 알려주는 것이 아닌 결과물의 형태 혹은 상태가 **무엇과 같은가**를 설명하는 것입니다. 선언형 프로그래밍을 도입하기위해 리엑트에(UI 라이브러리) 렌더링의 결과물을 나타내는 책임을 위임하게 됩니다. **개발자들은 최종 결과물의 형태만을 리엑트에 알려줍니다. 이 최종 결과물을 얻기위해 DOM에 직접 반영하는 것이 아닌 React에게 책임을 전가합니다.**
```js
function Choco(){
    return (
    <div>
        <h1>Chocolate</h1>
    </div>
    )
}
```

- 결과적으로 React는 선언적으로 컴포넌트를 개발하돌고 안내하고 있습니다. 이에 따라 개발자는 렌더링마다 DOM을 어떻게 조작해야 하는지 신경쓸 필요가 없습니다. **특정 상태에 따른 최종 UI의 결과물을 선언해서 React Element형태로 React에 전달하고 이를 DOM에 반영하는 것은 React의 책임입니다.**


## 명령형 프로그래밍
- 명령형 프로그래밍은 **어떤 방식으로** 결과물에 도달해야 하는지를 알려주는 프로그래밍 방식입니다. 해당 코드에서는 **원하는 동작**을 구체적으로 명시해야 합니다.
```js
function makeChoco(){
  const bodyTag = document.querySelector('body')
  const divTag = document.createElement('div')
  let h1Tag = document.createElement('h1')
  h1Tag.innerText = "Chocolate"
  divTag.append(h1Tag)
  bodyTag.append(divTag)
}
```
## UI 라이브러리로서의 React의 특징
- React를 사용할때는 DOM을 직접 조작하지 않습니다. DOM은 트리 형태이고 각 노드는 객체 형식입니다. 자바스크립트에서는 다음과 같은 코드로 DOM을 직접 제어하고 호출합니다. `const bodyTag = document.querySelector('body')` 하지만 React에서는 React가 자동으로 처리합니다. 이는 **IoC(Inversion of Control)과도 연결됩니다.**  *DOM을 렌더할 책임이 개발자에게서 react로 넘어오게 되었기 때문입니다.* 
- 내부에서 리엑트가 DOM을 컨트롤하기 위해 사용하는 방식이 Fiber 노드라고 할 수 있습니다. 개발자는 React element를 react에게 전달하고 react는 전달받은 React element를 통해 Fiber node를 형성하고 이를 기반으로 DOM트리를 업데이트 합니다.


### 제어의 역전으로 인한 React 라이브러리의 특징 
1. **React가 재조정을 지연할 수 있습니다.**
react가 컴포넌트 호출을 제어하면서 컴포넌트를 다시 렌더링 할때의 우선순위를 자동으로 설정하여 렌더 혈성을 자동으로 최적화 합니다. (ex. batch, suspense)

## Reaference
[UI 런타임으로서의 React](https://overreacted.io/ko/react-as-a-ui-runtime/)
[Declarative React, and Inversion of Control](https://blog.mathpresso.com/declarative-react-and-inversion-of-control-7b95f3fbddf5)


