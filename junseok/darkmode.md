## Vanila javascript에 다크모드 적용하기
- 순수 자바스크립트에서 darkmode를 구현하는 것은 css를 활용하는 방식을 통해 이루어집니다. 크게 두 가지 방법이 있습니다.
    1. html의 data attribute를 활용하기
    2. class 활용하기 

### 1. data attribute 활용하기 
- 특정 data attribute가 html에 적용되었을 때, 값에 따라 css 변수가 적용되도록 설정 합니다.
- "color-mode"라는 data-*에 "dark" 혹은 "light" 값을 설정합니다. 해당 data에 따라 css 변수가 변경 되도록 아래와 같이 설정합니다.
```css
/* root를 활용한 다크 모드 제어*/
:root{
  --background: white
  --textColor: black
}

[data-color-mode="dark"]{
  --background: #000
  --textColor: white
}

body{
  background-color: var(--background);
  color: var(--textColor);
}

```
- 자바스크립트를 활용하여, 특정 버튼, input이 활성화 되어 있을 때 darkmode css가 동작하도록 합니다.
- 자바스크립트를 활용해 toggle 버튼을 생성하고 toggle 버튼을 클릭할 때 마다, darkmode 활성화를 구현합니다.
```js {31}
class DarkModeToogle {
  isDarkMode = null;  
  constructor({ $target, onSearch }) {
    // toggle 버튼 생성
    const $darkModeToogle = document.createElement("input");
    this.$darkModeToogle = $darkModeToogle;
    this.$darkModeToogle.type = "checkbox";

    $darkModeToogle.className = "darkModeToogle";
    $target.appendChild($darkModeToogle);
    // 이벤트 달기
    $darkModeToogle.addEventListener("change", e => {
        // 클릭 될 때 마다 data-color-mode의 값을 dark, white 중 하나로 변경시킵니다.
        this.setColor(e.target.checked)
    });

    // 초기화 진행
    this.initColorMode()
  }

  // 다크모드 pc 상태와 맞추어 주기 초기화
  // js를 활용한 media query 접근하기 true, false 반환
  initColorMode(){
    this.isDarkMode = window.matchMedia('(prefers-color-scheme: dark)').matches;
    this.$darkModeToogle.checked = this.isDarkMode
    this.setColor(this.isDarkMode) 
  }

  // darkmode 상태 관리 값 정리 
  setColor(isDarkMode){
    document.body.dataset.theme = isDarkMode ? "dark" : "light"
  }
}
```

### 2. class 속성 활용하기
- 위의 작업을 class 값 변경을 통해 해결할 수 있습니다. 
```css
/* class를 활용한 다크 모드 제어*/
:root{
  --background: white
  --textColor: black
}

.dark {
  --background: #000
  --textColor: white
}

body{
  background-color: var(--background);
  color: var(--textColor);
}
```
```js {31-32}
class DarkModeToogle {
  isDarkMode = null;  
  constructor({ $target, onSearch }) {
    // toggle 버튼 생성
    const $darkModeToogle = document.createElement("input");
    this.$darkModeToogle = $darkModeToogle;
    this.$darkModeToogle.type = "checkbox";

    $darkModeToogle.className = "darkModeToogle";
    $target.appendChild($darkModeToogle);
    // 이벤트 달기
    $darkModeToogle.addEventListener("change", e => {
        // 클릭 될 때 마다 data-color-mode의 값을 dark, white 중 하나로 변경시킵니다.
        this.setColor(e.target.checked)
    });

    // 초기화 진행
    this.initColorMode()
  }

  // 다크모드 pc 상태와 맞추어 주기 초기화
  // js를 활용한 media query 접근하기 true, false 반환
  initColorMode(){
    this.isDarkMode = window.matchMedia('(prefers-color-scheme: dark)').matches;
    this.$darkModeToogle.checked = this.isDarkMode
    this.setColor(this.isDarkMode) 
  }

  // darkmode 상태 관리 값 정리 
  setColor(isDarkMode){
    if(isDarkMode) document.body.clasList.add("dark")
    else document.body.clasList.remove("dark")
  }
}
```

