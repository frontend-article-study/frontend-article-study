#📌 useImperativeHandle

####목차
1. [useImperativeHandle을 공부하기전, 알아야하는 점](#useimperativehandle을-공부하기전-알아야하는-점)
2. [참조](#reference-참조)
3. [개요](#개요)
4. [매개변수](#매개변수)
4. [사용법](#사용법)
---

- imperative : 반드시해야하는, 긴요한, 명령조의
useImperativeHandle은 ref로 노출되는 핸들을 사용자가 직접 정의할 수 있게 해주는 React훅입니다. 즉, hook을 사용하기 위해선 부모측에서 forwardRef를 통해 ref를 사용하고 있어야합니다.
</br>


### useImperativeHandle을 공부하기전, 알아야하는 점
- 리액트에서는 일반적으로 사용할 수 없는 Prop이 있습니다. 그 중에서도 ref prop도 HTML Element 접근이라는 특수한 용도로 사용되기 때문에 일반적인 Prop으로 사용할 수 없습니다.
- 예를 들어, 상위 컴포넌트에서 `useRef` Hook을 통해 ref를 생성한 후, 하위 컴포넌트에 넘겨주기 위해서는 `forwardRef()` 함수로 감싸주면 하위 컴포넌트는 2번째 매개변수를 갖게 됩니다. 이를 통해 ref를 prop으로 전달할 수 있습니다.
</br>
##### ref를 prop으로 전달하는 예시
```JSX
import { useRef, forwardRef } from "react";

// 두번째로 전달되는 ref 매개변수
const Input = forwardRef((props, ref) => {
    return <input type="text" ref={ref} />;
});

const Field = () => {
    const inputRef = useRef(null);
    const focusHandler = () => {
    	inputRef.current.foucus();
    };
    
    return( 
    	<> 
            <Input ref={inputRef} /> 
            <button onClick={focusHandler}>focus</button> 
        </> 
    ); 
};
```
</br>

### 개요
`useImperativeHandle(ref, createHandle, dependencies?)`

### Reference 참조
- 컴포넌트의 최상위 레벨에서 `useImperativeHandle`을 노출할 ref핸들을 사용자가 직접 정의할 수 있습니다.

```JSX
import { forwardRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  useImperativeHandle(ref, () => {
    return {
      // ... your methods ...
      // ... 메서드는 여기에 작성합니다 ...
    };
  }, []);
  // ...
```

#### 매개변수
- `ref` : forwardRef 렌더함수에서 두 번째 인자로 받은 ref입니다.
- `createHandle` : 인자가 없고 노출하려는 ref 핸들을 반환하는 함수입니다. 해당 ref 핸들은 어떠한 유형이든 될 수 있습니다. 일반적으로 노출하려는 메서드가 있는 객체를 반환합니다.
- `dependencies`: createHandle 코드 내에서 참조하는 모든 반응형 값을 나열한 목록입니다. 반응형 값은 props, state 및 컴포넌트 내에서 직접 선언한 모든 변수와 함수를 포함합니다. React에 대한 린터를 구성한 경우 모든 반응형 값이 올바르게 의존성으로 지정되었는지 확인합니다. 의존성 목록은 항상 일정한 수의 항목을 가지고 [dep1, dep2, dep3]와 같이 인라인으로 작성되어야 합니다. React는 각 의존성을 `Object.is` 비교를 사용하여 이전 값과 비교합니다. 리렌더링으로 인해 일부 의존성이 변경되거나 이 인수를 생략한 경우, createHandle 함수가 다시 실행되고 새로 생성된 핸들이 ref에 할당됩니다.

</br>
#### 반환값
- undefined를 반환합니다.

### 사용법

#### 1) 부모 컴포넌트에서 커스터마이징된 메서드 활용
`forwardRef`를 사용해서 ref를 사용하는 부모측에서 커스터마이징된 메서드를 사용할 수 있게 해줍니다.

<br>
만약 아래처럼 정의하면,

##### 부모 컴포넌트
```JSX
function ParentComponent() {
  // inputRef라는 동아줄을 만들어서 자식에게 보낸다
  const inputRef = useRef();
  return (
    <>
          <FancyInput ref={inputRef} />
          <button onClick={() => inputRef.current.reallyFocus()}>포커스</button>
    </>
  )
}
```
<br>

##### 자식 컴포넌트 
```JSX
function FancyInput(props, ref) {
  // 부모가 내려준 동아줄 ref에다가 이것 저것 작업을 한다
  // 부모는 이 로직에 대해 모르고, 위로 끌어올리지 않고도 그냥 ref.current로 접근하여 사용만 하면 된다
  useImperativeHandle(ref, () => ({
    reallyFocus: () => {
      ref.current.focus();
      console.log('Being focused!')
    }
  }));
  // ref는 input을 참조하고 있다. 
  return <input ref={inputRef} />
}
 
// ref를 컴포넌트에 달 때는 forwardRef로 감싼다
FancyInput = forwardRef(FancyInput)

```

- 원래 input 엘리먼트에는 reallyFocus라는 메서드가 없지만, useImperativeHandle 에서 정의하면 사용할 수 있게 됩니다.
- 이렇게 할 때의 장점은 child component에 상태나 로직을 isolate할 수 있다는 점입니다.
- 만약 훅을 사용하지 않았을 경우, 예시의 부모의 컴포넌트에서 onClick 함수 로직을 짜야했을 것입니다.
- 상태나 로직들은 자식 컴포넌트가 갖고 있고, 부모 컴포넌트는 ref.current에서 필요한 프로퍼티를 가져오기만 하면 됩니다.

하지만 기본적으로 리액트는 ref를 사용하는 것을 권장하지 않으므로 필요한 곳에서 선별적으로 사용하는 것이 좋을 것 같습니다. ^^;;

<br>
##### 참고 사이트
<a href="https://dori-coding.tistory.com/entry/React-ref%EB%A5%BC-prop%EC%9C%BC%EB%A1%9C-%EB%84%98%EA%B8%B0%EA%B8%B0-forwardRef">ref를 prop으로 넘기기</a>
<a href="https://developer-alle.tistory.com/372">useImperative의 장점</a>