# what is closure trap

클로저 트랩이란 무엇일까요?

예제를 통해 알아보겠습니다.

```tsx
import { useEffect, useState } from 'react';

export default function App() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    setInterval(() => {
      setCount(count + 1);
    }, 1500);
  }, []);

  useEffect(() => {
    setInterval(() => {
      console.log(count);
    }, 1500);
  }, []);

  return <div>Hello world</div>;
}
```

여기에 useEffect와 타이머함수를 이용하는 간단한 예제를 준비했습니다.

한 useEffect는 setCount 함수를 호출하며 카운트의 값을 1 증가시키는 일을 하고

한 useEffect는 count를 콘솔에 찍는 일을 수행합니다.


이 예제의 실행결과를 예측할 수 있나요?

콘솔창에 나타나는 결과는 다음과 같습니다.

`0 , 0 , 0 , 0 , 0`

<br/>

이것이 오늘 이야기할 클로저 트랩입니다.

아마 타이머함수를 useEffect와 함께 써본 경험이 있으신 분들이라면

한번쯤은 겪어보셨을 일이라고 생각합니다.

왜 이런 일이 발생하는 것일까요? 


# react runtime에서의 component

재조정자 , fiber architecture 등을 학습하면서 익숙해지셨을 것이라 생각합니다만

실제로 컴포넌트가 반환하는 것은 HTML이 아니라 react element

그리고 나아가 생각하면 fiber node라는 것을 알고 있습니다.

그리고 이 fiber node에는 memoizedState라는 속성이 존재하며 이는 linked list 입니다.

또한 컴포넌트의 각 훅들은 이 memoizedState라는 링크드 리스트의 Node에 해당하며

해당 Node에서 자신의 값에 접근합니다.

코드로 간단히 표현하면 이런 형태인 것입니다.


```tsx

const exampleFiber = {
    index : 0,
    lanes : 0,
    flags : 1540,
    memoizedState : node1 -> node2 -> node3
}

const Component = () => {
    const [node1,setNode1] = useState(0)

    useEffect(() => {
        const node2 = '이 useEffect 자체가 node2라고 생각하세요'

    },[])

    useEffect(() => {
        const node3 = '그럼 이 useEffect는 node3이겠죠?'
    },[])
}

```

그리고 컴포넌트에 존재하는 각각의 훅들은 자신의 memoizedState에 접근하여 로직을 완료합니다.


# 차근차근 생각해보기

우선 두 useEffect 모두 의존성 배열이 []이기 때문에

useEffect는 한번만 실행됩니다.

조금 더 자세히 이야기해볼까요?

## 훅의 dependency array

꽤 기본적으로 배우는 내용이기 때문에 쉽게 잊어버리고 있었거나

뭐 이런 걸 모르는 사람도 있나? 라고 생각할 수도 있겠습니다.

훅의 의존성배열은 인자를 전달해주지 않은 경우 모든 경우 리렌더링되며

의존성배열로 빈배열을 전달하며 한번만 실행되고

의존성배열의 요소를 넣어 전달한 경우에는 그 값이 변했을 때에만 실행됩니다.

<br/>

그리고 이는 소스코드의 구현부에도 드러나는 사항입니다.

다음은 useEffect의 구현부 코드입니다.

정말 그런지 한번 확인해볼까요?

<br/>

그 이전에 선수지식으로 알고가야할 것이 하나있습니다.

훅에는 마운트와 업데이트 두 단계가 존재합니다.

mountEffectImpl은 훅이 처음 생성될 때 실행되고

updateEffectImpl은 훅이 업데이트될때마다 실행되는것이지요

```tsx
function mountEffectImpl(fiberFlags, hookFlags, create, deps) {
    var hook = mountWorkInProgressHook();
    var nextDeps = deps === undefined ? null : deps;
    currentlyRenderingFiber$1.flags |= fiberFlags;
    hook.memoizedState = pushEffect(HasEffect | hookFlags, create, undefined, nextDeps);
}

function updateEffectImpl(fiberFlags, hookFlags, create, deps) {
    var hook = updateWorkInProgressHook()
    var nextDeps = deps === undefined ? null : deps;
    var destroy = undefined

    if(currentHook !== null) {
        var prevEffect = currentHook.memoizedState;
        destroy = prevEffect.destroy

        if(nextDeps !== null) {
            var prevDeps = prevEffect.deps;

            if(areHookInputsEqual(nextDeps, prevDeps)) {
                pushEffect(hookFlags, create, destroy, nextDeps)
                return
            }
        }
    }
    currentlyRenderFiber$1.flags |= fiberFlags
    hook.memoizedState = pushEffect(HasEffect | hookFlags, create, destroy, nextDeps)
} 

```

조금 생소하게 느껴질 수 있는 부분을 미리 짚자면

|= 는 자바스크립트에서 Bitwise OR Assignment 입니다.

쉬운 예제를 준비하면 이렇습니다.

```tsx
let a = 5; // 00000000000000000000000000000101
a |= 3; // 00000000000000000000000000000011

console.log(a); // 00000000000000000000000000000111
// Expected output: 7

```

위 예제에서 알 수 있듯이 |=는 

