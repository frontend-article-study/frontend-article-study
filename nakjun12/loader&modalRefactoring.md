### 발표 자료

### **react-router-dom v6**

React에서 사전에 데이터를 미리 받아올 수 있는 기능을 지원한다고 하여 해당 기능을 적용해보았습니다.

```jsx
createBrowserRouter([
  {
    element: <Teams />,
    path: "teams",
    loader: async () => {
      return fakeDb.from("teams").select("*");
    },
    children: [
      {
        element: <Team />,
        path: ":teamId",
        loader: async ({ params }) => {
          return fetch(`/api/teams/${params.teamId}.json`);
        }
      }
    ]
  }
]);
```

**loader prefetch 적용**

 <img src="https://velog.velcdn.com/images/wns450/post/e25aa446-8b81-42e5-aca8-9e846d23af12/image.gif" width="100%" height="100%">

**Before**

초기에 이미지가 빈배열이어서 레이아웃이 다시 잡혔습니다.

(스켈레톤이나 다른 방식을 통해 레이아웃을 잡을 수 있지만 차이를 보기 위해 적용하지 않았습니다.)

<br/>

 <img src="https://velog.velcdn.com/images/wns450/post/6a6d2d1c-57fa-48c6-9065-7b2a8ef1e664/image.gif" width="100%" height="100%">

**After**

초기에 prefetch를 통해 데이터가 받아와서 이미지를 넣고 시작할 수 있습니다.

<br/>
<br/>
<br/>

### 전역 Modal 관리 리펙토링

기존에는 content를 통해 JSX 받아서 다양한 스타일을 커버하였습니다.

```jsx
const Modal = () => {
  const { isOpen, content, closeOnBackdrop, closeModal } = useModalStore();

  const closePopup = closeOnBackdrop ? closeModal : null;
  return (
    <Popup isOpen={isOpen} closePopup={closePopup}>
      {content}
    </Popup>
  );
};
```

Zustand를 불러와 JSX를 넘겨주는 형식으로 설계하였습니다.

```jsx
* // `useModalStore`에서 필요한 함수를 가져옵니다.
 * const { openModal, closeModal } = useModalStore();
 *
 * // 팝업을 열기 위한 함수입니다.
 * const openCustomPopup = () => {
 *   const customContent = (
 *     <>
 *       <h2>Popup Title</h2>
 *       <p>Popup Content</p>
 *       <button onClick={closeModal}>Close Popup</button>
 *     </>
 *   );
 *   openModal(customContent); // 백드롭 클릭으로 팝업을 닫습니다.
 *  openModal(customContent, false); // 백드롭 클릭으로 팝업을 닫지 않습니다.
 * };
```

**아쉬운 점**

사람마다 넣는 모달 스타일이 달라서 재사용성이 아쉽고,

상태관리의 목적과 React 선언적 프로그래밍과 거리가 멀다는 점이 아쉬워서 리펙토링 하였습니다.

**리펙토링**

```jsx
* // 사용법 예시:
 * const { openModal, closeModal } = useModalStore();
 *
 * // 커스텀 팝업 열기
 * openModal({
 *   modalType: "default",
 *   modalProps: {
 *     title: `가입에 성공했습니다.`,
 *     message,
 *     confirmText: "확인",
 *     onConfirm: closeModal
 *   }
 * });
```

직접 jsx를 받는 형식에서 modalType과 modalProps를 zustand를 통해 업데이트할 수 있게 하였습니다.

저번과는 다르게 사용하기에 앞서 모달 컴포넌트를 등록해주어야합니다.

