# react 18

react 18 version에서의 가장 큰 변화는 "동시성 모델"이라고 생각합니다.

react 16 version의 stack -> fiber 아키텍처 / suspense 도입

역시도 동시성 모델의 완성을 위한 초석이라고 생각해요

`deepdive magic code` 에서는 동시성 모델을 한마디로

'대규모 화면 전환에서도 높은 응답성을 유지할 수 있다'라고 표현합니다.

이를 위해 리액트는 업데이트를 바라보는 관점 역시도 새롭게 제시합니다.

# 리액트의 눈으로 업데이트를 바라보기

리액트는 업데이트가 발생했을때 VDOM을 통해 변경점을 확인하고 반영합니다.

그런데 업데이트가 발생할때마다 변경을 하는 것보다는

업데이트를 모아서 일괄 처리(batch) 하는게 더 효율적이지 않을까요?

많은 상황에서는 옳은 생각이지만 그렇지 않을 때도 있습니다.

## 업데이트는 모두 같은 종류의 업데이트일까?

업데이트는 먼저 이벤트의 시작점을 통해 구분 할 수 있습니다.


1. 비동기 스코프로 인한 업데이트

2. 사용자 행동에 따른 업데이트

그런데 업데이트는 곧 렌더링으로 이어집니다.

렌더링의 관점에서 바라보면 업데이트를 더욱 세분화하여 분류할 수 있습니다.

그것이 전환(transition) 입니다.

## transition

UI 전환 렌더링을 발생시키는 업데이트를 전환 업데이트라고 합니다.

이 UI 전환은 리액트 입장에서 스스로 판단할 수 없습니다.

따라서 개발자가 명시적으로 리액트에게 알려줍니다.

```tsx
const SearchApp = () => {
  const [text, setText] = useState('');
  const [transitionText, setTransitionText] = useState('');

  return (
    <>
      <input onChange={(...) => {
        setText(...);
        React.startTransition(() => setTransitionText(...));
      }} />
      <AsyncAutoComplete target={transitionText} />
    </>
  )
}
```
React의 startTransition api를 통해 

transitionText의 상태 업데이트의 시작점을 전환 이벤트로 설정하는 코드입니다.

## 업데이트 간 중요도

업데이트를 여러가지로 분류할 수 있다면

각 업데이트의 중요도를 매기는 것도 가능할 것입니다.

업데이트간의 중요도를 매겨야하는 이유는 간단합니다.

컴퓨터의 자원은 한정되어 있기 때문입니다.

<br/>


### 그럼 어떤걸 먼저 업데이트 해야할까요?

리액트는 사용자 경험과 연관지어 중요도를 책정합니다.

사용자는 자신의 물리적 행위에 대해 즉각적인 반응을 기대하며

그렇지 않다면 무언가 잘못되어가고있다고 생각합니다.

따라서 우리는 사용자의 물리적 행동에 빠른 반응을 해야합니다.

리액트는 이를 위해 useTransition 과 같은 훅을 제공합니다.

# 리액트식 해결 방법

리액트는 사용자 액션에 대한 응답을 최우선적으로 처리하고 그 다음 transition 에 대한 응답을 처리하기로 합니다.

여담으로 이러한 업데이트의 우선순위를 결정하고 무엇을 먼저 실행할지 고르기 위해서는

fiber 아키텍처가 필요합니다.

## 동시성 렌더링

일반적으로 개발자가 업데이트를 생성한 경우 해당 업데이트는 동기로 렌더링이 진행됩니다.

즉 렌더링이 끝날때까지 메인 스레드를 점유한 상태로 작업을 진행한다는 것입니다.

반면 동시성 렌더링은 렌더링 과정을 비동기로 / 점진적으로 진행합니다.

그래서 동시성 렌더링 작업은 각 태스크가 잘게 쪼개져 있는 것을 볼 수 있습니다.

이것을 Time Slicing 이라고 부릅니다.

<br/>

이렇게 잘게 쪼개져서 메인스레드를 계속 점유하지 않는 렌더링 방식을 취하면

전환 렌더링 중에 우선순위가 더 높은 사용자 상호작용 이벤트가 발생했을 때

해당 업데이트를 먼저 렌더링할 수 있습니다.

리액트에서는 

1. concurrent render : 전환업데이트의 비동기 점진적 렌더링

2. sync render : 동기 렌더링

으로 분류하여 부릅니다.


# transition lane

transition lane 모델은 concurrency 모드의 구현체라고 표현할 수 있습니다.

즉 concurrency 모드를 위해 만들어진 기법이라고 표현할 수 있으며

useTransition , startTransition, useDeferredValue 를 통해

개발자들은 concurrency features를 사용할 수 있습니다.

<br/>

react lane 에서는 차선이 작을수록 우선순위가 높아지며

현재 총 31개의 레인이 Bit으로 이루어져 잇습니다.

