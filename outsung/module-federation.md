# 모듈 페데레이션 적용기

서비스에 모듈 페더레이션을 적용하기로 한 이유 및 시행착오에 대한 발표를 하겠습니다.

- 모듈 페더레이션 이란?
- 실 사용법 및 장단점
- 서비스 환경 with 적용하기로한 이유
- 모듈 페더레이션을 적용하면서 얻어갈 이득

## 모듈 페더레이션 이란?

webpack 이나 다른 번들러를 살펴보시면, 지원하는 기능중 code splitting이 있습니다.

> 코드 스플리팅은 webpack의 가장 매력적인 기능 중 하나입니다. 이 기능을 사용하여 코드를 다양한 **번들**로 분할하고, 요청에 따라 로드하거나 병렬로 **로드**할 수 있습니다. 더 작은 번들을 만들고 리소스 우선순위를 올바르게 제어하기 위해서 사용하며, 잘 활용하면 로드 시간에 큰 영향을 끼칠 수 있습니다.

간단히 요약하면 모든 코드를 포함 시키지 않고, 해당 코드를 사용할때 로드 할 수 있는 기능입니다.

초기 로드 성능을 증가 시킬 수 있습니다.

---

module federation은 code splitting과 조금 비슷하다고 할 수 있습니다, 더 나아가 더 많은 제어, 확장성, 유연함을 가집니다.

module federation은
하나의 **서비스**를 독립적인 **모듈**로 **분리**하고 **배포**하고. 이후 해당 모듈을 사용할때, **로드**해서 포함시키는 방식 입니다.


예를 들면 자신의 서비스에서 나온 번들을 로드하는것 뿐만 아니라, 다른 서비스에서 빌드된 번들을 로드하는 것도 가능합니다.

독립적인 서로 다른 두 서비스가 있고, 그 두개의 서비스를 하나의 서비스로 합치는 것도 가능합니다.

다양한 번들러가 module federation을 지원합니다만, 이번 서비스에서는 webpack과 vite를 사용하기로 했습니다.

## 실 사용법 및 장단점

해당 테스트 레포 입니다.
https://github.com/outsung/vite-module-federation

a, b, c 라는 서비스가 있습니다.

a에서 b가 사용되고, b에서 c가 사용된다고 가정합니다.

webpack 설정 파일에 있는 plugin 부분에 `ModuleFederationPlugin`을 추가로 넣어주면 됩니다.


c 먼저 설명드리겠습니다.
```typescript
federation({
    name: "c",
    filename: "remoteEntry.js",
    exposes: {
        "./App": "./src/App.tsx",
    },
    shared: ["react"],
}),
```
---
b 입니다.
```typescript
federation({
    name: "b",
    filename: "remoteEntry.js",
    exposes: {
        "./App": "./src/App.tsx",
    },
    remotes: {
        c: "http://localhost:3002/assets/remoteEntry.js",
    },
    shared: ["react"],
}),
```

위 처럼 설정 해주시면
아래 처럼 실제 컴포넌트, 함수를 사용 할 수 있습니다.

```typescript
import App from 'c/App';
// or
const App = React.lazy(() => import('c/App'));
```

---
마지막으로 a 입니다.
```typescript
federation({
    name: "a",
    filename: "remoteEntry.js",
    remotes: {
        b: "http://localhost:3001/assets/remoteEntry.js",
    },
    shared: ["react"],
}),
```


---

이제 빌드를 해보면, 

exposes가 있는 b, c 빌드 파일들
<img width="384" alt="사진 1" src="https://github.com/outsung/frontend-article-study/assets/40460655/16d3f5c3-b07c-45b5-991d-c424be8b1bf7">

exposes가 없는 a 빌드 파일들
<img width="386" alt="사진 2" src="https://github.com/outsung/frontend-article-study/assets/40460655/1ea53826-363a-45cd-8054-91cb592870bc">


이렇게 번들이 나옵니다.


vite preview를 통해서 해당 파일을 서빙 해보겠습니다.

a의 config 파일을 보시면
b는 `http://localhost:3001/assets/remoteEntry.js` 이걸 가져와야한다.
라고 명시해두었죠

실제로 b 서비스에서 `vite preview` 하고 `http://localhost:3001/assets/remoteEntry.js` 해당 주소로 접근 해보면, 

<img width="1680" alt="사진 3" src="https://github.com/outsung/frontend-article-study/assets/40460655/2ff32b90-3ffb-49dc-bd04-cfe80afbe213">

이렇게 보입니다.

그럼 `http://localhost:3000/` 로 접근해서 실제 서비스(a)를 보면,

