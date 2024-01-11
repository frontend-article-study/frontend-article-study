# Electron에 대하여

일렉트론을 사용하면서 알아야 할 가장 기본적인 프로세스 및 일렉트론 구조에 대해 설명하도록 하겠습니다.

일렉트론 내부적으로 어떻게 동작하고 있는지 원리에 대한 설명보단,<br>
실제 개발할 때의 유의점, 웹과 다른 점을 위주로 발표하겠습니다.

## Electron이란?

일렉트론은 데스크탑 앱을 만드는 크로스 플랫폼 프레임워크 입니다.


기본적으로 아래와 같은 기능을 제공합니다.

- `Node.js`를 사용해서 데스크탑의 네이티브 기능들을 사용할 수 있습니다.
- 웹(`JavaScript`, `HTML`, `CSS`)을 기반으로 하기 때문에 따로 익히지 않아도 가능합니다.<br>
=> 요것이 가능한 이유는 크로미움을 내장하고 있기 때문입니다.

일렉트론은 `Node.js` + 크로미움을 내장하고 있는 데스크탑 앱 이라고 할 수 있습니다.


## Electron 프로세스

일렉트론의 프로세스는 아래와 같이 나뉘는데요, 

- `Main process`
- `Renderer process`
- `Preload scripts`
- `Utility process`

이렇게 코드 구조가 제공됩니다.

<img width="263" alt="사진1" src="https://github.com/outsung/frontend-article-study/assets/40460655/bd82f214-10b5-421f-8c85-6fe33268020a">

<img width="1382" alt="사진2" src="https://github.com/outsung/frontend-article-study/assets/40460655/4d2e0ee4-cf33-4409-82f0-40a642e9de22">

### Main process

`Main process`는 `Node.js` 기반입니다.

`Main process`에서 아래와 같은 기능을 사용할 수 있습니다.

---

`Main process`는 기본적으로 윈도우를 열고 관리하는 기능을 수행할 수 있습니다.

여기서 윈도우는 `BrowserWindow` 라는 걸 사용해서 만들 수 있습니다.

```js
const { BrowserWindow } = require('electron')

const win = new BrowserWindow({ width: 800, height: 1500 })
win.loadURL('https://github.com')
```

---

`app` 이라는 객체를 사용해서 기본적인 어플리케이션 관련 이벤트를 받아볼 수 있습니다.

```js
import { app } from 'electron';

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit()
})
```

> `Windows` 환경에서는 윈도우가 전부 닫히면 앱을 종료함.<br>
> `macOS`에서는 윈도우가 전부 닫혀도 앱이 종료되면 안 됨.


---

네이티브 기능을 사용할 수 있습니다. 메뉴, 트레이메뉴, 대화상자 등<br>
기본 데스크탑에서 지원하는 네이티브 기능들을 사용할 수 있습니다.


### Renderer process

`BrowserWindow`으로 윈도우가 생성 될 때마다 <br>
`Renderer process`가 웹을 그려줍니다.

여기서는 무조건 웹 환경처럼 동작합니다. -> `Node.js`의 `fs` & 네이티브 기능 요런건 사용 불가능.

*어? 그럼 일렉트론 사용해서 개발하면 사용자 인터페이스는 웹(`Renderer process`) 인데,<br>
웹(`Renderer process`)에서는 `Node.js` 기능 & 네이티브 기능을 사용을 못한다고??*

맞습니다 사용할 수 없습니다. 지금까지 설명드린 내용으로는 불가능합니다.


### Preload scripts

`Preload script`는 웹 컨텐츠가 그려지기 전에, 로드되는 스크립트 입니다.

`Preload script`는 `Renderer process`에서 실행 되지만,<br>
`Node.js`의 API에 접근할 수 있으므로 더 많은 기능을 사용할 수 있습니다.

`Preload script`를 사용해서 윈도우 객체에 `Node.js` API를 넘길 수 있습니다.<br>
그럼 `Renderer process`에서 window 객체에 있는 함수를 실행 할 수 있습니다.

