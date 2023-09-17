# 헤드리스 컴포넌트
headless-components는  스타일이 없고 로직만 존재하는 컴포넌트 패턴을 의미합니다. 

### UI 라이브러리의 한계 와 Headless UI 라이브러리의 등장
외부 UI 라이브러리를 사용할 경우, 개발자 입맛대로 디자인이나 기능을 수정하기가 매우 어렵다는 단점이 있습니다. ~~ 더 나아가 해당 라이브러리에 심각한 버그가 있거나, 유지보수를 종료한다고 하면 ~~ 그 컴포넌트를 버려야하는 안전성의 문제가 있습니다. 그래서 나온 개념이 Headless UI Components 입니다. 

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
|Component 기반 UI 라이브러리|기능 0 스타일 0|Material UI, Ant Design|바로 사용할 수 있는 마크업과 스타일이 존재 설정이 거의 필요 없음|마크업을 자유롭게 할 수 없고 스타일은 대부분 라이브러리에 있는 테마 기반으로만 변경할 수 있어 한정적임 큰 번들 사이즈|
|Headless UI 라이브러리|기능 0 스타일 x |Headless UI, Radix UI, Reach UI|마크업과 스타일을 완벽하게 제어 가능 모든 스타일링 패턴 지원 (ex. CSS, CSS-in-JS, UI 라이브러리 등) 작은 번들 사이즈|추가 설정이 필요함 마크업, 스타일 혹은 테마 모두 지원되지 않음, 러닝커브가 있음, |
   
### Headless UI 라이브러리 장점과 단점을 더 자세히
#### 장점
1. **재사용성** : 로직을 독립적인 컴포넌트로 캡슐화함-> 여러 UI 요소에서 이러한 컴포넌트를 재사용 가능. 이는 코드 중복 방지와  애플리케이션 전반에 걸쳐 일관성 강화.
2. **관심사 분리** : 로직과 표현을 명확하게 분리. 이로 인해 코드베이스가 더 관리하기 쉽고 이해하기 쉬워지며, 특히 업무가 분산된 대규모 팀에게 유용.
3. **유연성** : 헤드리스 컴포넌트는 프레젠테이션에 구애받지 않기 때문에 디자인 유연성을 높일 수 0. 기본 로직에 영향을 주지 않고 원하는 만큼 UI를 사용자 정의할 수 있습니다.
4. **테스트 가능성**: 프레젠테이션과 로직이 분리되어 있기 때문에, 비즈니스 로직에 대한 단위 테스트 작성이 더 쉬움.
5. 작은 번들 사이즈 
#### 단점
1. 초기 부담: 더 단순한 애플리케이션 또는 컴포넌트의 경우, 헤드리스 컴포넌트를 생성하는 것이 과도한 엔지니어링처럼 느껴져 불필요한 복잡성 초래.
2. 학습 곡선: 이 개념에 익숙하지 않은 개발자들은 처음에 이해하기 어려움.
3. 남용 가능성: 모든 컴포넌트를 헤드리스로 만들려고 하다 보면, 불필요한 경우에도 과도하게 사용하게 되어 코드베이스가 지나치게 복잡해질 수 0.
4. 잠재적 성능 문제: 일반적으로 큰 문제는 아니지만, 신중하게 처리하지 않으면 공통 로직을 사용하여 여러 컴포넌트를 다시 렌더링하는 것이 성능 문제로 이어질 수 0.

## Headless components 패턴의 대표 라이브러리 Downshift 
UI를 렌더링하지 않고 동작과 상태를 관리하는 헤드리스 컴포넌트의 개념을 적용한 대표적인 라이브러리입니다. *Render Prop과 Hooks 방식을 모두 지원한다고 하네요.

*💡 Render props pattern 이란?
패턴 render props란 props에 넘겨주는 값이 JSX요소를 반환하는 함수는 prop을 말한다. render props는 코드의 재사용성을 높임과 동시에, 관심사의 분리를 이룰 수 있도록 만들어 준다.
 
### 사용법
**설치**
```shell
$ npm i downshift
```
**적용**
 ```jsx
import { useSelect } from "downshift"

const StateSelect = () => {
  const {
    isOpen, // 드롭다운 열려 있는 지 상태관리
    selectedItem,//현재 선택된 상태 값 관리
    getToggleButtonProps,
    getLabelProps,
    getMenuProps,
    highlightedIndex,// 현재 강조 표시된 목록 항목 
    getItemProps,
  } = useSelect({items: states});

  return (
    <div>
      <label {...getLabelProps()}>Issued State:</label>
      <div {...getToggleButtonProps()} className="trigger" >
        {selectedItem ?? 'Select a state'}
      </div>
      <ul {...getMenuProps()} className="menu">
        {isOpen &&
          states.map((item, index) => (
            <li
              style={
                highlightedIndex === index ? {backgroundColor: '#bde4ff'} : {}
              }
              key={`${item}${index}`}
              {...getItemProps({item, index})}
            >
              {item}
            </li>
          ))}
      </ul>
    </div>
  )
}
```
- 드롭다운이 열려 있으면, states.map은 선택 가능한 상태의 정렬되지 않은 목록을 생성합니다.
- 스프레드 (...) 연산자는 Downshift의 훅에서 컴포넌트에 props를 전달하는 데 사용됩니다. 여기에는 클릭 핸들러, 키보드 탐색 및 ARIA 속성과 같은 것들이 포함됩니다.
- 상태가 선택되면 버튼 내용으로 표시됩니다. 그렇지 않으면 ‘Select a state’라고 표시됩니다.

# 정리 
디자인이 그렇게 중요하지 않고, 커스텀할 곳이 많지 않다면 Component 기반 라이브러리를,  하지만 만약 반응형에 따라 디자인이 달라지고, 기능 변경이나 추가가 많이 발생한다면 Headless 라이브러리가 유지보수에 더 좋다는 입장입니다.

### 출처
(번역) React에서 UI와 로직 분리하기:헤드리스 컴포넌트를 사용한 클린 코드 접근법
https://soobing.github.io/react/decoupling-ui-and-logic-in-react-a-clean-code-approach-with-headless-components/

headless 컴포넌트 
https://www.howdy-mj.me/design/headless-components
