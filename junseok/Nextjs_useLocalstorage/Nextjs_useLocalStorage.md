## localStorage 사용하기

Next.js로 프로젝트를 진행하는 과정에서 localStorage에 jwt 토큰 값과 각 페이지의 방문 여부를 localstorage에 저장하고 관리해야 하는 상황이 발생했습니다. localStorage는 client side에서 접근할 수 있기 때문에, 관련된 컴포넌트를 csr 형식으로 만들고 관련 라이브러리에서 이를 호출하여 사용을 했습니다. 
```jsx
'use client';
import { useLocalStorage } from '@uidotdev/usehooks';
import { useRouter } from 'next/navigation';
import { ChangeEvent, useState } from 'react';
import { postUser } from '@/api/clientApi';
import { PATH, VISITED, GENDER } from '@/config';

export default function ChoiceSex() {
  const [sex, setSex] = useState<Gender>(GENDER.man);
  const [nickname, setNickName] = useState('');
  const [, saveToken] = useLocalStorage<null | string>('token', null);
  const [, saveIsVisited] = useLocalStorage<null | { [key: string]: boolean }>(
    'isVisited',
    null,
  );
  const router = useRouter();

  const handleSubmit = async (e: ChangeEvent<HTMLFormElement>) => {
    e.preventDefault();
    if (nickname.trim() === '') alert('Nickname을 작성해주세요!');

    const jwt = await postUser(nickname, sex);

    if (jwt) {
      saveToken(jwt.token);
      saveIsVisited(VISITED);
      router.push(PATH.chatingList);
    } else alert('token 생성에 실패했습니다.');
  };
}
```
이때 해당 프로젝트를 build하는 과정에서 다음과 같은 문제가 발생했습니다. 
```
[=   ] - info Generating static pages (3/5)
Error occurred prerendering page "/". Read more: https://nextjs.org/docs/messages/prerender-error
Error: useLocalStorage is a client-only hook
    at getLocalStorageServerSnapshot (C:\Users\js953\Desktop\Github\mbtmi\client\.next\server\chunks\824.js:12804:9)
    at Object.useSyncExternalStore (C:\Users\js953\Desktop\Github\mbtmi\client\node_modules\next\dist\compiled\react-dom\cjs\react-dom-server.edge.production.min.js:110:195)
    at exports.useSyncExternalStore (C:\Users\js953\Desktop\Github\mbtmi\client\node_modules\next\dist\compiled\react\cjs\react.production.min.js:29:469)
    at useLocalStorage (C:\Users\js953\Desktop\Github\mbtmi\client\.next\server\chunks\824.js:12810:52)
    at useRedirectIfKeyExists (C:\Users\js953\Desktop\Github\mbtmi\client\.next\server\app\page.js:632:56)
    at Home (C:\Users\js953\Desktop\Github\mbtmi\client\.next\server\app\page.js:665:23)
    at jg (C:\Users\js953\Desktop\Github\mbtmi\client\node_modules\next\dist\compiled\react-dom\cjs\react-dom-server.edge.production.min.js:117:273)
    at Z (C:\Users\js953\Desktop\Github\mbtmi\client\node_modules\next\dist\compiled\react-dom\cjs\react-dom-server.edge.production.min.js:124:91)
    at jg (C:\Users\js953\Desktop\Github\mbtmi\client\node_modules\next\dist\compiled\react-dom\cjs\react-dom-server.edge.production.min.js:118:9)
    at jg (C:\Users\js953\Desktop\Github\mbtmi\client\node_modules\next\dist\compiled\react-dom\cjs\react-dom-server.edge.production.min.js:123:11)
```
해당 오류는 Next.js의 `Prerender Error`입니다. pre-rendering은 Next.js가 JS로 만든 페이지를 보여주기 전 미리 각 페이지의 HTML 파일을 미리 만들어서 보여주는 것을 의미합니다. 즉 라이브러리에서 제공하는 useLocalStorage를 적용하는 과정에서 에러가 발생했다는 메시지입니다. 
useLocalstorage는 ClientSide에서만 사용할 수 있는데, 정적 페이지를 생성하는 과정에 이를 Serverside에서 처리하려고하여 발생한 문제입니다. 

해당 문제 해결을 위해 mount 되었는지 확인하는 단계를 추가했습니다. 
```jsx
  const [hasMounted, setHasMounted] = useState(false);

  useEffect(() => {
    setHasMounted(true);
  }, []);

  if (hasMounted) {
    return [storedValue, setValue] as const;
  }

  return [initialValue, setValue] as const;
```

### 전체 코드
```jsx
'use client';
import { useEffect, useState } from 'react';

/**
 * @description 페이지 새로 고침을 통해 상태가 유지되도록 로컬 저장소에 동기화합니다.
 *
 * @param key 로컬 저장소에 저장될 키
 * @param initialValue 초기 값
 * @returns [storedValue, setValue] - 로컬 저장소에 저장된 값, 저장 함수
 */
function useLocalStorage<T>(key: string, initialValue: T) {
  // State to store our value
  // Pass initial state function to useState so logic is only executed once
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === 'undefined') {
      return initialValue;
    }
    try {
      const item = window.localStorage.getItem(key);

      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);

      return initialValue;
    }
  });

  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore =
        value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      if (typeof window !== 'undefined') {
        window.localStorage.setItem(key, JSON.stringify(valueToStore));
      }
    } catch (error) {
      console.error(error);
    }
  };

  const [hasMounted, setHasMounted] = useState(false);

  useEffect(() => {
    setHasMounted(true);
  }, []);

  if (hasMounted) {
    return [storedValue, setValue] as const;
  }

  return [initialValue, setValue] as const;
}

export default useLocalStorage;

```

## localstorage 사용을 배제하자
Next.js에서 로컬 스토리지 사용은 생각보다 까다로웠습니다. CSR과 SSR을 구분하여, CSR 컴포넌트에서 작업을 진행해야했습니다. 더불어 useLocalStorge hook을 활용하게 되면, 해당 hook이 mount되는 것을 기다려야 했기에, 한번더 useEffect()를 활용해서 localstorage 동기화 작업이 이루어지는 환경을 보장해주어야 했습니다.

Next.js의 가장 큰 장점은 클라이언트에서 처리해야 하는 작업을 서버에게 위임하여 SEO와 렌더링 속도를 향상기키는 것으로 생각합니다. 이러한 점에서 localstorage를 활용해야 한다면, 해당 데이터의 구조를 다시금 고민해보아야 할 것 같습니다. 

프로젝트에서 사용하는 jwt에는 민감한 정보 없이, 사용자가 해당 페이지에서의 답안 작성 여부만을 검사하기 위해 발급하는 용도입니다. 해당 로직을 client측이 아닌, 서버 측에서 하는 방법을 고민해 보았습니다. 

## Auth.js
[Auth.js](https://authjs.dev/getting-started/providers/credentials-tutorial)에서 제공하는 Credentials authentication을 활용하면, 해당 문제를 해결할 수 있을 것 같습니다. 해당 라이브러리를 통해 jwt를 관리한다면, 이를 서버측으로 위임하여 굳이 localstorage에 저장할 필요가 없을 것 같습니다. 