## React에서 다크모드 구현하기
- React는 단방향 바인딩을 활용해서 data를 관리합니다. 또한 일반적으로 data는 prop을 활용해 상위 컴포넌트에서 하위 컴포넌트로 전해지는 방식을 활용합니다. 
이러한 특징을 활용해서 **darkmode를 위한 state**를 만들어서 css가 변경되는 모든 컴포넌트를 업데이트 해야 합니다. 
- 이때 하위 컴포넌트에 **darkmode를 위한 state**를 전달하기 위해 굳이 변경이 필요없는 컴포넌트에게도 prop을 전달해야 하고 이는 불 필요한 업데이트를 만들고 state관리를 어렵게 합니다.
따라서 ContextAPI와 state를 활용하여 darkmode를 관리하거나 전역 state 관리를 위한 라이브러리를 활용해서 darkmode를 구현할 수 있습니다. 혹은 localstorage와 contextAPI를 조합해서 활용할 수 있습니다.
아래의 예는 localstorage와 contextAPI를 조합한 예시입니다.

```jsx
// https://www.gatsbyjs.com/blog/2019-01-31-using-react-context-api-with-gatsby/
// context 생성
const defaultState = {
  dark: false,
  toggleDark: () => {},
}

const ThemeContext = React.createContext(defaultState)

// media 쿼리를 통한 dark 세팅 초기화
const supportsDarkMode = () =>
  window.matchMedia("(prefers-color-scheme: dark)").matches === true

// DarkMode Provider 생성
class ThemeProvider extends React.Component {
  state = {
    dark: false,
  }

  toggleDark = () => {
    let dark = !this.state.dark
    localStorage.setItem("dark", JSON.stringify(dark))
    this.setState({ dark })
  }

  componentDidMount() {
    // Getting dark mode value from localStorage!
    const lsDark = JSON.parse(localStorage.getItem("dark"))
    if (lsDark) {
      this.setState({ dark: lsDark })
    } else if (supportsDarkMode()) {
      this.setState({ dark: true })
    }
  }

  render() {
    const { children } = this.props
    const { dark } = this.state
    return (
      <ThemeContext.Provider
        value={{
          dark,
          toggleDark: this.toggleDark,
        }}
      >
        {children}
      </ThemeContext.Provider>
    )
  }
}

export default ThemeContext

export { ThemeProvider }
```
```jsx
import React from "react"

import { ThemeProvider } from "./src/context/ThemeContext"

export const wrapRootElement = ({ element }) => (
  <ThemeProvider>{element}</ThemeProvider>
)
```
- 해당 방법 이외에 useRef, useEffect, css 변수를 활용해서 다크모드를 구현할 수 있습니다. 
```jsx
// app.jsx
import React from "react";
import styled from "styled-components";
import "../App.scss";

function App() {
  useEffect(() => {
    const bgMode = window.localStorage.getItem("bgMode");
    if (bgMode === "dark") {
      document.getElementsByTagName("html")[0].classList.add("ui-dark");
    }
  }, []);

  const darkOnOff = () => {
    if (
      document.getElementsByTagName("html")[0].classList.contains("ui-dark")
    ) {
      document.getElementsByTagName("html")[0].classList.remove("ui-dark");
      window.localStorage.setItem("bgMode", "light");
    } else {
      document.getElementsByTagName("html")[0].classList.add("ui-dark");
      window.localStorage.setItem("bgMode", "dark");
    }
  };

  return (
    <Background>
      <span>dark mode</span>
      <button onClick={darkonOff}>on/off darkMode</button>
    </Background>
  );
}
```

