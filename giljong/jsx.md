# jsx란 무엇일까?

jsx가 포함된 코드를 아무런 처리 없이 실행하면 에러가 발생합니다.

당연하다면 당연한 것인데 왜냐하면 jsx는 자바스크립트의 표준 문법이 아니기 때문입니다.

따라서 jsx를 실행하기 위해서는 별도의 트랜스파일러를 거쳐야만 합니다.


# jsx는 왜 쓰나요?

jsx는 다양한 트랜스파일러에서 다양한 속성을 가진 트리 구조를 토큰화하여 ECMAScript로 변환하는 것을 초점에 둡니다.

리액트에 한정해서 사용하는 것이 아니라 다른 구문으로도 확장 될 수 있게끔 고려되어있고

개발자에게 문법적인 편의성을 제공합니다.


# jsx의 구성

jsx는 기본적으로 JSXElement , JSXAttributes , JSXChildren , JSXStrings 네가지 컴포넌트를 기반으로 구성되어있습니다.



## JSXElement

JSX를 구성하는 가장 기본적인 요소로서 HTML의 Element와 비슷한 역할을 합니다.

조금 별개의 이야기로 실제로 타입스크립트를 이용하여 코딩을 할 때 꽤나 자주 사용하게되는 타입이기도 하는데요

보통 컴포넌트들의 반환타입이 바로 이 JSXElement이기 때문입니다.

### JSXOpeningElement

일반적으로 볼 수 있는 요소인데요 만약 JSXOpeningElement로 시작했다면 같은 단계에서

JSXClosingElement가 선언되어 있어야 올바른 문법으로 간주됩니다.

### JSXClosingElement

JSXOpeningElement가 끝났다는 것을 알리는 요소입니다. 반드시 오프닝엘리먼트와 함꼐 사용되어야합니다.

### JSXSelfClosingElement

요소가 시작되고 스스로 종료되는 형태를 말합니다.

<script/>와 같은 형태를 표현한다고 볼 수 있고 이는 잠재적으로 자식을 가족있지 않다는 것을 암시하기도 합니다.

### JSXFragment

아무런 요소가 없는 형태로 JSXSelfClosingElement 형태를 띨 수 없습니다.

<></> 이런식으로 작성되어있는 것을 의미합니다.

조금 생소하지만 막상 읽어보면 당연하게 쓰고있는 녀석들입니다.