```jsx
const DefaultModal = lazy(() => import("./modal/BaseModal"));
const AnotherModal = lazy(() => import("./modal/AnotherModal"));

const modalComponents = {
  default: DefaultModal,
  anotherModalType: AnotherModal
  // 다른 모달 타입에 대한 정의를 추가할 수 있습니다.
};

const Modal = () => {
  const { isOpen, modalType, modalProps, closeOnBackdrop, closeModal } =
    useModalStore();

  const closePopup = closeOnBackdrop ? closeModal : null;

  const SelectedModal = modalComponents[modalType] || modalComponents.default;

  return (
    <Popup isOpen={isOpen} closePopup={closePopup}>
      <SelectedModal {...modalProps} />
    </Popup>
  );
};

export default Modal;
```

modalComponent 객체로 관리하여서 이를 사용하는 방식으로 수정하였습니다.

```jsx
const BaseModal = (props) => {
  // props 객체에서 필요한 값들을 추출합니다.
  const {
    title,
    message,
    onConfirm,
    confirmText = "확인" // 기본값 설정
  } = props;

  return (
    <div className="modal-box">
      <h3 className="font-bold text-lg">{title}</h3>
      <p className="py-4">{message}</p>
      <div className="modal-action">
        <button className="btn" onClick={onConfirm}>
          {confirmText}
        </button>
      </div>
    </div>
  );
};

export default BaseModal;
```

위와 같이 props를 받아서 사용할 수 있게 만들었습니다.

### 재사용을 높이는 문서 방식에 관한 고민

**1) JSDoc 작성 방식 수정**

````jsx
//모달 컨텐츠 스타일 정의
const ModalContent = styled.div`
  background-color: white;
  padding: 20px;
  border-radius: 5px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
`;

/**
 * 사용 예시:
 *
 * 아래 예시는 `Modal` 컴포넌트를 사용하여 사용자 정의 팝업을 생성하고 열기/닫기 기능을 구현합니다.
 *
 * ```javascript
 * // `useModalStore`에서 필요한 함수를 가져옵니다.
 * const { openModal, closeModal } = useModalStore();
 *
 * // 팝업을 열기 위한 함수입니다.
 * const openCustomPopup = () => {
 *   const customContent = (
 *     <>
 *       <h2>Popup Title</h2>
 *       <p>Popup Content</p>
 *       <button onClick={closeModal}>Close Popup</button>
 *     </>
 *   );
 *   openModal(customContent); // 백드롭 클릭으로 팝업을 닫습니다.
 *  openModal(customContent, false); // 백드롭 클릭으로 팝업을 닫지 않습니다.
 * };
 *
 * // 버튼 클릭 시 `openCustomPopup` 함수를 호출하여 팝업을 엽니다.
 * <button onClick={openCustomPopup}>Open Custom Popup</button>
 * ```
 *
 * 위 예시에서 `openModal` 함수는 사용자 정의 컨텐츠를 받아 모달을 열고,
 * `closeModal` 함수는 모달을 닫습니다.
 */
````

**Before**

기존에는 하단에 작성하여서 jsdoc 이를 인지하지 못하여서 제대로 나오지 않았습니다.

```jsx
/**
 * Modal 컴포넌트는 팝업 UI를 제공합니다.
 * 이 컴포넌트는 `useModalStore` 훅을 사용하여 팝업의 상태를 관리하며,
 * `Popup` 컴포넌트를 사용하여 실제 모달을 렌더링합니다.
 *
 * - `useModalStore`를 통해 모달의 상태(isOpen, content 등)를 관리합니다.
 * - 모달을 열고 닫는 동작은 `useModalStore`의 액션(openModal, closeModal)을 통해 수행됩니다.
 * - 백드롭 클릭으로 모달을 닫는 동작은 `closeOnBackdrop` 상태에 따라 결정됩니다.
 *
 * 이 컴포넌트는 훅을 통한 상태 관리와 컴포넌트 렌더링을 분리하는 방식을 사용합니다.
 *
 * @returns {JSX.Element} 모달 컴포넌트.
 *
 * @example
 * // 사용법 예시:
 * const { openModal, closeModal } = useModalStore();
 *
 * // 커스텀 팝업 열기
 * openModal({
 *   modalType: "default",
 *   modalProps: {
 *     title: `가입에 성공했습니다.`,
 *     message,
 *     confirmText: "확인",
 *     onConfirm: closeModal
 *   }
 * });
 */
```

