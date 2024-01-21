# context api를 하나의 interface로 생각해보세요

context api는 Props Drilling을 피하기 위한 api 입니다.

전역 상태 관리 라이브러리를 공부할 때에도 둘의 차이를 학습하며

이 말을 자연스럽게 많이 접하셨을 것이라 생각합니다.

이러한 context api는 props drilling을 피하기 위한 도구임과 동시에

context 내부에는 무엇이든 넣을 수 있다는 특징이 존재합니다.

<br/>

하지만 그런 자유로움은 반대로 이야기하면 어떤게 베스트 프래티스인지 알기 어렵다는 말이 되기도 합니다.

context api는 어떻게 활용하면 좋을까요?

contextapi는 DI를 위한 Container와 같이 사용할 수 있습니다.

무엇이든 컨텍스트에 넣을 수 있다는 것을 기억해보세요

<br/>

```tsx
interface PaymentStrategy {
  getCurrencySign(): string;
  roundUp(amount: number): number;
}
```

앞서 context api를 하나의 인터페이스로 생각해보자고 언급했습니다.

이것을 기억하면서 예제를 읽어봅시다.

이 PaymentStrategy 인터페이스는 매우 간단한 인터페이스입니다.

이 인터페이스의 구현 조건만 충족하면 세부구현에는 신경 쓰지 않는 것을 통하여

개발자들은 코드의 결합도를 낮출 수 있습니다.

<br/>

타입스크립트의 implements 키워드를 이용하여 해당 인터페이스의 조건을 구현하는 클래스

AustralianStrategy 를 만들어보겠습니다.

```tsx
class AustralianStrategy implements PaymentStrategy {
  getCurrencySign() {
    return '$';
  }
  roundUp(amount: number): number {
    return Math.floor(amount + 1);
  }
}
```

이제 이 충실하게 구현된 클래스의 인스턴스를 이용하여

컴포넌트에게 인스턴스를 건네주는 코드를 작성해보겠습니다.

```tsx
const Payment = ({
  amount,
  strategy,
}: {
  amount: number;
  strategy: PaymentStrategy;
}) => {
  return <button>{strategy.roundUp(amount)}</button>;
};

const strategy = new AustralianStrategy();

const CheckoutPage = () => {
  return <Payment amount={amount} strategy={strategy} />;
};
```

눈치챘겠지만 이 코드는 프론트엔드 개발자 입장에서 보았을때 굉장히 어색하고 생소합니다.

게다가 Payment 컴포넌트를 이용하기 위해서는 매번 AustralianStrategy 클래스를 인스턴스화해야하죠

# Context API에 사용한다면 어떨까요?

```tsx
export interface PaymentStrategyContextType {
  strategy: PaymentStrategy;
}

export default createContext<PaymentStrategyContextType | null>(null);
```

먼저 컨텍스트를 위한 인터페이스를 정의하고

createContext의 제네릭으로 우리가 컨텍스트에 사용할 인터페이스를 제공해주겠습니다.

이후 컨텍스트 api를 통하여 런타임에 구현체를 제공해줍니다.

```tsx
import PaymentStrategyContext from './PaymentStrategyContext';

const App = () => {
  const strategy = new AustralianStrategy();
  return (
    <PaymentStrategyContext.Provider value={{ strategy }}>
      <Route />
    </PaymentStrategyContext.Provider>
  );
};
```

이제 우리의 컴포넌트는 useContext() 키워드를 이용하여 구현체를 가져오고 사용할 수 있습니다.

```tsx
const Payment = ({ amount }: { amount: number }) => {
  const { strategy } = useContext(PaymentStrategyContext);
  return <button>{strategy.roundUp(amount)}</button>;
};
```

이것이 바로 context api를 통해 di를 수행하는 방법입니다.

# 이렇게 하면 뭐가 좋은가요?

1. 유연하다.

더이상 컴포넌트들은 클래스의 구체적인 구현 세부사항을 알 필요가 없습니다.

따라서 인터페이스의 요구사항만 충족되면 얼마든지 구현체를 바꿔끼울 수 있습니다.

이렇게 결합도가 느슨해진 코드들은 변경되어도 서로에게 영향을 거의 주지 않기 때문에

리팩토링, 유지보수가 쉬워집니다.

2. 캡슐화

더이상 컴포넌트들은 클래스의 구현을 알지 못합니다.

그저 인터페이스에 의존할 뿐입니다.

컴포넌트들은 클래스가 공개하기로 결정한 public api를 통해서만 소통하며

내부 구현은 클래스, context api 내부로 숨겨지게 됩니다.

3. 유지보수

위에서도 언급했지만 결합도가 느슨해졌기 때문에

변경에 자유로워지고 그만큼 유지보수도 쉬워지게 됩니다.

4. 테스트

인터페이스(PUBLIC API)의 관점에서만 테스트를 작성할 수 있습니다.

또한 context api를 통해 가짜 객체를 주입하는 형태로 테스팅을 할수도 있어집니다.

# 마치며

개인적으론 context api의 리렌더링 이슈, Provider hell 문제는 마음에 들지 않지만

유지보수적인 관점에서 di를 익히게되는 계기가 되실거라고 생각합니다.