## Next.js에서 darkmode 구현하기
- (next-themes)[https://github.com/pacocoursey/next-themes]라는 라이브러리를 통해 쉽게 darkmode를 구현할 수 있습니다. 해당 라이브러리는 page router, app router 모두 지원하고 있습니다.
- 해당 라이브러리를 사용하면, 화면 깜빡임 등을 쉽게 해결하여 darkmode를 구현할 수 있습니다. 
```
$ npm install next-themes
$ yarn add next-themes
```

1. **page router**
- next-themes를 활용하기 위해서는 Custom App이 필요합니다. 이는 `_app`을 통해 쉽게 구현할 수 있습니다.
```jsx
// pages/_app.js

import { ThemeProvider } from 'next-themes'

function MyApp({ Component, pageProps }) {
  return (
    <ThemeProvider>
      <Component {...pageProps} />
    </ThemeProvider>
  )
}

export default MyApp
```


2. **app router**
- `layout.jsx`에서 next-theme을 사용합니다. 
```jsx
// app/layout.jsx

import { Providers } from './providers'

export default function Layout({ children }) {
  return (
    <html suppressHydrationWarning>
      <head />
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  )
}
```
```jsx
// app/providers.jsx

'use client'

import { ThemeProvider } from 'next-themes'

export function Providers({ children }) {
  return <ThemeProvider>{children}</ThemeProvider>
}
```

- `next-themes`애서 제공하는 hook `useTheme`을 활용해서 쉽게 다크모드를 전환하는 버튼을 생성할 수 있습니다. 
- 이때 주의해야 할 점은 해당 기능은 client에서 이루어진다는 점입니다. 따라서 client에 마운트 되기 전까지, 해당 값은 `undefined`입니다. 해당 문제를 고치기 위해서는
 페이지가 client에 마운트될 때 현재 테마를 사용하는 UI만 렌더링해야 합니다

```jsx {9-11}
import { useState, useEffect } from 'react'
import { useTheme } from 'next-themes'

const ThemeSwitch = () => {
  const [mounted, setMounted] = useState(false)
  const { theme, setTheme } = useTheme()

  // useEffect only runs on the client, so now we can safely show the UI
  useEffect(() => {
    setMounted(true)
  }, [])

  if (!mounted) {
    return null
  }

  return (
    <select value={theme} onChange={e => setTheme(e.target.value)}>
      <option value="system">System</option>
      <option value="dark">Dark</option>
      <option value="light">Light</option>
    </select>
  )
}

export default ThemeSwitch
```

3. **css 설정하기** 
- html attribute중 data-theme 속성에 따라 css 변수를 설정하여 쉽게 다크모드에 따른 색상을 수정할 수 있습니다.
```css
:root {
  /* Your default theme */
  --background: white;
  --foreground: black;
}

[data-theme='dark'] {
  --background: black;
  --foreground: white;
}
```

4. **tailwindcss**
- tailwindcss에서 다크모드 사용을 운영 체제의 의존하지 않고 사용하기 위해서는 다음과 같은 설정이 필요합니다. 
우선 css `media` 프로퍼티 사용이 아닌 `class` 사용을 통해 dark mode를 조정하겠다는 것을 명시해야 합니다.
```js
//tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  darkMode: 'class',
  // ...
}
```
- 이제 HTML class 상에 `dark:{class}` class가 있을 경우 다크모드에서 해당 css를 적용합니다.
```html
<!-- Dark mode enabled -->
<html class="dark">
<body>
  <!-- Will be black -->
  <div class="bg-white dark:bg-black">
    <!-- ... -->
  </div>
</body>
</html>
```


## Reference
- [vanilla javascript darkmode](https://jeonghwan-kim.github.io/dev/2021/05/17/css-variable.html)
- [react-darkmode-예시](https://kyounghwan01.github.io/blog/etc/CSS/dark-mode/)
- [react-darkmode-예시, 게츠비](https://www.gatsbyjs.com/blog/2019-01-31-using-react-context-api-with-gatsby/)
- [tailwindcss darkmode](https://tailwindcss.com/docs/dark-mode)
- [next-theme](https://github.com/pacocoursey/next-themes)