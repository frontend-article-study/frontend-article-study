## Refresh Token

Refresh Token은 Access Token을 재발급할 때 사용하는 키이다. Access Token이 긴 만료 시간을 가지게 되면, 탈취당하여 악의적인 공격에 사용될 수 있다. 따라서 Access Token의 만료 기간을 짧게 유지하고, 상대적으로 긴 만료 시간을 가지는 Refresh Token을 통해 Access Token을 재발급 받음으로서 사용자가 로그아웃 없이 로그인 상태를 유지할 수 있게 한다. 

### 일반적인 경우
![](https://velog.velcdn.com/images/chchaeun/post/e59f1dcd-a972-4437-a58f-0f322a02c8a5/image.png)
일반적인 경우 Refresh Token을 통한 인증/인가 플로우는 위와 같이 동작한다.

Access Token과 Refresh Token은 Cookie, LocalStorage 등의 브라우저의 저장 공간에 저장된다. 일반적으로 Access Token은 30분~1시간, Refresh Token은 2주 정도의 유효 기간을 가진다. 

### 취약점

Access Token의 탈취로 인한 공격을 막기 위해 Refresh Token을 사용하지만, 브라우저에 확실히 의도된 앱만 접근할 수 있는 저장 매커니즘이 없기 때문에 Refresh Token 역시 탈취 가능성이 있다. 만약 Refresh Token이 유출된다면, 공격자는 이것으로 얼마든지 Access Token을 발급받아 유저 정보에 접속할 수 있다.

## Refresh Token Rotation

Refresh Token Rotation(RTR)은 Access Token이 만료될 때마다 Refresh Token도 함께 교체를 해주는 것이다. 


![](https://velog.velcdn.com/images/chchaeun/post/e25c3cc8-4e97-4e4f-a594-2186729f0490/image.png)

위 그림은 PKCE를 통한 인증에서 RTR이 사용되는 방식을 나타낸 것이다. 하지만, 보편적인 원리는 PKCE가 아닌 다른 인증 플로우에서도 적용될 수 있다.

1. Code Verifier와 Code Challenge을 생성한다. 
2. 서버에 Authorization 요청을 보낸다.
3. User임이 확인되면 Authorization Code를 클라이언트에 return한다.
4. 클라이언트가 서버에 Authorization Code를 Access Token으로 교환한다(OAuth를 사용하지 않는 경우, 이 부분에서 클라이언트가 아이디와 비밀번호를 서버에 보내며 플로우가 시작된다).
5. 서버는 AT(1)와 RT(1)를 반환한다.
6. AT(1)이 만료되면, 클라이언트가 새 AT(2)를 얻기 위해 RT(1)를 전달한다.
7. 서버는 새 AT(2)와 새 RT(2) 반환하고, RT(1)는 만료된다.


### 자동 재사용 감지
클라이언트가 새 Access Token을 발급받을 때, Refresh Token과 함께 요청을 보낸다. 새로운 토큰이 발행되면 요청에 사용된 Refresh Token은 무효화된다. 이미 사용된 토큰으로 다시 Access Token을 요청하는 Replay Attack 상황으로부터 앱을 보호하는 것이다.

>**Replay Attack**
공격자가 원본 메시지와 같은 자격증명을 얻기 위해 이전에 보내진 메시지를 가로채서 다시 보내는 것

발신자를 제약하는 조건 없이는 Replay Attack 상황에서 서버가 액터가 정당하거나 악의적임을 아는 것이 불가능하다. 따라서, 이전에 사용된(무효화 된) Refresh Token이 인증 서버로 전송되면 가장 최근 발행된 Refresh Token도 즉시 무효화되어야 한다. 같은 Token Family에 속한 어떤 Refresh Token도 새로운 Access Token을 얻을 수 없도록 하는 것이다.

> **Token Family**
클라이언트에 대해 발행된 원본 Refresh Token으로부터 교환한 모든 Refresh Token.

### 탈취 시나리오 1

![](https://velog.velcdn.com/images/chchaeun/post/7a13fd55-cbe2-4ce6-b535-67d0a44efbe3/image.png)

위 탈취 시나리오와 함께 RTR이 공격에 대응하는 방법을 살펴보자.

1. 사용자(정당한 클라이언트)가 RT(1)을 공격자(악의적인 클라이언트)에게 탈취 당했다.
2. 사용자가 RT(1)을 RT/AT 쌍을 얻기 위해 사용한다.
3. 서버는 RT(2)/AT(2)를 반환한다.
4. 공격자가 RT(1)으로 AT를 얻으려고 시도한다. 서버는 RT(1)이 재사용됨을 인지하고, 즉시 RT(2)을 포함한 RT Family를 무효화한다.
5. 서버는 공격자에게 Access Denied를 응답한다.
6. AT(2)는 만료됐고 사용자가 RT(2)를 새로운 토큰 요청에 사용해도 서버는 Access Denied를 응답한다.
7. Re-authentication이 요구된다.

핵심은 클라이언트가 정당하든, 악의적이든 하나의 Refresh Token으로는 단 한번 토큰 쌍을 교환할 수 있다는 것이다. 재사용이 감지되면 re-authenticate가 있을 때까지 후속 요청은 거부된다. 그리고 서버는 감지된 재사용 이벤트를 로그에 캡처해둔다.

### 탈취 시나리오 2
![](https://velog.velcdn.com/images/chchaeun/post/f8803a7d-402c-4045-8b3e-0365805040be/image.png)

다른 예시로는 공격자가 RT(1)을 탈취하고, 사용자보다 먼저 AT를 요청하는 시나리오가 있다. 이 경우, 공격자의 접근은 단기적일 것이다. 왜냐하면 사용자가 RT(1)을 사용하면, RT(2)(또는 그후 발생된 어떤 Refresh Token이든)가 자동적으로 취소되기 때문이다.

1. 공격자가 RT(1)을 탈취한다.
2. 공격자가 RT(1)으로 AT를 요청한다.
3. AT(2)와 RT(2)가 공격자에게 반환된다.
4. RT(1)가 무효화되고, 사용자가 RT(1)으로 AT를 요청한다.
5. 재사용이 감지되어 RT Family가 무효화 된다. 서버는 사용자에게 Access Denied 응답한다.
6. 공격자가 RT(2)로 AT를 요청한다.
7. 모든 RT가 무효화되었으므로 서버는 Access Denied를 응답한다.

이 경우 Access Token의 유효기간을 최대한 짧게 유지하는 것이 중요하다. Access Token이 만료되어야 사용자가 다시 Refresh Token을 사용할 것이기 때문이다.

## Access Token은 어디에 저장할까
유효기간을 짧게 유지하고, 탈취를 막기 위해 Access Token을 브라우저 저장공간이 아닌 자바스크립트 내부 변수에 저장하고 새로고침 시 Refresh Token을 통해 재발급 받도록 하고 싶다.

예상되는 문제는 서버에 Access Token 재발급 요청 및 Refresh Token 무효화가 너무 많이 발생해서 부하가 일어날 수 있다는 것이다. 

이때 애플리케이션 서버와 인증 서버를 분리하고, 인증 서버를 Redis 등의 인메모리 저장소로 둔다면 부하를 방지할 수 있을 것으로 생각된다.

## 구현
처음에는 아래 코드처럼 기존 상태 관리 라이브러리인 `zustand`로 구현했다.
`accessToken`과 `expirationTime`을 저장하고, `getAccessToken` 메서드로 RTR 과정 처리를 구현했다.

```typescript
import axios from "axios";
import create from "zustand";
import { deleteRefreshToken, getRefreshToken, setRefreshToken } from "./utils";

interface AccessTokenInfo {
  accessToken: string;
  expirationTime: number;
  setAccessToken: (accessToken: string, expirationTime: number) => void;
  getAccessToken: () => void;
  deleteTokens: () => void;
}

const useAccessToken = create<AccessTokenInfo>((set, get) => ({
  // ...
}));

export { useAccessToken };

```

하지만 API 요청 코드 내부에서 RTR 코드를 호출해야 됐기 때문에, Hook은 최상단에서 호출돼야 한다는 원칙에 따라 해당 구조 사용이 불가했다. 

따라서 Hook 형태의 라이브러리가 아닌 함수를 구현했다. 

`accessToken`을 은닉하기 위해 클로저를 사용했고, 사용시에는 `getAccessToken` 메서드를 호출해서 `accessToken`을 가져오는 것이다.


```typescript

const refreshTokenRotation = () => {
  let accessToken: string = null;
  let expirationTime: number = null;

  return {
    getAccessToken: async () => {
      const refreshToken = getRefreshToken();
      if (!accessToken || expirationTime < new Date().getTime()) {
        await axios
          .post("/api/refresh", {
            refreshToken,
          })
          .then((res) => {
            accessToken = res.data.accessToken;
            expirationTime = res.data.expirationTime;
            setRefreshToken(res.data.refreshToken);
          })
          .catch(() => {
            accessToken = null;
            expirationTime = null;
            alert("로그인이 만료 되었습니다.");
          });
      }
      return accessToken;
    },
    setAccessToken: (at: string, et: number) => {
      accessToken = at;
      expirationTime = et;
    },
    deleteTokens() {
      accessToken = null;
      expirationTime = null;
      deleteRefreshToken();
    },
  };
};
```

하지만 위 코드의 문제는 `getAccessToken`이 Promise를 반환하기 때문에, `accessToken`을 API 호출부에서 사용하기 위해서는 토큰을 가져오는 코드를 작성해주어야 한다는 것이다. 

```typescript
function Example() {
  	const [token, setToken] = useState();
	useEffect(async () => {
      await getAccessToken()
      	.then((res) => {
      		setToken(res.accessToken);
      	})
      await axios.get('/example', {
		headers: {
        	Authorization: `Bearer ${token}`
        }
      })
    }, [])
}
```

매번 이런 구조를 작성하기 번거롭기 때문에, axios interceptor를 통해 이 부분을 해결했다.

`api`가 호출되면 먼저 `setAuthHeader`가 실행되어, Access Token이 없거나 만료된 경우 요청을 통해 세팅해준다. 이후 config.header.Authorization에 토큰을 넣어준 뒤 config를 리턴해주면 Access Token이 헤더에 입력된 상태로 API가 호출된다. 

`setAuthHeader`를 제외한 함수들은 외부에서 사용되지 않기 때문에 변수들과 함께 은닉해주었다.

```typescript
import axios, { AxiosRequestConfig } from "axios";
import { deleteRefreshToken, getRefreshToken, setRefreshToken } from "./utils";

const api = axios.create();

const refreshTokenRotation = () => {
  let accessToken: string = null;
  let expirationTime: number = null;

  const setAccessToken = (at: string, et: number) => {
    accessToken = at;
    expirationTime = et;
  };

  const deleteTokens = () => {
    accessToken = null;
    expirationTime = null;
    deleteRefreshToken();
  };

  const moveHome = () => {
    window.location.href = "/";
  };

  const isExpired = () => {
    const now = new Date().getTime();
    return expirationTime < now;
  };

  const getNewTokens = async function () {
    const refreshToken = getRefreshToken();
    await axios
      .post("/api/refresh", {
        refreshToken,
      })
      .then((res) => {
        const {
          data: { accessToken, expirationTime, refreshToken },
        } = res;
        setAccessToken(accessToken, expirationTime);
        setRefreshToken(refreshToken);
      })
      .catch(() => {
        deleteTokens();
        alert("로그인이 만료되었습니다.");
        moveHome();
      });
  };

  return {
    setAuthHeader: async function (
      config: AxiosRequestConfig
    ): Promise<AxiosRequestConfig> {
      if (!accessToken || isExpired()) {
        await getNewTokens();
      }

      config.headers.Authorization = `Bearer ${accessToken}`;

      return config;
    },
  };
};

const { setAuthHeader } = refreshTokenRotation();

api.interceptors.request.use(setAuthHeader);

export default api;
```

현재 서버 측에 RTR 관련 API 수정을 요청한 상태라 Next.js로 간단히 Mock 서버를 만들어 테스트해보았다. `/api/refresh`에 요청하면 새로운 랜덤 값으로 Access Token을 반환한다.
- `/pages/api/refresh.tsx`
```typescript
import type { NextApiRequest, NextApiResponse } from "next";

type ResponseData = {
  accessToken: string;
  refreshToken: string;
};

export default function handler(
  req: NextApiRequest,
  res: NextApiResponse<ResponseData>
) {
  res.status(200).json({
    accessToken: `New Access Token: ${Math.random()}`,
    refreshToken: "New Refresh Token",
  });
}
```

- `/pages/api/test.tsx`
```typescript
import axios from "axios";
import React, { useEffect } from "react";
import api from "../auth/refreshTokenRotation";

function refreshtokentest() {
  useEffect(() => {
    api.post("/api/test").then((res) => console.log(res));
  }, []);
  return <></>;
}

export default refreshtokentest;
```
![](https://velog.velcdn.com/images/chchaeun/post/8f1d978d-b8e4-4f12-b296-8f3bb5e51c32/image.png)
![](https://velog.velcdn.com/images/chchaeun/post/4fea1b60-e6e5-4dbe-8166-3d8866c95f58/image.png)


새로고침을 하면 헤더에 새로운 Access Token이 잘 들어가 있다. 서버에서 API 수정이 완료되고 연동만 하면 완료된다! 

## 마치며

이전에 Refresh Token을 통한 인가 과정을 구현하면서 Refresh Token도 브라우저 저장공간에 저장하면 결국 탈취 위험이 있는 거 아닌가? 그러면 Access Token과 Refresh Token을 굳이 따로 두었을 때 보안이 강화되긴 하는 건가? 하는 의문이 들었었다. 그래서 찾아보고 알게 됐던 내용을 이번에 프로젝트를 리팩터링하면서 탈취 시나리오에 대해서도 생각해보고, 직접 코드로 구현했다는 점이 뿌듯하다.

그리고 최근에 Text Me와 타 사이트를 연동하는 작업을 하나 제안 받았는데, OAuth 시스템을 구축해볼 수 있을 것 같다. 만약 하게 된다면 위에서 잠깐 언급한 PKCE와 RTR을 같이 구현해볼 수 있을 것 같다. 재밌는 작업일 것 같아서 빨리 해보고 싶다!!! 


>해당 포스트에 대한 의견이나 조언이 있으시면 댓글로 나누어주세요! 감사합니다.

### 도움이 된 아티클
- [Refresh Token Rotation - Auth0 by Okta](https://auth0.com/docs/secure/tokens/refresh-tokens/refresh-token-rotation#maintain-user-sessions-in-spas)
