# 15장 let, const, 키워드와 블록 레벨 스코프

## var 키워드의 문제점

1. 변수 중복 선언 허용
   : 변수 중복 선언시 두번째로 선언하는 문에 초기화문 유무에 따라 다르게 동작함이 인상깊었습니다.
   - 초기화문 0 : var 키워드가 없는 것처럼 동작해 재할당됨.
   - 초기화문 x : 무시됨.

```js
var x = 1;
var y = 1;

var x = 100; // 초기화문 0
var y; //초기화문 x

console.log(x); //100
console.log(y); //1
```

2. 함수 레벨 스코프
3. 변수 호이스팅

## let 키워드

1. 변수 중복선언 금지
2. 블록 레벨 스코프
3. 변수 호이스팅 : 변수 호이스팅이 발생하나 발생하지 않는 것처럼 동작하며 에러를 발생시킨다.
   let 키워드로 선언한 변수는 "선언 단계"와 "초기화 단계"가 분리되어 진행된다. let 키워드는 선언단계 - 일시적 사각지대 TDZ - 초기화 단계 - 할당 단계 로 일어난다.

```js
console.log(foo); // 에러
let foo;

console.log(foo); //undefined
foo = 1;
console.log(foo); //1
```

4. 전역 객체와 let
   : 1. var 키워드로 선언한 전역 변수 2. 전역 함수 3. 선언하지 않는 변수에 값을 할당한 암묵적 전역 => 전역 객체 window의 프로퍼티가 됨.

   하지만 let 키워드로 선언한 전역 변수는 전역 객체의 프로퍼티가 아닙니다. window.foo 와 같이 접근 불가합니다. let 전역 변수는 보이지 않는 개념적인 블록(전역 렉시컬 환경의 선언적 환경 레코드)내에 존재하기 때문입니다.

## const 키워드

1. 선언와 초기화 동시에
2. 재할당 금지
3. 상수는 스네이크 케이스로 표현
4. const 키워드로 선언된 변수에 객체를 할당할 경우 값을 변경할 수 있다. 이는 재할당을 금지할 뿐 "불변"을 의미하는 것은 아니다.

# 16장 프로퍼티 어트리뷰트

내부 슬롯 과 내부 메서드
:자바스크립트 엔진의 구현 알고리즘을 설명하기 위해 ECMAScript 사양에서 사용하는 의사 프로퍼티와 의사 메서드

자바스크립트는 일부 내부 슬롯과 내부 메서드에 한하여 간접적으로 접근할 수 있도록 했다.

모든 객체는 [[prototype]]이라는 내부 슬롯을 갖는다. 이 내부 슬롯은 직접 접근할 수 없지만 .\_ _ proto _ \_ 로 접근가능하다.

프로퍼티 어트리뷰트 : 프로퍼티의 상태를 나타내는 내부 슬롯

- [[value]]: 프로퍼티의 값
- [[writable]]: 값의 갱신 가능 여부
- [[enumberable]]: 열거 가능 여부 -> false 인 경우 for ...in 문 , Object.keys 메서드 등으로 열거 x
- [[configurable]]: 재정의 가능 여부 -> **프러포티 삭제 불가, 프로퍼티 어트리뷰트 값 변경 금지**

## writable 과 configurable 의 차이

writable은 우리가 알고 있듯이 할당연산자 = 로 속성값을 변경하는 일을 합니다. 하지만 configurable 은 속성 Attribute의 재정의가 가능한지 여부를 판단하는 일 맡는 다는 것에 차이가 있습니다. Attribute 값 중 속성값을 변경하는 value 가 있어 configurable 개념이 writable 보다 깊은 개념이지만 **configurable:false 이고 writable : true 일때는 값 변경과 writable 프로퍼티 어트리뷰트 값 변경이 가능한 걸 보아 우선순위는 writable로 높다는 것을 알 수 있습니다.**

주의해야할 것은 **프로퍼티의 프로퍼티 어트리뷰트 값, writable를 false 처리 했다면 이후 수정시 에러가 나지 않고 수정이 되지 않는 다는 것입니다.(configurable:false 처리도 동일) 다만 strict mode일경우 에러를 반환합니다.**
또한 단순히 프로퍼티를 추가했다면(점표기법등) 프로퍼티 어트리뷰트의 값은 전부 true 처리 되지만 프로퍼티 어트리뷰트 재정의시 특정 프로퍼티 어트리뷰트값을 작성하지 않는다면,즉 디스크립터객체의 프로퍼티를 누락시키면 undefined, false (기본값)처리 됩니다.

```js
const person = {
  name: 'Lee',
};
console.log(Object.getOwnPropertyDescriptor(person, 'name'));
// 단순히 프로퍼티를 추가했다면(점표기법등) 프로퍼티 어트리뷰트의 값은 전부 true 처리
//{value "Lee" ,writable:true,enumerable:true,configurable:true}
```

```js
const object1 = { property1: 11 };
Object.defineProperty(object1, 'property2', {
  value: 22, //   property2라는 속성의 value값을 정의할 수 있습니다.
  writable: false, //할당연산자로 속성값 변경 불가
  configurable: false, // 속성 변경 or 삭제 불가
});

object1.property2 = 22; //수정이 되지 않지 않지만  에러가 나지 않습니다.
//strict mode일경우 에러를 반환합니다.

delete object1.property2; //삭제가 되지 않지만 에러가 나지 않는다.

console.log(object1); //property2가 보이지 않는다 왜일까?  프로퍼티 재정의시 enumarable 설정하지 않으면 기본값 false 처리하여 보이지 않게 된다.
console.log(object1.property2); //22
```

## 접근자 프로퍼티

접근자 프로퍼티는 자체적으로 값(프로퍼티 어트리뷰트 [[value]])를 가지지 않으며 다만 데이터 프로퍼티의 값을 읽거나 저장할 때 관여한다.
접근자 프로퍼티의 프로퍼티 어트리뷰트 :[[Get]],[[Set]],[[Enumerable]],[[Configurable]]

### 만약 접근자 프로퍼티 "fullName" 가 프로퍼티 값에 접근한다면

1. [[Get]]내부 메서드가 호출
2. 프로퍼티 키"fullName"가 유효한지 확인 : 심벌또는 문자열인지
3. 프로토타입 체인에서 프로퍼티 검색
4. 검색한 프로퍼티 fullName이 데이터 프로퍼티인지, 접근자 프로퍼티인지 확인
5. 접근자 프로퍼티 fullName의 프로퍼티 어트리뷰트 [[Get]]의 값 즉 getter 함수를 호출 -> 결과 반환
