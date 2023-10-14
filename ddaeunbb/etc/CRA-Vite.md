리액트 공식문서에서도 CRA에서 Vite를 권장하고 있습니다.
Vite가 무엇이길래 보일러플레이트를 줄여주던 CRA 대신 권장하고 있을까요?
먼저 Vite가 등장하기 전까지의 번들러의 역사에 대해 알아야하는데요, 등장순서대로 포스팅해고자합니다.

> 이 포스팅은 모듈의 역사를 알고있다는 가정 하에 기술하였습니다.

![231002-115635](https://img1.daumcdn.net/thumb/R800x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcbLzfb%2FbtrB0EteFV3%2FnKWJ22UY4F312LgeymJL50%2Fimg.png)

---

## 번들러

먼저 번들러란 **여러개의 모듈들을 하나의 파일로 묶어주는 도구**를 말합니다.

원래 태초에 브라우저에서는 `ES Modules`을 지원하지 않았습니다.
즉, 공식적으로 지원해주는 JavaScript 모듈화가 없었습니다.

다른 이야기로는 `ES6 import` 문법을 사용할 수 없었고, `<script>`간 모듈을 내보낼 수 없다는 것입니다.

예전에는 자바스크립트 파일을 많이 사용하지 않았기 때문에 `HTML`에 하나씩 주입하는게 어렵지 않았지만
점점 사용자의 요구가 커지고, 서비스의 규모가 커지게 되면서 많은 `<script>` 주입이 필요해졌습니다.

그에 따라 모듈간의 의존성, 모듈 내에서 사용 하지 않는 코드 제거, 최적화, 모듈 경량화 등
이런 부분들을 일일이 개발자가 해결해야했습니다. 그러다보니 전역 변수를 해치거나 하는 많은 오류들을 마주하게 됩니다.
서비스가 커지면 현실적으로 불가능했기 때문에 개발자들은 당연하게 번들러를 사용하게 됩니다.

개발자들은 공식적인 모듈화방법이 없으니 번들링(bundling)이라는 우회적인 방법을 사용할 수 밖에 없었습니다.
> 번들링: 모듈화된 소스 코드를 브라우저에서 실행할 수 있는 파일로 한데 묶어 연결해주는 작업

따라서 번들러의 필요성이 대두된 것입니다.
이런 상황 속에 등장한 것이 `Webpack`입니다.

---

## Webpack
![231002-122926](https://images.velog.io/images/kysung95/post/f516d657-e17c-4042-bf9f-4d1e1e53ef12/header.jpeg)

webpack의 정체성은 번들러의 목적인 **통합**에 있습니다.
**'번들을 통합해서 한번에 관리할 순 없을까?'** 에 대한 고민 덕에 webpack이 부상하게 되었습니다.
(공식적인 모듈방식이 없어서 한번에 처리하는 방법을 찾는 불쌍한 개발자들...)

### 설명이 간편적이고 직관적
하나의 설정 파일에서 원하는 번들이 생성 될 수 있도록 컨트롤할 수 있습니다.
`entry`와 `output`을 명시하고 어떤 `plugin`과 `loader`로 파일들을 다룰 건지 명시하면 됩니다.

```javascript
const path = require('path');
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  entry: {
    home: './pages/home.js',
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].bundle.js',
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: ["babel-loader"],
      },
    ],
  },
  plugins: [new HtmlWebpackPlugin()],
};
```

### plugins와 loaders 생태계

webpack은 굉장히 강력한 개발 커뮤니티가 구성되어있습니다.
쉽게 plugin과 loader를 부착할 수 있기 때문인데요. loader를 통해서 파일들을 변환, 번들링, 빌드를 진행하고,
plugin을 통해서 output 파일을 튜닝해줍니다.
오랜 커뮤니티를 가지고 있음에 따라 plugin, loader가 오픈소스로 개발되고 있다는 장점이 있습니다.

### 강력한 개발서버
webpack은 Hot Module Replacement(HMR)를 처음으로 제안했고 2012년부터 사용되어왔습니다.
HMR는 소스코드의 변화를 감지하여 브라우저를 직접 새로고침할 필요가 없이 변화를 바로 반영해줍니다.
개발자는 덕분에 더욱 신속하게 개발할 수가 있게 되었습니다.
webpack v2, v3, v4을 거쳐서 다양한 파일(css)의 변화도 감지할 수 있게 되었고 안정화되었습니다.

### Code Splitting
한마디로 설명하자면 번들 로드 최적화하는 작업입니다.
파일들을 여러 번들 파일으로 분리하여 병렬로 스크립트를 로드하여 페이지 로딩속도를 개선할 수 있습니다.
추가로 초기에 구동될 필요가 없는 코드를 분리하여 lazy loading을 통해 페이지 초기 로딩속도를 개선할 수 있습니다.

### 단점

webpack은 `node.js`기반으로 돌아가기 때문에(Common JS) 싱글 스레드 구조를 갖고 있습니다. 따라서 빌드가 느리다는 단점이 있습니다. (blocking 방식이기때문에)
또한 개발서버가 느리다는 단점도 존재합니다.

![231002-151729](https://d585tldpucybw.cloudfront.net/sfimages/default-source/blogs/2022/2022-01/bundle-based-dev-server.png?sfvrsn=251b2cac_2)

소스 코드와 모든 종속 관계의 모듈을 번들링 한 후 서버가 준비됩니다.
따라서 변경점이 있을 시, 변경점만 찾아 번들링하는 것이 아니라 처음부터 다시 번들링을 하기 때문에 개발서버가 느립니다.
이런 단점은 webpack v5 이후부터 캐싱처리로 개선이 되긴했습니다.

---

## Rollup
![231002-123023](https://miro.medium.com/v2/resize:fit:1200/1*fpiQCpRdCMy3gx5lMBxzFA.png)
webpack 시대 개막과 함께 2017년부터 개발이 시작된 module bundler입니다.
대세적으론 webpack의 열풍에 가려져 있었지만 점차 그의 매력이 인정을 받아 많은 차세대 번들러가 rollup을 벤치마킹하게 되었습니다.

### vs webpack
한마디로 정리하면,
webpack은 내부적으로 Commonjs를 사용하고 rollup은 typescript(ESM)를 사용합니다.
이로 인해서 아래 특성들이 있다고 볼 수 있습니다.

rollup은 `ESM` 모듈 형태로 빌드할 수 있습니다.
webpack은 `CommonJS` 형태로만 번들링이 가능했습니다. 물론 webpack v5부턴 `ESM`으로 번들할 수 있습니다.

**rollup은 좀 더 빠르다.**
webpack은 모든 모듈들을 함수로 감싸서 평가하는 방식을 사용하지만(IIFE) rollup은 모듈들을 차례대로 호이스팅하여 한번에 평가하기에 성능상 이점이 있습니다.

**rollup은 더 가벼운 번들을 생성한다.**

브라우저 환경에서는 페이지 렌더링을 빠르게 하는 것이 중요한데, 이때 JavaScript는 로딩되어 실행되는동안 페이지 렌더링을 중단시키는 리소스들중 하나입니다. 

따라서 JS 번들 사이즈를 줄여서 렌더링이 중단되는 시간을 최소화하는 것이 중요합니다.
이를 위해 필요한것이 `Tree Shaking`입니다. (사용하지 않는 코드 제거 및 코드 최적화)
`CommonJS`는 `Tree-shaking`이 어렵고 ESM은 쉽게 가능합니다.

그 이유로는 단순히 레퍼런스되지 않는 코드를 제거하는 것이 아니라,
사용되는 모듈만 AST 트리에 포함시키는 방식으로 불필요한 코드를 제거하기 때문입니다.
rollup은 공식 플러그인을 통해서 `CommonJS` 코드를 `ESM` 코드로 변환할 수도 있습니다.

**rollup은 CommonJS 코드를 ESM코드에서 사용할 수 있습니다.**
기본적으로 ESM에서는 CommonJS식의 require를 지원하지 않습니다.
webpack에서도 공식적으론 ESM 혹은 CommonJS 한 형태의 코드 베이스를 사용하기를 권장합니다.

> 대체로 라이브러리는 `ESM` 번들에 생성에 대한 수요가 강합니다.
`ESM` 환경에서는 `ESM` 번들이 사용되고 `CommonJS` 환경에서는 `CommonJS` 번들이 사용되도록 해줘야 라이브러리 사용자는 더욱 최적화된 애플리케이션을 빌드해줄 수 있습니다.

---

## ESbuild

![231002-131240](/posts/CRA-Vite/231002-131240.png)

앞서 봐왔던 번들러는 모두 내부적으로 JavaScript을 기반으로 번들링을 합니다.
따라서 JavaScript 언어가 가질 수 밖에 없는 성능상의 한계가 있습니다.(단일 스레드)
그 한계를 부수고자 나타난 것이 `ESBuild`입니다.

`ESBuild`는 `Go`라는 저급언어 기반으로 작성된 빌드툴입니다.
JS 기반의 번들러보다 10배에서 100배까지 빠른 엄청난 퍼포먼스를 보여줍니다.

- 네이티브 코드 방식 사용 (ESM)
- 병렬처리 최적화
- 메모리 사용 최적화
- 자체 JavaScript 파서 사용

### 단점
그러나 HMR(Hot-Module Replacement)를 공식적으로 지원하지 않아 어려움이 있었고
이외에 Babel 트랜스 파일 지원 미흡 등의 단점이 있었습니다
code splitting 및 CSS와 관련된 처리가 아직 미비합니다. ESBuild는 ES5 이하의 문법을 아직 100% 지원하지 않습니다.


---

## Vite

HMR을 지원하지 않는 ESbuild의 단점과 느린 Webpack의 개발서버를 보완하고,
각각의 장점을 섞어 만든게 Vite라고 생각합니다.

앞서 이야기한 바와 같이 브라우저에서 ESM(ES Modules)을 지원하기 전까지,
JavaScript 모듈화를 네이티브 레벨에서 진행할 수 없었습니다.
그래서 소스 모듈을 브라우저에서 실행할 수 있는 파일로 크롤링, 처리 및 연결하는 "번들링(Bundling)"이라는 해결 방법을 사용해야 했습니다. 그래서 Webpack, Rollup 등 많은 번들러들이 등장하게 되었는데요.

이러한 상황에서 JavaScript 기반의 도구는 성능의 한계가 있었고 종종 개발 서버를 가동하는 데 비합리적으로 오랜 시간을 기다려야 한다거나 HMR을 사용하더라도 변경된 파일이 적용될 때 까지 수 초 이상 소요되곤 했습니다.

따라서 Vite는 `ESBuild`로 미리 번들링한 모듈을 필요할 때 동적으로 가져오기 때문에 즉각적으로 개발서버가 구동됩니다.
![231002-155116](https://velog.velcdn.com/images/eamon3481/post/acd6e4ef-3cda-4b6a-98d9-df917c5e2f99/image.png)

개발 모드로 빌드를 할 때, Vite는 자바스크립트 모듈을 두 가지 카테고리로 나눕니다.
- 의존성 모듈
- 소스 코드

의존성 모듈은 `node_modules` 폴더로부터 import 되는 자바스크립트 모듈이며 개발하는 동안엔 자주 바뀌지 않는 편입니다. 이 모듈은 `ESBuild`를 사용하여 처리됩니다.
Webpack이 브라우저의 요청 이전에 모든 자바스크립트 모듈을 처리하는 반면, Vite는 브라우저 요청 전에 의존성 모듈만 미리 번들링합니다.

소스코드는 .jsx, .vue, .scss와 같은 라이브러리 관련 확장자를 포함할 수도 있으며, 자주 수정되는 파일입니다.
또한 소스코드 전체는 동시에 로드될 필요가 없다는 특징이 있습니다.

Vite는 `native ESM`을 통해 소스코드를 제공합니다.
Vite는 브라우저의 요청이 있을 시에만 소스코드를 변환하고 제공합니다.

이전에 Webpack에서는 소스 코드가 변경되면 전체적으로 번들링 과정을 다시 거쳐야하다 보니 서비스가 거대해질수록 빌드시간이 늘어나게 되었습니다. 따라서 프로젝트의 크기가 커질 수록, HMR의 시간 또한 길어지게 되므로 이는 생산성 저하로 이어지게 되는 문제가 있었습니다.

Vite는 네이티브 `ESM`을 기반으로 HMR을 구현하므로 다른 번들링 툴보다 우위에 있다고 할 수 있습니다.
모듈이 수정되면, Vite는 수정된 모듈과 관련된 부분만을 교체할 뿐이고,
브라우저에서 해당 모듈을 요청하면 교체된 모듈을 전달할 뿐입니다.
이는 어플리케이션의 크기와 관계 없이 HMR을 언제든 빠르게 이뤄지도록 해줍니다.

### Webpack vs Vite 비교
- **개발 서버**
  - Webpack: 소스 코드와 모든 종속 관계의 모듈을 번들링 한 후 서버가 준비됩니다.
  - Vite: ESBuild로 미리 번들링한 모듈을 필요할 때 동적으로 가져오기 때문에 즉각적으로 서버가 구동됩니다.

- **프로덕션 빌드**
  - Webpack: 각 모듈을 `범위마다 함수로 맵핑`하여 결합합니다. (변수 충돌을 위해 모듈마다 스코프 생성)
  - Vite: `하나의 파일에 모든 종속 모듈`을 전역 범위로 선언하여 결합합니다. 중복을 제거하기 때문에 가볍고 빠르게 빌드할 수 있습니다.

### 주의 해야할 점
vites는 기본적으로 `ESM`을 타겟으로 번들을 생성합니다. 따라서 ES5이하로 타겟을 잡으려면 별도로 polyfill를 다뤄야 합니다. 그래도 공식적으로 @vitejs/plugin-legacy 플러그인을 제공하고 있습니다.

Vite는 번들링 속도에서 강점이 있지만, 마이그레이션과 안정성 측면에서의 문제가 거론되고 있습니다.
Vite는 개발 환경에서 `ESbuild`를 사용하지만 프로덕션 환경에서는 `Rollup`으로 번들링을 수행합니다.
따라서 Webpack에서 Vite로 마이그레이션을 한다면 `Rollup config`를 설정하는 것에 많은 시간을 투자해야합니다.
![231002-160818](/posts/CRA-Vite/231002-160818.png)

Rollup 설정에는 CommonJS 처리, Polyfill 처리 등이 되어있지 않기 때문에 일일이 플러그인을 설치해야해서 번거롭습니다. 하지만 설정을 마치게 되면 정작 프로덕션 환경에서는 Webpack과 빌드 시간에서 차이가 크게 나지 않는다고 합니다.
게다가 개발 환경과 프로덕션 환경의 설정이 다르기 때문에 빌드 안정성이 낮아서 개발 환경에서만 사용한다는 의견도 많습니다.



## Reference Doc
[왜 Create React App 대신 Vite일까](https://velog.io/@jaewoneee/%EB%A6%AC%EC%95%A1%ED%8A%B8-%EB%B3%B4%EC%9D%BC%EB%9F%AC%ED%94%8C%EB%A0%88%EC%9D%B4%ED%8A%B8-Create-React-App-vs-Vite)
[CommonJS와 ESM에 모두 대응하는 라이브러리 개발하기](https://toss.tech/article/commonjs-esm-exports-field)
[이제는 Vite로 개발을 해보자](https://myhappyman.tistory.com/316)
[모듈 번들러란? (Webpack vs Vite 무엇을 써야 할까요?)](https://www.enjoydev.life/blog/frontend/4-module-bundler#webpack-vs-vite)
[webpack과 vite](https://hmk1022.tistory.com/entry/webpack%EA%B3%BC-vite)
[vite 공식문서](https://ko.vitejs.dev/guide/why.html)