**After**

수정 이후에는 JSDoc 문법에 맞게 작성하였고, 아래처럼 문서를 뽑을 수 있었습니다.

 <img src="https://velog.velcdn.com/images/wns450/post/143860d9-03b3-4edd-b1cf-fa385eb00f00/image.png" width="100%" height="100%">

JSDoc을 작성하는 것도 좋았지만, 사용할 줄 아는 사람 입장에서는 설명문이 있는 것이 번거로울 수 있다고 생각하여서 아쉬웠습니다.

그래서 이를 분리하는 방법에 관해 다시 고민해보았습니다.

````markdown
# 모달 컴포넌트

## 개요

`Modal` 컴포넌트는 사용자 인터페이스에 팝업 형태의 UI를 제공합니다. 모달의 상태 관리를 위해 `useModalStore` 훅을 활용하며, `Popup` 컴포넌트를 사용하여 실제 모달 내용을 렌더링합니다.

### 핵심 기능

- **상태 관리**: `useModalStore` 훅을 통해 모달의 상태(예: 열림/닫힘 상태, 내용 등)를 관리합니다.
- **액션 처리**: 모달의 열기와 닫기 동작은 `useModalStore` 내의 `openModal` 및 `closeModal` 액션을 통해 처리됩니다.
- **백드롭 클릭**: `closeOnBackdrop` 속성에 따라 백드롭(모달 외부) 클릭 시 모달을 닫을지 결정합니다.

### 컴포넌트 구성

- `DefaultModal`: 기본 모달 컴포넌트로, 기본적인 모달 UI를 제공합니다.
- `AnotherModal`: 다양한 시나리오나 스타일에 맞춰 커스터마이징 가능한 추가 모달 컴포넌트입니다.

## 사용 방법

```jsx
import { useModalStore } from "@/utils/hooks/store/useModalStore";
import Modal from "@/components/popup/Modal";

const App = () => {
  const { openModal, closeModal } = useModalStore();

  const handleOpenModal = () => {
    openModal({
      modalType: "default",
      modalProps: {
        title: "가입에 성공했습니다.",
        message: "환영합니다!",
        confirmText: "확인",
        onConfirm: closeModal
      }
    });
  };

  return (
    <div>
      <button onClick={handleOpenModal}>모달 열기</button>
      <Modal />
    </div>
  );
};
```
````

위 예시는 `openModal` 함수를 사용하여 모달을 열고, 모달 내의 확인 버튼을 통해 `closeModal` 함수를 호출하여 모달을 닫는 과정을 보여줍니다.

## 컴포넌트 속성

- `isOpen`: 모달의 열림 및 닫힘 상태를 제어하는 불리언 값입니다.
- `modalType`: 렌더링할 모달 컴포넌트의 유형을 지정합니다. (`default`, `anotherModalType` 등)
- `modalProps`: 모달 컴포넌트로 전달될 속성들을 지정하는 객체입니다.
- `closeOnBackdrop`: 백드롭 클릭 시 모달을 닫을지 여부를 제어하는 불리언 값입니다.

## 추가 커스터마이제이션

모달 컴포넌트를 추가로 커스터마이즈하고자 한다면, `modalComponents` 객체에 새로운 모달 유형과 해당 컴포넌트를 정의하세요. 각 모달 컴포넌트는 필요에 따라 특정 스타일이나 기능을 포함할 수 있습니다.

이 문서는 `Modal` 컴포넌트의 주요 기능, 사용법, 속성 및 확장 방법에 대해 설명합니다. 필요에 따라 더 많은 세부 정보나 추가 섹션을 포함하여 문서를 확장할 수 있습니다.

```

md 파일로 분리하여서 파일을 두어서 사용법 md 파일로 관리하는 방법도 좋은 대체 방안이 될 수 있을 것 같습니다.
```
