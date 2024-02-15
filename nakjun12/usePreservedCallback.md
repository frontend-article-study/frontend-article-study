## ⭐ usePreservedCallback

`usePreservedCallback`은 `useCallback`의 단점을 보완하기 위한 커스텀 훅입니다. 이 훅은 함수의 참조 안정성을 유지하면서도 내부적으로 최신 상태나 prop을 반영할 수 있는 방식으로 설계되었습니다.

```javascript
import { useCallback, useEffect, useRef } from "react";

export function usePreservedCallback(callback) {
  const callbackRef = useRef(callback);

  useEffect(() => {
    callbackRef.current = callback; // 최신 콜백으로 업데이트
  }, [callback]);

  return useCallback(() => callbackRef.current(...args), []);
}
```

이 접근 방식의 핵심은 `useRef`를 사용하여 함수 참조를 저장하고, `useEffect`로 최신 함수를 `ref.current`에 지속해서 업데이트하는 것입니다. 이를 통해, 컴포넌트 리렌더링 시에도 함수의 동일한 참조를 유지하면서 최신 값이나 상태를 반영할 수 있습니다.

`usePreservedCallback`은 특히 **여러 상태**나 prop에 의존하면서도 함수의 **참조 안정성이 중요한 경우**에 유용하며, `useCallback`의 단점을 보완하여 보다 효과적인 성능 최적화를 가능하게 합니다.

## 🧪 `usePreservedCallback` 테스트 코드 분석

`usePreservedCallback`은 `useCallback`의 단점을 극복하고자 개발된 커스텀 훅입니다. 이 훅의 주요 장점은 함수의 참조 안정성을 유지하면서도, 최신 상태나 prop을 반영할 수 있다는 것입니다. 이 섹션에서는 `usePreservedCallback`의 실제 작동 방식과 그 효과를 검증하기 위한 테스트 코드를 살펴보겠습니다.

### 테스트 1: 상태 업데이트 후의 참조 안정성

첫 번째 테스트는 상태 업데이트 후에도 `usePreservedCallback`이 반환하는 함수의 참조가 유지하는지 확인합니다. 상태 값이 변하고 컴포넌트가 다시 렌더링되어도, `usePreservedCallback`은 동일한 함수 참조를 유지함으로써 불필요한 리렌더링을 방지합니다.

```javascript
it("preserves the callback reference even after state updates", () => {
  const { result, rerender } = renderHook(() => {
    const [stateValue, setStateValue] = useState(10);
    const testCallback = jest.fn(() => stateValue);
    const preservedCallback = usePreservedCallback(testCallback);

    return { preservedCallback, setStateValue };
  });

  const initialCallback = result.current.preservedCallback;
  act(() => {
    result.current.setStateValue(20);
  });
  const updatedCallback = result.current.preservedCallback;

  expect(updatedCallback).toBe(initialCallback); // 동일 참조 확인
});
```

### 테스트 2: 동일 참조로 최신 상태 값 반영 여부

두 번째 테스트는 참조를 유지하면서 `usePreservedCallback`이 상태 업데이트를 정확히 반영하는지 검증합니다. 상태가 업데이트하면 `usePreservedCallback`은 내부적으로 최신 상태를 반영하는 새로운 함수를 생성하지 않고도 최신 상태 값을 정확히 반환합니다.

```javascript
it("returns updated value from the callback after state change", () => {
  const { result } = renderHook(() => {
    const [stateValue, setStateValue] = useState(10);
    const testCallback = jest.fn(() => stateValue);
    const preservedCallback = usePreservedCallback(testCallback);

    return { preservedCallback, setStateValue };
  });

  const initialValue = result.current.preservedCallback();
  expect(initialValue).toBe(10); // 초기 상태 값 반영 확인

  act(() => {
    result.current.setStateValue(20);
  });

  const updatedValue = result.current.preservedCallback();
  expect(updatedValue).toBe(20); // 최신 상태 값 반영 확인
});
```

### 테스트 3: 인자의 정확한 전달

세 번째 테스트는 `usePreservedCallback`을 통해 전달된 인자가 내부 콜백 함수에 올바르게 전달되는지 확인합니다. 이는 `usePreservedCallback`이 인자를 적절히 처리하여 기대하는 동작을 수행함을 보여줍니다.

```javascript
it("ensures arguments are correctly passed to the wrapped callback function", () => {
  const externalCallback = jest.fn((increment) => increment);
  const { result } = renderHook(() => usePreservedCallback(externalCallback));

  act(() => {
    result.current(10);
  });

  expect(externalCallback).toHaveBeenCalledWith(10); // 인자 전달 확인
});
```

### 표 정리

| 특성      | `useCallback` (의존성 배열 없음) | `useCallback` (의존성 배열 있음)                    | `usePreservedCallback`     |
| --------- | -------------------------------- | --------------------------------------------------- | -------------------------- |
| 참조 유지 | 동일한 참조 유지                 | 의존성 배열의 값이 변경될 때만 참조 갱신            | 동일한 참조 유지           |
| 최신 값   | 최초 생성 시점의 스코프를 기억   | 의존성 배열에 포함된 값이 변경되면 최신 값으로 갱신 | 항상 최신 콜백 함수를 호출 |
