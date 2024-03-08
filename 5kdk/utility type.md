# 유틸리티 타입 알아보기

## 들어가며

오늘은 우리가 자주 사용하는 프로그래밍 언어 중 하나인 타입스크립트에 대해서 이야기 해볼텐데요, 특히 '유틸리티 타입'이라는 매우 유용한 기능에 대해 알아보려고 합니다.

타입스크립트는 자바스크립트의 슈퍼셋으로, 자바스크립트에 정적 타입 기능을 추가한 언어입니다. 이를 통해 코드의 안정성을 높이고 버그를 줄일 수 있게 되는데요, 이러한 타입스크립트의 장점 중 하나가 바로 '유틸리티 타입'입니다.

그렇다면 '유틸리티 타입'이란 무엇일까요? 유틸리티 타입은 기존 타입을 변환하여 새로운 타입을 생성하는 타입스크립트의 기능입니다.

기존 타입을 더욱 강력하고 유연하게 사용할 수 있도록 도와주죠. 예를 들어 자바스크립트에서는 배열을 다루는데 있어서 `for`, `while`문이면 충분할 것 같지만, 실제로 개발을 진행하다 보면 `forEach`, `map`, `find`, `filter`와 같은 루프 헬퍼 함수가 필요하게 됩니다.

이처럼 유틸리티 타입은 복잡한 타입을 쉽게 만들거나, 타입의 일부를 변경하거나, 제한하는 등 다양한 용도로 사용할 수 있습니다.

## 유틸리티 타입

그럼 대표적인 유틸리티 타입 몇 가지를 살펴볼까요?

### 1. Partial<T>

이 유틸리티 타입은 모든 속성을 선택적으로 만듭니다. 객체의 일부만 업데이트하고 싶을 때 유용하게 사용할 수 있습니다.

```tsx
interface Todo {
  title: string;
  description: string;
}

function updateTodo(todo: Todo, fieldsToUpdate: Partial<Todo>) {
  return { ...todo, ...fieldsToUpdate };
}

const todo1 = { title: 'organize desk', description: 'clear clutter' };

const todo2 = updateTodo(todo1, { description: 'throw out trash' }); // { title: 'organize desk', description: 'throw out trash' }
```

### 2. Readonly<T>

이 유틸리티 타입은 모든 속성을 읽기 전용으로 만듭니다. 객체의 속성을 변경하지 못하게 하고 싶을 때 사용합니다.

```tsx
interface Todo {
  title: string;
}

const todo: Readonly<Todo> = {
  title: 'Delete inactive users',
};

todo.title = 'Hello'; // Error: cannot reassign a readonly property
```

### 3. Pick<T, K>

이 유틸리티 타입은 특정 타입에서 몇 개의 속성만 선택할 수 있게 합니다. 즉, 타입 `T`의 하위 집합을 만드는 데 사용됩니다.

```tsx
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type TodoPreview = Pick<Todo, 'title' | 'completed'>;

const todo: TodoPreview = {
  title: 'Clean room',
  completed: false,
};
```

### 4. Omit<T, K>

이 유틸리티 타입은 특정 타입에서 원하는 속성을 제외할 수 있게 합니다. 즉, 타입 `T`에서 일부 속성을 제외한 타입을 만드는 데 사용됩니다.

```tsx
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type TodoInfo = Omit<Todo, 'completed'>;

const todoInfo: TodoInfo = {
  title: 'Pick up kids',
  description: 'Kindergarten at 4pm',
};
```

### 5. Record<K, T>

이 유틸리티 타입은 `K`를 속성 이름, `T`를 속성 값의 타입으로 하는 객체를 만들어줍니다.

```tsx
interface PageInfo {
  title: string;
}

type Page = 'home' | 'about' | 'contact';

const nav: Record<Page, PageInfo> = {
  home: { title: 'Home' },
  about: { title: 'About' },
  contact: { title: 'Contact' },
};
```

### 6. Required<T>

이 유틸리티 타입은 모든 속성을 필수로 만듭니다. 선택적 속성이 없어져야 할 때 사용합니다.

```tsx
interface Props {
  a?: number;
  b?: string;
}

const obj: Props = { a: 5 }; // OK

const obj2: Required<Props> = { a: 5 }; // Error: property 'b' missing
```

### 7. Exclude<T, U>

**이 유틸리티는 `T`에서 `U`에 할당 할 수 있는 것들을 제외합니다.**

```tsx
type T0 = Exclude<'a' | 'b' | 'c', 'a'>; // "b" | "c"
type T1 = Exclude<'a' | 'b' | 'c', 'a' | 'b'>; // "c"
```

### 8. Extract**<T, U>**

**이 유틸리티 타입은 `T`에서 `U`에 할당 할 수 있는 것들을 추출합니다.**

```tsx
type T0 = Extract<'a' | 'b' | 'c', 'a' | 'f'>; // "a"
type T1 = Extract<string | number | (() => void), Function>; // () => void
```

### 9. NonNullable<T>

**이 유틸리티는 `T`에서 `null`과 `undefined`를 제외합니다.**

```tsx
type T0 = NonNullable<string | number | undefined>; // string | number
type T1 = NonNullable<string[] | null | undefined>; // string[]
```

### 10. ReturnType<T>

**이 유틸리티는 함수 `T`의 반환 타입을 가져옵니다.**

```tsx
type T0 = ReturnType<() => string>; // string
type T1 = ReturnType<(s: string) => void>; // void
```

### 11. Awaited<T>

**이 유틸리티 타입은 비교적 최근에 추가된 유틸리티 타입으로 (v 4.5에서 추가) `Promise`에서 반환된 타입을 가져옵니다.**

```tsx
type T0 = Awaited<Promise<string>>; // string
type T1 = Awaited<Promise<() => void>>; // () => void
```
