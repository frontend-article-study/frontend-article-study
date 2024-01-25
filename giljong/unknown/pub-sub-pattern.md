# pub-sub 패턴이란?

publish / subscribe의 줄임말로 프론트엔드 개발자라면 이름은 몰라도 익숙하게 사용하셨을 패턴입니다.

DOM을 다룰 때의 이벤트 핸들러와 이벤트 개념이 pub-sub과 유사하기 때문입니다.

우리는 쉽게 이런 코드를 작성하곤 합니다.

```tsx

return <button onClick={(event) => {console.log(event)}}>
```

이때에 onClick 이벤트 안의 함수는 이벤트리스너로서 onClick 이벤트가 발생한 경우 이 함수를 실행하겠다는 의미를 내포하게됩니다.

# 근데 이거 어떤 행동이 발생하면 뭔가를 한다는 점에서 Observer 패턴이랑 같지않아요?

Observer 패턴은 Subject와 Observer로 나뉘게되며 각 Observer 들은 Subject들의 행동을 관찰합니다.

이렇게 생각하면 pub-sub과 유사하다고 생각이 들 수 있는데요 이 둘에는 큰 차이점이 존재합니다.

---

       |               |
       |    Subject    |
       |               |


                      Fire Event
                         |

^ \/
| SubScribe

           observer

---

위 그림과 같이 옵저버 패턴에서는 옵저버가 바로 subject를 구독하고 있음을 알 수 있습니다.

즉 옵저버패턴에서는 옵저버는 자신이 "누구"를 구독하는지를 알고있어야 합니다.

누굴 관찰해야할지 모르는 입장에서 관찰을 할 수는 없겠죠?

반면 pub-sub 패턴은 어떨까요?

---

       |               |
       |   Publisher   |
       |               |


       & publish event &


       | Event Channel |

                    Fire Event
       ^
       | Subscribe

       |  Subscriber  |

---

반면 pub-sub 패턴에서 Subscriber는 이벤트를 발생시킨 주체가 누구인지를 모르더라도 아무런 상관이 없습니다.

반대로 publisher 역시도 subscriber의 위치, 존재를 알 필요 없이 중간의 이벤트 채널에 자기가 발생시킨 이벤트를 던져놓기만 하면 되죠

<br/>

## 정리하면

이렇듯 두 패턴은 "뭔가를 관찰한다"는 점에서는 서로 같지만 "무엇을" 관찰하는지가 다릅니다.

또한 그런 차이로 인하여 pub-sub 패턴의 경우에는 observer 패턴에 비해 결합도가 낮습니다.

이러한 개념은 이벤트 핸들러를 보지 않더라도 굉장히 익숙한 개념인데요

사실 프론트엔드 진영의 전역 상태 관리 라이브러리들의 구현 방식을 떠올려보면 더욱 이해가 쉽습니다.

redux의 경우에는 selector라는 개념을 두는 것을 통하여

컴포넌트가 특정한 상태를 "구독"하도록 할 수 있고 "누가 상태를 변경했는지"를 셀렉터로 알 수는 없지만 구독하고 있는 상태가 변경된 경우 그에 맞게 "리렌더링"을 수행합니다.

즉 redux 에서 dispatch는 publish | selector는 subscribe이다. 라고 이해할수도 있는것이지요

# code time

## core type

```tsx
export type EventName<Event> = Event extends string ? Event : never;

export type EventProperty<Prop> =
  Prop extends Record<string, any> ? Prop : never;

export type EventCreator<T = EventName<string>, P = EventProperty<{}>> = P & {
  type: T;
};

export type EventHandler<
  Event extends EventCreator<string, {}> = EventCreator<string, {}>,
> = (event: Event) => void;

export type EventHandlersMap<
  Event extends EventCreator<string, {}> = EventCreator<string, {}>,
> = { [K in Event['type']]?: Array<EventHandler<Event>> };

```

## core class

```tsx
import { EventCreator, EventHandler, EventHandlersMap } from './type';

export class PubSubManager<Event extends EventCreator<string, {}>> {
  private subscribers: {
    [eventType in Event['type']]?: Array<EventHandler<Event>>;
  } = {};

  subscribe(eventType: Event['type'], handler: EventHandler<Event>) {
    if (!this.subscribers[eventType]) {
      this.subscribers[eventType] = [];
    }
    this.subscribers[eventType]?.push(handler);
  }

  unsubscribe(eventType: Event['type'], handler: EventHandler<Event>) {
    const handlers = this.subscribers[eventType];
    if (handlers) {
      this.subscribers[eventType] = handlers.filter((h) => h !== handler);
    }
  }

  publish(event: Event) {
    const handlers =
      this.subscribers?.[event?.type as unknown as Event['type']] || [];

    handlers.forEach((handler) => handler(event));
  }

  initiate(handlerObject: EventHandlersMap<Event>) {
    const entries = Object.entries(handlerObject) as [
      Event['type'],
      EventHandlersMap<Event>[Event['type']],
    ][];
    entries.forEach(([eventType, handlers]) => {
      handlers?.forEach((handler) => {
        this.subscribe(eventType, handler);
      });
    });
  }

  initialize() {
    this.subscribers = {};
  }
}

```

```tsx
type CreateUserEvent = { type: 'CREATE_USER'; userName: string };
type DeleteUserEvent = { type: 'DELETE_USER'; userId: string };

export type MyEvent = CreateUserEvent | DeleteUserEvent;

export const pubSubManager = new PubSubManager<MyEvent>();

export const PubSubProvider = ({ children }: React.PropsWithChildren) => {
  React.useEffect(() => {
    const handlers: EventHandlersMap<MyEvent> = {
      DELETE_USER: [
        (event) => {
          console.log('지우기 유저', event);
        },
      ],
    };
    pubSubManager.initiate(handlers);
  }, []);
  return children;
};

export const Button = () => {
  React.useEffect(() => {
    const handler: EventHandler<MyEvent> = (event) => {
      console.log('handler event', event);
    };
    pubSubManager.subscribe('CREATE_USER', handler);
    return () => pubSubManager.unsubscribe('CREATE_USER', handler);
  }, []);
  return (
    <button
      onClick={() => {
        pubSubManager.publish({ type: 'CREATE_USER', userName: 'moonjae' });
      }}
      {...prop}
    >
      {prop.children}
    </button>
  );
};
```

# 마치며

간단하게 pub-sub 패턴을 구현하는 라이브러리까지 만들어본 뒤 react와 함께 사용해보았습니다.

무언가를 구독하고 , 발행하는 로직의 구현은 크게 어렵지 않지만 개인적으로는 타입을 유연하면서 견고하게 만드는 게 조금 더 어려웠던 것 같습니다.

이렇듯 pub-sub 패턴을 잘 활용하면 패키지간의 의존성을 최소화하면서도 상호작용을 원활하게 할 수 있어진다는 장점이 생기니

참고하시면 좋을 것 같습니다. 감사합니다.

# 레퍼런스

https://jistol.github.io/software%20engineering/2018/04/11/observer-pubsub-pattern/
