### Context API
리액트 버전 16부터  리액트에서 제공하는 React 컴포넌트 트리 안에서 전역 상태를 전달 및 공유할 수 있는 내장 API 입니다. 주로 전역 상태를 공유할 때 사용하지만 반드시 전역적으로 필요는 없습니다. 또  context API는 단순히 리액트 컴포넌트간에 값들을 공유하는 일만 하며, 주로  useState 와 같이 사용해 useState가 *상태관리를, context API 가 Props가 아닌 방식으로 값을 공유하는 역할을 합니다. 따라서 context API는 "상태 관리"도구가 아님을 유의해야 합니다. context API는 파이프 같은 역할로 props Drilling 을 막을 수 있지만 남용하면 컴포넌트를 재사용하기 어려울 수 있어 주의하여 사용해야 합니다. Context API는 상태가 Context Tree 내부에 포함된 다른 컴포넌트들과 공유되는 방식입니다. 그래서 새로운 상태 값을 생성 할 때 해당 Context 내부에 포함된  컴포넌트들이 상태 값의 일부에만 관심이 있더라도 re-render 되므로 성능 문제가 발생할 수 있습니다. (왜 이런게 만든거야잉)

즉, Context API 를 활용한 전역 상태관리는 <span style="color:blue"> 단순하게 Props Drilling을 피하고 싶고 낮은 규모와  자주 업데이트할 필요가 없는 경우, 단순히 값을 공유</span>할 때 사용하는 것이 좋습니다.

정확히는 다음과 같은 데이터일때 사용하기 좋다고 하네요

- 테마 데이터 (다크 모드 혹은 라이트 모드)
- 사용자 데이터 (현재 인증된 사용자)
- 로케일 데이터 (언어 혹은 지역)

  ![img](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcL5Mb3%2FbtstwGs25ME%2Fi7LhUzmo6kJAfQ5twdWhHk%2Fimg.png)
 ![img](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbUOVbT%2Fbtstw3Io6dX%2FsgtOCaIvw5sBA5fEknBplk%2Fimg.png)
### Context API 를 사용하는 법.
createContext 메서드를 사용해 context를 생성한다.
생성된 context를 가지고 context provider로 컴포넌트 트리를 감싼다.
value prop을 사용해 context provider에 원하는 값을 입력한다.
useContext를 통해 필요한 컴포넌트에서 그 값을 조회할 수 있다.
  ```jsx
//userContext.js
import { createContext, useContext, useState } from "react";

const initial = {
  id: "leh",
  name: "연습",
};

const UserContext = createContext(); //1. createContext 메서드를 사용해 context를 생성
// createContext()매게변수로 넣어주는 것과13 줄 하단에 value로 넘어주는 것차이는 ?

export function UserProvider({ children }) {
  const [state, setState] = useState(initial); // useState 로 감싼 이유 -> 상태관리를 위해/Provider 상위 컴포넌트가 리렌더링되더라도 initialState는 바뀌지 않도록, 참조 동일성을 유지하도록 하기 위해 사용

  //2. context를 가지고 context provider로 컴포넌트 트리를 감싼다.
  //3. value prop을 사용해 context provider에 원하는 값을 입력
  return <UserContext.Provider value={state}>{children}</UserContext.Provider>;
}
export function useUserContext() {
  const context = useContext(UserContext); // context 조회
  if (context === null) {
    throw new Error("useUserContext must be used within UserProvider");
  }
  return context;
}
```
```jsx
//Playground.js -> app.js와 연결됨
import { useState } from "react";
import { UserProvider, useUserContext } from "./userContext";

function UserComponent() {
  console.log("UserComponent render");
  const userContext = useUserContext();
  //context 조회 가장 가까이에 있는 provider value 값으로 보여준다

  return (
    <div style={{ border: "1px solid red" }}>
      UserComponent
      <h2>user id : {userContext.id}</h2>
      <h2>user id : {userContext.name}</h2>
    </div>
  );
}

export default function Playground() {
  const [count, setCount] = useState(0);

  return (
    <>
      <UserProvider>
        <UserComponent />
      </UserProvider>
      <button onClick={() => setCount(count + 1)}>count {count}</button> 
    </>
  );
}

```