OR 비트 연산 이후 나온 값을 대입합니다.

흐름을 이해하는 데 있어 그리 중요하지는 않은 부분이니 넘어가도록 하겠습니다.

<br/>

두 함수 모두에서 나타나고 있는

```tsx
var nextDeps = deps === undefined ? null : deps;
```

이 변수선언부가 핵심적입니다.

만약 deps에 아무것도 전달되지않았다면,

즉 개발자가 의도적으로 의존성배열을 주는 두번째 인수에 아무것도 전달하지 않았다면

nextDeps는 null로 처리되며 그렇지 않은 경우 의존성배열을 전달합니다.

## 만약 의존성배열이 있다면?

그런 경우에 대한 코드는 `updateEffectImpl` 의 구현부에서 찾아볼 수 있습니다.

다음은 `updateEffectImpl` 에서 발췌한 if문입니다.

```tsx
    if(currentHook !== null) {
        var prevEffect = currentHook.memoizedState;
        destroy = prevEffect.destroy

        if(nextDeps !== null) {
            var prevDeps = prevEffect.deps;

            if(areHookInputsEqual(nextDeps, prevDeps)) {
                pushEffect(hookFlags, create, destroy, nextDeps)
                return
            }
        }
    }
```

우리는 위에서 nextDeps의 값이 undefined일때 null이 된다. 라는 것을 알고있습니다.

즉 currentHook !== null 이라면 의존성배열에 무언가가 전달된 상태라고 볼 수 있는것입니다.

```tsx
var prevDeps = prevEffect.deps

if(areHookInputsEqual(nextDeps, prevDeps))
```

만약 nextDeps가 null이 아니라면 리액트는 이전에 사용한 prevDeps와 nextDeps를 비교합니다.

그 비교는 `areHookInputsEqual` 이라는 query 함수에게 위임하며

`areHookInputsEqual` 함수가 true를 반환하면 pushEffect 함수를 호출합니다.

```tsx
  
function pushEffect(tag, create, destroy, deps) {
  const effect: Effect = {
    tag,
    create,
    destroy,
    deps,
    next: null,
  }
  if (componentUpdateQueue === null) {
    componentUpdateQueue = createFunctionComponentUpdateQueue() // return { lastEffect: null }
    componentUpdateQueue.lastEffect = effect.next = effect
  } else {
    const lastEffect = componentUpdateQueue.lastEffect
    if (lastEffect === null) {
      componentUpdateQueue.lastEffect = effect.next = effect // circular
    } else {
      const firstEffect = lastEffect.next // circular
      lastEffect.next = effect
      effect.next = firstEffect
      componentUpdateQueue.lastEffect = effect
    }
  }
  return effect
}
```

그렇다면 areHookInputsEqual 함수는 어떻게 구성되어있을까요?

중요한 코드만 일부 추출한 areHookInputsEqual의 구현부를 보겠습니다.

```tsx
function areHookInputsEqual(nextDeps, prevDeps) {
    if(prevDeps === null) {
        return false
    }

    if(nextDeps.length !== prevDeps.length) {
        error('The final argument passed to %s changed size')
    }

    for(var i = 0 ; i < prevDeps.length && i < nextDeps.length; i++ ) {
        if(objectIs(nextDeps[i], prevDeps[i])) {
            continue
        }
        return false
    }
    return true
}
```

이제 우리는 이것을 통해서 useEffect가 어떻게 각 요소의 변경을 감지하고 

이펙트 실행 여부를 결정하는지 알았습니다.


# 결론을 향해

이제 알 수 있는 것을 크게 두가지로 정리할 수 있습니다.

1. useEffect와 같은 훅들은 fiber node의 memoizedState의 데이터에 접근합니다.

2. 훅은 deps가 같은지 비교하고 콜백함수의 실행 여부를 결정합니다.


# 해결방법은?

useEffect가 항상 최신상태를 얻기위해서는 count 값이 변할때마다

useEffect의 콜백함수를 실행시켜야합니다.

따라서 의존성배열에 [count]를 전달하면 최신상태를 적절히 얻을 수 있습니다.

또한 컴포넌트가 리렌더링되더라도 타이머함수는 리액트의 바깥 영역에서 사이드이펙트로 동작하고있기 때문에

이를 언마운트 될 때 clear 해주는 로직을 추가하고 나면

우리는 클로저 트랩에서 비로소 자유로워질 수 있습니다.
```tsx
useEffect(() => {
  let timer = setInterval(() => {
    setCount(count + 1);
  }, 1500);
  return () => clearInterval(timer);
}, [count]);

useEffect(() => {
  let timer = setInterval(() => {
    console.log(count);
  }, 1500);
  return () => clearInterval(timer);
}, [count]);
```



# 마치며

분량이 너무 많아지는 것 같아 원글에서 묘사하는 내용들을 조금 덜 자세하게 설명했습니다.

더욱 깊은 이해를 원하신다면 react 의 소스코드를 탐구해보시는 걸 추천드리겠습니다.




# 레퍼런스

https://velog.io/@superlipbalm/the-closure-trap-of-react-hooks

https://www.valentinog.com/blog/react-object-is/

https://goidle.github.io/react/in-depth-react-reconciler_5/
