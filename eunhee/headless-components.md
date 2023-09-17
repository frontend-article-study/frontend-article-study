# 헤드리스 컴포넌트
headless-components는  스타일이 없고 로직만 존재하는 컴포넌트 패턴을 의미합니다. 

### UI 라이브러리의 한계 와 Headless UI 라이브러리의 등장
외부 UI 라이브러리를 사용할 경우, 개발자 입맛대로 디자인이나 기능을 수정하기가 매우 어렵다는 단점이 있습니다.  더 나아가 해당 라이브러리에 심각한 버그가 있거나, 유지보수를 종료한다고 하면 그 컴포넌트를 버려야하는 안전성의 문제가 있습니다. 그래서 나온 개념이 Headless UI Components 입니다. 

### 헤드리스 컴포넌트 패턴을 이용한 리팩토링: 
**즉 두 개의 유사한 컴포넌트에서 ‘useToggle’ 훅을 추출하여 코드 중복을 줄이는 방법** <br/>
여기 토글 컴포넌트와 드롭다운 컴포넌트가 있습니다. 두가지 컴포넌트의 코드를 살펴보고 ‘useToggle’ 훅을 추출해봅시다. 하단 두개의 사진에서 보이는 것처럼 토글 컴포넌트와 드롭다운 컴포넌트의 UI는 다릅니다. 하지만 코드 로직에는 명백한 유사성이 있습니다. ToggleButton의 ‘on’ 과 ‘off’ 상태는 Dropdown의 ‘펼치기(expand)’ 와 ‘접기(collapse)’ 작업과 유사합니다. 따라서 각각의 UI와 공통이되는 로직을 분리하여 커스텀 훅을 다음과 같이 작성할 수 있습니다. 

<br/>
**토글 컴포넌트**
<br/>
<img src="./img/toggleComponent.png" alt="토글 컴포넌트" width="300" >
<br/>
**드롭다운 컴포넌트**
<br/>
<img src="./img/dropdownComponent.png" alt="드롭다운 컴포넌트"  width="300" >

토글 컴포넌트 코드입니다. 
```jsx
const ToggleButton = () => {
  const [isToggled, setIsToggled] = useState(false);

  const toggle = useCallback(() => {
    setIsToggled((prevState) => !prevState);
  }, []);

  return (
    <div className="toggleContainer">
      <p>Do not disturb</p>
      <button onClick={toggle} className={isToggled ? "on" : "off"}>
        {isToggled ? "ON" : "OFF"}
      </button>
    </div>
  );
};
```
하단 코드는 드롭다운 컴포넌트 코드 입니다. 
```jsx
const Dropdown = ({ title, children }: DropdownType) => {
  const [isOpen, setIsOpen] = useState(false);

  const toggleOpen = useCallback(() => {
    setIsOpen((prevState) => !prevState);
  }, []);

  return (
    <div>
      <h2 onClick={toggleOpen}>{title}</h2>
      {isOpen && <div>{children}</div>}
    </div>
  );
};
```
공통이되는 로직을 분리하여 만든 useToggle 커스텀 훅
```jsx
const useToggle = (init = false) => {
  const [state, setState] = useState(init);

  const toggle = useCallback(() => {
    setState((prevState) => !prevState);
  }, []);

  return [state, toggle];
};
```
## Component 기반 UI 라이브러리 vs Headless UI 라이브러리
|라이브러리 |정의|종류|장점|단점|
|------|---|---|---|---|
|Component 기반 UI 라이브러리|기능과 스타일이 존재하는 라이브러리|Material UI, Ant Design|바로 사용할 수 있는 마크업과 스타일이 존재
설정이 거의 필요 없음|마크업을 자유롭게 할 수 없음
스타일은 대부분 라이브러리에 있는 테마 기반으로만 변경할 수 있어 한정적임
큰 번들 사이즈|
|Headless UI 라이브러리|기능은 있지만 스타일이 없는 라이브러리|Headless UI, Radix UI, Reach UI|마크업과 스타일을 완벽하게 제어 가능
모든 스타일링 패턴 지원(ex. CSS, CSS-in-JS, UI 라이브러리 등)
작은 번들 사이즈|추가 설정이 필요함
마크업, 스타일 혹은 테마 모두 지원되지 않음 |

Component 기반 UI 라이브러리는 기능과 스타일이 존재하는 라이브러리를 말하며, 대표적으로 Material UI, Ant Design가 있다.

장점	
바로 사용할 수 있는 마크업과 스타일이 존재
설정이 거의 필요 없음
단점	
마크업을 자유롭게 할 수 없음
스타일은 대부분 라이브러리에 있는 테마 기반으로만 변경할 수 있어 한정적임
큰 번들 사이즈

Headless는 기능은 있지만 스타일이 없는 라이브러리로, Headless UI, Radix UI, Reach UI 등이 있다.

장점	
마크업과 스타일을 완벽하게 제어 가능
모든 스타일링 패턴 지원(ex. CSS, CSS-in-JS, UI 라이브러리 등)
작은 번들 사이즈
단점	
추가 설정이 필요함
마크업, 스타일 혹은 테마 모두 지원되지 않음

# 정리 
디자인이 그렇게 중요하지 않고, 커스텀할 곳이 많지 않다면 Component 기반 라이브러리를 사용하면 된다. 하지만 만약 반응형에 따라 디자인이 달라지고, 기능 변경이나 추가가 많이 발생한다면 Headless 라이브러리가 유지보수에 더 좋을 것 같다.

혹은 회사에 디자이너가 없거나 기한이 촉박한 프로젝트라면, 아예 특정 UI 라이브러리만을 사용해서 만드는 경우도 있다.

# Downshift 
헤드리스 컴포넌트 패턴을 사용하여 동작(또는 상태 관리)과 표현을 분리한 대표적인 라이브러리입니다. 







이 패턴이 정확히 무엇인지, 
왜 유용한지,
인터페이스 디자인에 대한 접근 방식을 어떻게 혁신할 수 있는지

장점
-  유연성, 재사용 가능성, 코드베이스의 구성과 깔끔함 향상

### 출처
(번역) React에서 UI와 로직 분리하기:헤드리스 컴포넌트를 사용한 클린 코드 접근법
https://soobing.github.io/react/decoupling-ui-and-logic-in-react-a-clean-code-approach-with-headless-components/

headless 컴포넌트 
https://www.howdy-mj.me/design/headless-components
