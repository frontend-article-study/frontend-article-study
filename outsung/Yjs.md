# Yjs 사용해보기

## Yjs 란?

실시간으로 데이터가 동기화되는 어플리케이션을 만들기 위한 고성능 CRDT입니다.

### CRDT 란?
Conflict free replicated data type 충돌 없는 복제 데이터 유형 입니다.

Yjs를 사용할때 가장 기본적인 프로세스는<br/>
변경사항이 발생 -> 이벤트를 발생 -> 병합 충돌 없이 자동으로 병합 입니다.


### 클라이언트 쪽에서는
이벤트 발생 + 데이터 다시 동기화 해주기

> 만약 `rich-text` 에디터를 실시간 동기화 하고 싶으면,
> 지원하는 편집기 목록에서 골라쓰면 됩니다.
> 
> - ProseMirror
> - TipTap
> - Monaco
> - Quill
> - CodeMirror
> - Remirror
> - Milkdown
> - Slate

### 서버 쪽에서는
`Connection Provider`라는 것을 통해 데이터 병합 및 데이터 내려주기

> 여러 방식으로 데이터를 주고 받을 수 있습니다.
> - y-websocket
> - y-webrtc
> - y-dat
> - Liveblocks
> - y-sweet
> - PartyKit
> - y-libp2p
> - MatrixCRDT
> - yrb-actioncable
> - ypy-websocket

`DB Provider` 를 통해 DB에 저장 가능합니다.

> - y-indexeddb
> - y-leveldb
> - y-redis
> - y-mongodb-provider

## 독립적이다?

백엔드가 어떻게 구성 되어있는지<br/>
데이터를 받고 넘길때 하나의 방식만 사용해야하는가?

이런 고민을 하지 않아도됩니다.<br/>
Yjs는 네트워크와 분리되어있는 소프트웨어입니다.

(provider 같은걸 추가로 지원함) -> 커스텀도 가능

## 성능
Yjs가 현존하는 CRDT 중 가장 빠르다고 합니다.


### 간단하게 사용해보기

각 사용자들이 만약 실시간으로 동기화 되는 어플리케이션을 만들때는
어떤 데이터들을 주고 받을지는 아래에서 타입을 선택하고, 만들 수 있다.

### Shared Types
> - Y.Map
> - Y.Array
> - Y.Text
> - Y.XmlFragment
> - Y.XmlElement
> - Y.XmlText


일단 간단하게 사용해보는거니까, 

실제 데이터를 공유하지는 않고, 유저별로 데이터를 저장 공유 할 수 있다고 한다.<br/>
(협업툴에서 다른 사람 커서 등 에서 쓰임)

`Awareness`라는 기능을 이용해서 현재 사용자을 불러올 수 있으니까 요걸 사용해보겠다.

y-webrtc를 사용해서 서버없이(p2p) 해보겠다.

```typescript
"use client";
import * as Y from "yjs";
import { WebrtcProvider } from "y-webrtc";

import { useEffect, useState } from "react";

interface User {
  name: string;
  color: string;
  x: string;
  y: string;
}
const yDoc = new Y.Doc();
const provider = new WebrtcProvider("yjs-example", yDoc, {
  signaling: ["wss://expressjs-on-koyeb-outsung.koyeb.app"],
});

export function useYjs(): User[] {
  const [users, setUsers] = useState<User[]>([]);

  useEffect(() => {
    const name = Math.random();
    const color = randomColor();
    provider.awareness.setLocalStateField("user", {
      name,
      color,
      x: undefined,
      y: undefined,
    });

    provider.awareness.on("change", (changes: any) => {
      setUsers(
        Array.from(provider.awareness.getStates().values()).map((a) => a.user)
      );
    });

    window.addEventListener("mousemove", (e) => {
      provider.awareness.setLocalStateField("user", {
        name,
        color,
        x: e.pageX,
        y: e.pageY,
      });
    });
  }, []);

  return users;
}

function randomColor() {
  const r = Math.floor(Math.random() * 255);
  const g = Math.floor(Math.random() * 255);
  const b = Math.floor(Math.random() * 255);
  return `rgb(${r},${g},${b})`;
}
```

```typescript
"use client";

import { useYjs } from "@/hooks";

export function UsersMouse() {
  const users = useYjs();

  return (
    <div style={{ position: "absolute", zIndex: 100 }}>
      {users.map((user, index) => (
        <div
          style={{
            position: "absolute",
            top: user.y,
            left: user.x,
            backgroundColor: user.color,
            width: 10,
            height: 10,
            borderRadius: 10,
          }}
          key={index}
        />
      ))}
    </div>
  );
}

```

코드처럼 `Awareness`에서 유저 변경 사항을 옵저빙하고,
내정보는 마우스 포인터에 따라 수정한다.

이렇게 배포 해보니 nextjs 로 만들고 버셀에 배포하니 같은 데이터가 브라우저에서만 공유된다.

시그널링 서버를 예시로 제공해주길래 

요것도 배포해서 해봤다.

https://yjs-example-brown.vercel.app/