### 그렇다면 Redux는?
Redux는 전역 상태관리를 위한 도구이며, Context API와 비교했을 때 보다 더 폭넓은 기능을 제공하고 특정 구성 요소만 re-render 시키거나, 사이드이펙트를 줄일 수 있습니다. 하지만 Context API에 비해 작성해야 하는 코드의 길이도 많고, 또 복잡하다는 단점이 있습니다. 그래서  Context API보다는  여러 위치에서 많은 양의 상태 관리와 대규모의 프로젝트, 그리고 더 강력한 기능이 필요할 때 리덕스를 사용하면 좋습니다.
####  더 강력하고 폭넓은 리덕스 기능으로는 
1. 로컬스토리지에 상태를 영속적으로 저장하고, 시작할 때 다시 불러오는 것에 뛰어나다.

2. 상태를 Server에서 미리 채워서, HTML에 담아 Client로 보내고, 앱을 시작할 때 다시 불러오는 것에 뛰어나다.

3. 사용자의 액선을 직렬화(Serialize)해서, 상태와 함께 자동으로 버그 리포트에 첨부할 수 있고, 개발자들은 이를 통해 에러를 재현할 수 있다.

4. 액션 객체를 네트워크를 통해 보내면, 코드를 크게 바꾸지 않고도 협업 환경을 구현할 수 있다.

5. 코드를 크게 바꾸지 않고도 실행 취소 내역의 관리나 Optimistic Mutations(낙관적인 변경)을 구현할 수 있다.

6. 개발할 때, 상태 내역 사이를 오가고 액션 내역에서 현재 상태를 다시 계산하는 일을 TDD 스타일로 할 수 있다.

7. 개발자 도구가 완전한 조사, 제어를 할 수 있게 함으로써 개발자들이 자신의 앱을 위한 도구를 직접 만들 수 있게 해준다.

8. 비즈니스 로직을 대부분 재사용하면서 UI를 변경할 수 있게 한다.

### 그럼 리덕스보다 가벼운 라이브러리 Recoil과 비교한다면?
Recoil은 상태관리 라이브러리로, 리덕스보다는 가볍지만 리덕스 store의 상태들처럼 구독(subscribe)할 수 있고 Atom의 상태가 변경되면 구독하고 있는 컴포넌트들이 다시 렌더링되면서 변경된 Atom의 상태를 공유합니다.  또한 Recoil은 변경된 Atom의 상태를 공유하고 있는 컴포넌트만 리렌더링 되므로, Context API처럼 쓸모없는 리렌더링이 계속 일어나지 않습니다.  또다른 장점으로는 동시성 모드(Concurrent Mode) 와같은  React와 개발 방향성이 같다는 점.  비동기 처리를 간단하게 할 수 있고  내부적으로 캐싱이 된다는 점이 있습니다. 반면 단점으로는 캐싱관련문제, 안전성에 대한 우려, 리덕스의 devtools 만큼 훌륭한 디버깅툴의 부재, 아직도 많은 실험적인 API들이 있습니다.
 Recoil은  contextApi에 대한 보안제이자,  서버 데이터를 신뢰할 수 있게 하는 상태관리로, 정보가 서버에서 들어오고 거의 변화하지 않는 값들을 관리해줄 때 Recoil이 적당하고 생각합니다.
 
### 출처 
https://ko.legacy.reactjs.org/docs/context.html

https://bum-developer.tistory.com/entry/React-Context-API-%EC%96%B8%EC%A0%9C-%EC%82%AC%EC%9A%A9%ED%95%B4%EC%95%BC%ED%95%A0%EA%B9%8C
bum-developer.tistory.com
https://velog.io/@dldngus5/TILReact-%EC%83%81%ED%83%9C%EA%B4%80%EB%A6%AC-%EA%B3%A0%EB%AF%BC-Context-Recoil

https://velog.io/@ckstn0777/Context-API%EC%9D%98-%EC%B5%9C%EB%8C%80-%EB%8B%A8%EC%A0%90%EC%9D%80-%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C

## 참고 예제
