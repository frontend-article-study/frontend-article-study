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

# jsx에는 컴포넌트가 대문자로 시작해야한다는 제약이 없다.

컴포넌트는 대문자로 시작해야한다는 제약은 리액트에서 자체적으로 제공하는 제약인데요

이것은 기본 html 과 리액트 컴포넌트를 원활히 구분하기 위한 제약입니다.

# JSXElementName

JSXElementName은 JSXElement의 요소이름으로 쓸 수 있는 것을 의미합니다.

이름으로 가능한 것은 다음과 같은데요

## JSXIdentifier

JSX 내부에서 사용할 수 있는 식별자를 의미합니다.

자바스크립트의 식별자 규칙과 동일한데요

숫자로 시작하거나 $, \_ 외의 다른 특수문자로는 시작할 수 없습니다.

## JSXNamespacedName

JSXIdentifier:JSXIdentifier의 조합이라고 할 수 있습니다. :를 통해 서로 다른 식별자를 이어주는 것도 하나의 식별자로 취급되는데요

:로 묶을 수 있는 것은 한개 뿐이며 두개 이상은 올바른 식별자로 취급되지 않습니다.

이거 이해하기가 좀 빡셀 수 있는데 예시를 보면 이해가 조금 쉽습니다.

```tsx

<foo:bar></foo:bar> // <<< 이건 가능 왜냐면 :로 묶는게 한개뿐이었음

<foo:bar:baz></foo:bar:baz> // <<< 이건 불가능 왜냐면 :로 묶는게 한개 이상
```

## JSXMemeberExpression

JSXidentifier.JSXIdentifier의 조합입니다.

.을 통해 서로 다른 식별자를 이어주는 것도 하나의 식별자로 취급되는데

그러면 위의 NamespacedName과 무슨 차이가 있냐?? 라는 궁금증이 생깁니다.

:로 묶는것과는 다르게 .을 여러개 이어서 사용할 수도 있다는 차이점이 존재하지만

네임 스페이스를 이어서 사용하는것은 불가능합니다.

```tsx
<foo.bar.baz></foo.bar.baz> // 이건 okay

<foo:bar.baz></foo:bar.baz> // 이건 불가능
```

# JSXAttributes

JSXElement에 부여할 수 있는 속성을 의미합니다.

단순히 속성을 의미하기 때문에 모든 경우에서 필수값이 아니고 존재하지 않아도 에러가 나지 않습니다.

## JSXSpreadAttributes

자바스크립트의 전개 연산자와 동일한 역할을 한다고 볼 수 있습니다.

{...AssignmentExpression} 이 표현식에는 객체뿐만 아니라 자바스크립트에서 표현식으로 취급되는 모든것이 들어갈 수 있습니다.

조건문 표현식, 화살표 함수, 할당식 등등이 가능합니다.

## JSXAttribute

속성을 나타내는 키와 값으로 짝을 이루어서 표현합니다.

키는 JSXAttributeName , 값은 JSXAttributeValue로 불립니다.

### JSXAttributeName

속성의 키 값입니다. 여기서도 마찬가지로 :를 이용하여 키를 나타낼 수 있는데요

```tsx
function valid1() {
  return <foo.bar foo:bar="baz"></foo.bar>;
}
```

### JSXAttributeValue

속성의 키에 할당할 수 있는 값입니다.

밸류는 조금 까다로운 면이 있는데 다음 조건을 만족해야합니다.

#### "큰따옴표로 구성된 문자열"

#### '작은 따옴표로 구성된 문자열'

문자열 내부에 아무 내용이 없어도 괜찮습니다.

#### { AssignmentExpression }

자바스크립트의 표현식을 의미합니다.

즉 자바스크립트에서 변수에 할당할 수 있는 표현식이라면 JSX 속성 값으로도 가능합니다.

#### JSXElement

다른 JSX 요소가 들어갈수도 있습니다.

리액트에서는 보기 힘든 코드지만 이런 형태도 가능합니다.

```tsx
function Child({attribute}) {
    return <div>{attribute}</div>
}

export default function App() {
    return (
        <div>
            <Child attribute=<div>hello</div>>
        </div>
    )
}

```

굉장히 어색하게 느껴지는데 사실 이렇게 써도 됩니다.

이게 어색한 이유는 보통 다른 jsx를 attribute로 넣을때에는 중괄호로 감싸주었기 때문일것입니다.

```tsx
<Card item={<div>hello</div>} />
```

이렇게요 그런데 사실 이렇게 중괄호로 감싸주는 것은

jsx의 규칙이 아니라 prettier의 규칙입니다.

prettier는 태그가 포함된 JSX 구문을 좀 더 쉽게 읽히도록 하기 위해 이런 규칙을 두었다고하네요


### JSXFragment

값으로 별도 속성을 갖지 않는 형태의 JSX 요소가 들어갈 수 있습니다.

즉 비어있는 형태의 <></>를 attribute로 넣는것도 okay 입니다.


## JSXChildren

JSXElement의 자식의 값을 나타냅니다.

JSX는 트리구조를 나타내기 위해 만들어졌기 때문에 JSX로 부모 자식 관계를 나타낼수 있는데요

그 자식관계를 JSXChildren이라고합니다.

이 JSXChildren은 또 내부적으로 여러가지 기본 단위를 가지는데요..

### JSXChild

JSXChildren을 이루는 기본단위입니다.

단어의 차이에서 예상할 수 있듯이 JSXChildren은 JSXChild를 0개 이상 가질 수 있습니다.

### JSXText

{ , } , < , >을 제외한 문자열을 의미합니다. 이것은 다른 jsx 문법과 혼동을 부를 수 있기 때문에 존재하는 제약입니다.

만약에 이 문자를 표현하고 싶다면 문자열로 표현해야합니다.

```tsx
<>{'{ } < >'}</>
```

### JSXElement

값으로 다른 jSX 요소가 들어갈 수 있습니다.

### JSXFragment

값으로 프래그먼트가 들어갈 수 있습니다.

### JSXChildExpression

이거 좀 생소한데요

```tsx
<>{(() => 'foo')()}</>
```


이런식으로 쓸 수 있습니다.

### JSXString

HTML에서 사용 가능한 문자열은 모두 JSXStrings에서도 가능합니다.

이것은 개발자가 HTML의 내용을 손쉽게 JSX로 가져올 수 있도록 의도한 설계입니다.

정의된 문자열은 큰따옴표, 작은따옴표로 구성된 문자열을 의미합니다.