<img width="1501" alt="사진 4" src="https://github.com/outsung/frontend-article-study/assets/40460655/a3a4e56d-67f2-4384-834f-1e09790b57a3">

이렇게 순차적으로 번들을 가져옵니다.

- `index-*.js`: 메인 번들 입니다.
    실제 가장 먼저 로드 되어야 하는 js 파일입니다.
    a가 호스트앱 (가장 먼저 로드가 되는) 이기 때문에 a의 메인 번들을 사용합니다.
    실제로 b, c도 메인 번들이 있습니다. b, c 를 독립적인 서비스로 배포를 따로 할 때는 b, c 의 메인 번들이 사용 될 것입니다.
- `remoteEntry.js`: 컨테이너 번들.
    다른 서비스(a)에서 해당 서비스(b)를 사용할때 해당 서비스(b)의 컨테이너 번들을 먼저 로드 해야합니다.
- `__federation_*.js`: 청크 번들.
    컨테이너 번들을 먼저 로드 한 후, 나머지 청크 번들을 가져오는 식입니다.



```typescript
const App = React.lazy(() => import('c/App'));
```

이렇게 사용하긴 했지만 `Suspense`를 사용하지 않아서 실제로는 로딩이 오래 걸리고, 모든 a -> b -> c 까지 번들이 모두 로드 된 이후 화면이 보입니다.

<img width="1497" alt="사진 5" src="https://github.com/outsung/frontend-article-study/assets/40460655/fe294f56-f063-4076-8586-0605591e70f3">

`Suspense`를 쓰면 좋습니다..

---
### 장단점

장점은 아래와 같습니다.

- 서비스 또는 기능별로 완전 독립적으로 개발할 수 있습니다.
- 초기 로드 속도 단축, 사용하지 않는 부분을 로드 하지 않을 수 있습니다.
- 전체 서비스중 일부만 변경 되었을때 빠르게 변경된 모듈만 빌드 후 배포 할 수 있습니다.


단점은 아래와 같습니다.
- 서비스 자체가 복잡해집니다.
- 레퍼런스가 많이 없습니다.
- webpack, vite 등 해당 번들러에 대한 러닝커브가 있습니다.



## 서비스 환경 with 적용하기로한 이유

제공 되어있는 서비스는 대략

모바일
- A 협회 (공지, 채팅, 설문, 전자명함)
- B 협회 (공지, 설문, 행사, 경조사, 전자명함-b)

데스크탑
- A 협회 (공지, 채팅, 설문, 전자명함)

웹
- A 관리자 페이지 (공지, 채팅, 설문, 전자명함)
- B 관리자 페이지 (공지, 설문, 행사, 경조사, 전자명함-b)
- A 전자명함
- B 전자명함
- B 행사
- A 메신저

... 추가 협회가 들어올 수 있음.


### 요구사항

- 여러 플랫폼을 공통으로 관리 (web, app, desktop)
- 공통 기능 사용, 개발 가능
    - 배포 할때 각 기능 별로 배포, 적용 하고 싶음
- 일부 기능이 독립적인 서비스로 배포 가능 (전자명함, 채팅, 행사)
---
- 이후 협회 요구 사항이 외부 서비스 (vue)가 포함 되어야함

실제 서비스에 적용 하려면 여러 요구사항이 실제 동작하는지 검증이 필요합니다.

### 검증 필요
- 버저닝 가능?
- 각 서비스가 다른 버전을 바라볼 수 있는가?
- 개발 환경에서도 완벽하게 동작하는가?
- 어느 서비스는 개발, 어느 서비스는 production으로 가능?
- 번들을 서빙하는 번들 업데이트 서버 제작 가능?
- 외부 서비스를 사용할 때 타입은 어떻게 처리할 건가?
- 데스크탑에서도 가능?
- React Native 에서도 가능?
- Expo에서도 가능한가?
- code-push (expo update)와 사용 가능한가?
- electron update와 사용 가능한가?

대략적으로 확인 후 가능하다고 생각되어서 도입하고 있습니다.

## 모듈 페더레이션을 적용하면서 얻어갈 이득

- 외부 독립적인 서비스도 포함 가능
- 기능 단위로 개발, 테스트, 배포 가능


## 레퍼런스
https://fe-developers.kakaoent.com/2022/220623-webpack-module-federation/
https://www.youtube.com/watch?v=-jYSGaPAEHE
https://maxkim-j.github.io/posts/module-federation-concepts/
https://maxkim-j.github.io/posts/module-federation-shared/
https://blog.gangnamunni.com/post/saas-microfrontends/