```tsx
export const TotalLanes = 31;

export const NoLanes: Lanes = /*                        */ 0b0000000000000000000000000000000;
export const NoLane: Lane = /*                          */ 0b0000000000000000000000000000000;

export const SyncHydrationLane: Lane = /*               */ 0b0000000000000000000000000000001;
export const SyncLane: Lane = /*                        */ 0b0000000000000000000000000000010;

export const InputContinuousHydrationLane: Lane = /*    */ 0b0000000000000000000000000000100;
export const InputContinuousLane: Lane = /*             */ 0b0000000000000000000000000001000;

export const DefaultHydrationLane: Lane = /*            */ 0b0000000000000000000000000010000;
export const DefaultLane: Lane = /*                     */ 0b0000000000000000000000000100000;

export const SyncUpdateLanes: Lane = /*                */ 0b0000000000000000000000000101010;

const TransitionHydrationLane: Lane = /*                */ 0b0000000000000000000000001000000;
const TransitionLanes: Lanes = /*                       */ 0b0000000011111111111111110000000;
const TransitionLane1: Lane = /*                        */ 0b0000000000000000000000010000000;
const TransitionLane2: Lane = /*                        */ 0b0000000000000000000000100000000;
const TransitionLane3: Lane = /*                        */ 0b0000000000000000000001000000000;
const TransitionLane4: Lane = /*                        */ 0b0000000000000000000010000000000;
const TransitionLane5: Lane = /*                        */ 0b0000000000000000000100000000000;
const TransitionLane6: Lane = /*                        */ 0b0000000000000000001000000000000;
const TransitionLane7: Lane = /*                        */ 0b0000000000000000010000000000000;
const TransitionLane8: Lane = /*                        */ 0b0000000000000000100000000000000;
const TransitionLane9: Lane = /*                        */ 0b0000000000000001000000000000000;
const TransitionLane10: Lane = /*                       */ 0b0000000000000010000000000000000;
const TransitionLane11: Lane = /*                       */ 0b0000000000000100000000000000000;
const TransitionLane12: Lane = /*                       */ 0b0000000000001000000000000000000;
const TransitionLane13: Lane = /*                       */ 0b0000000000010000000000000000000;
const TransitionLane14: Lane = /*                       */ 0b0000000000100000000000000000000;
const TransitionLane15: Lane = /*                       */ 0b0000000001000000000000000000000;
const TransitionLane16: Lane = /*                       */ 0b0000000010000000000000000000000;

const RetryLanes: Lanes = /*                            */ 0b0000111100000000000000000000000;
const RetryLane1: Lane = /*                             */ 0b0000000100000000000000000000000;
const RetryLane2: Lane = /*                             */ 0b0000001000000000000000000000000;
const RetryLane3: Lane = /*                             */ 0b0000010000000000000000000000000;
const RetryLane4: Lane = /*                             */ 0b0000100000000000000000000000000;

export const SomeRetryLane: Lane = RetryLane1;

export const SelectiveHydrationLane: Lane = /*          */ 0b0001000000000000000000000000000;

const NonIdleLanes: Lanes = /*                          */ 0b0001111111111111111111111111111;

export const IdleHydrationLane: Lane = /*               */ 0b0010000000000000000000000000000;
export const IdleLane: Lane = /*                        */ 0b0100000000000000000000000000000;

export const OffscreenLane: Lane = /*                   */ 0b1000000000000000000000000000000;
```

이렇게 bit 으로 이루어진 lane 들을 연산하기 위해 bit 연산을 수행해야하기 때문에

리액트는 리액트 내부에서 주로쓰는 함수들을 구현해두었습니다.

```tsx
export function getHighestPriorityLane(lanes: Lanes): Lane {
  return lanes & -lanes;
}

export function pickArbitraryLane(lanes: Lanes): Lane {
  return getHighestPriorityLane(lanes);
}

function pickArbitraryLaneIndex(lanes: Lanes) {
  return 31 - clz32(lanes);
}

function laneToIndex(lane: Lane) {
  return pickArbitraryLaneIndex(lane);
}
```

또한 fiber 노드들은 lanes , childLanes 프로퍼티를 갖고 있습니다.

lanes는 해당 fiber가 현재 처리해야할 lanes 들을

childLanes는 자식 fiber 들이 어떤 lanes를 가지고 있는지 저장합니다.

또한 이 lanes는 concurrency 모드 뿐만 아니라 sync 모드에서도 사용됩니다.


# 마치며

react18 버전에서의 동시성 렌더링을 구현하기 위해 정말 많은 기법과 고민이 들어간 설계를 볼 수 있었습니다.

그리고 이를 잘 활용하기 위해서 개발자도 알아두어야 할 부분들이 꽤 많네요

예를 들면 기존과 달리 렌더링이 중간에 중단될수도 있기 때문에

렌더링의 멱등성이 보장되어야 한다.라는 부분은 리액트 뿐만 아닌 개발자의 노력도 필요한 부분이었습니다!


# 레퍼런스

https://goidle.github.io/react/in-depth-react18-intro/

https://velog.io/@eunseo9808/React-Lane%EC%9D%84-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90

https://goidle.github.io/react/in-depth-react18-transition_1/

https://goidle.github.io/react/in-depth-react18-transition_2/