하지만..<br>
`contextIsolation` 때문에<br>
`Preload script`와 `Renderer process`는 같은 window 객체를 공유하고 있지만<br>
`Preload script`에서 정의한 변수나 함수들은 `Renderer process`에서 값을 수정하거나 불러올 수 없습니다. (???)

#### contextIsolation 이란?

웹에서 네이티브 기능이나 `Node.js` API에 접근해서 뭔가 위험한 일을 할 수 있으므로 격리하자! 한겁니다.

실제로 아래처럼 값을 사용할 수 없습니다.

```js
// preload.js
window.myAPI = {
  desktop: true
}
```
```js
// renderer.js
console.log(window.myAPI) // undefined
```

그래서 대신 `contextBridge`를 사용해서 변수나 함수를 공유할 수 있습니다.

```js
// preload.js
const { contextBridge } = require('electron')

contextBridge.exposeInMainWorld('myAPI', {
  desktop: true
})
```

```js
// renderer.js
console.log(window.myAPI) // { desktop: true }
```

### Utility process

요거는 `Main process`에서 하위 프로세스를 만들고 싶을 때 사용하는 프로세스 입니다.
사실 `Main process`도 `Node.js` 기반이기에 `Node.js` 에 있는 `child_process`를 사용해도 됩니다.

`Utility process`의 차이점으로는 `MessagePorts`라는걸 이용해서 `Renderer process`와 통신채널을 연결 할 수 있다고 합니다.


## Process Sandboxing

말 그대로 프로세스들을 샌드박스에서 실행해버려서 권한을 제한한다 입니다.

`Preload script`에서 모든 강력한 API들을 모두 `Renderer process`한테 넘겨버리기
<br>
-> 결국 똑같은 문제발생

그래서 샌드박스를 도입해서 

`Renderer process`는 당연히 웹 환경처럼만 실행 가능합니다.

`Preload script`는 아래와 같은 제한적인 기능만 `Renderer process`한테 넘길 수 있습니다.

- `electron` (`contextBridge`, `crashReporter`, `ipcRenderer`(중요), `nativeImage`, `webFrame`)
- `events`
- `timers`
- `url`
- ...

*그럼 `Node.js` & 네이티브 기능들을 `Renderer process`에서는 실행을 못하나요???*


## 프로세스간 통신

그건 아닙니다.

Inter-process communication (`IPC`)를 통해서 `Renderer process`에서 `Main process`으로 통신을 할 수 있습니다! (반대도 가능)

`ipcMain` `ipcRenderer`를 사용해서 이벤트를 발생시키거나, 이벤트를 받을 수 있습니다.


그래서 `Node.js` & 네이티브 기능을 웹 부분과 연동, 밀접하게 사용되어야 한다면,<br>
먼저 `Main process`에서 기능을 미리 만들어두고, 이벤트를 등록을 해야합니다.

```js
// main.js
ipcMain.handle('set-title', (event, title) => {
    const webContents = event.sender
    const win = BrowserWindow.fromWebContents(webContents)
    win.setTitle(title)
});
```

당연히 `Renderer process`에서 `ipcRenderer`를 사용하나 싶었는데..<br>
사용하지 못합니다..

먼저 `Preload script`에서 `ipcRenderer.send`를 호출하는 함수를 만들어서<br>
해당 함수를 `contextBridge`를 통해 window 객체에 넣어줍니다.

```js
// preload.js
const { contextBridge, ipcRenderer } = require('electron/renderer')

contextBridge.exposeInMainWorld('electronAPI', {
  setTitle: (title) => ipcRenderer.send('set-title', title)
})
```

그러고 이제 `Renderer process`에서 사용할 수 있습니다.

```js
// renderer.js
const onClick = () => {
    window.electronAPI.setTitle(title);
}
```

### 마무리

저는 간단하게 일렉트론을 사용하면

웹에서 데스크탑 네이티브 기능 + `Node.js` API을 사용할 수 있을 줄 알았는데

이렇게 어렵게 기능을 추가해야 하는걸 배웠습니다.

실제로 개발할때 이런 구조를 처음 접하다보니 어려움이 많았던 것 같습니다.
