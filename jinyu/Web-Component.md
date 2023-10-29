# 웹 컴포넌트 API

웹 컴포넌트를 알아보기 전에 컴포넌트란 무엇일까요? 컴포넌트란 여러 개의 프로그램 함수들을 모아 하나의 특정한 기능을 수행할 수 있도록 구성한 작은 기능적 단위를 말합니다. 컴포넌트는 모듈성, 재사용성, 독립성을 가지며 이러한 컴포넌트를 가지고 개발하는 방식을 컴포넌트 기반 개발(Component-Based Development, CBD) 혹은 컴포넌트 기반 아키텍처(Component-Based Architecture) 이라고 합니다.

- 모듈성: 소프트웨어는 독립적인 컴포넌트로 나뉘며, 각 컴포넌트는 특정 기능을 수행하는 모듈 역할을 한다.
- 재사용성: 컴포넌트는 다양한 애플리케이션에서 재사용될 수 있어야 한다.
- 독립성: 컴포넌트는 독립적으로 운영될 수 있으며, 다른 컴포넌트에 최대한 의존하지 않아야 한다.

MDN에서 정의하는 **웹 컴포넌트**는 "그 기능을 나머지 코드로부터 캡슐화하여 재사용 가능한 커스텀 엘리먼트를 생성하고 웹 앱에서 활용할 수 있도록 해주는 다양한 기술들의 모음"이라고 합니다.

쉽게 말해서, 리액트에서 웹 애플리케이션에 UI를 렌더링하기 위해 컴포넌트를 작성하는 것처럼 HTML 태그들의 묶음을 재사용 가능하고 캡슐화된 덩어리로 만들 수 있습니다. 이러한 덩어리를 "웹 컴포넌트"라고 하며 이러한 웹 컴포넌트를 만들 수 있는 API를 자바스크립트에서 제공합니다.

웹 컴포넌트를 구현하기 위해서는 다음과 같은 요소가 있습니다.

- Custom elements: 사용자 인터페이스에서 원하는대로 사용할 수있는 사용자 정의 요소 및 해당 동작을 정의 할 수있는 JavaScript API입니다. 또한 컴포넌트의 생명 주기도 설정해줄 수 있습니다.
- Shadow DOM: 웹 컴포넌트를 캡슐화하기 위해 사용됩니다. 실제 DOM과 Shadow DOM은 완전히 다른 영역이라는 점을 기억하는 것이 중요합니다. 실제 DOM의 요소에 대한 변경 사항은 Shadow DOM의 요소에 적용되지 않으며, 그 반대의 경우도 마찬가지입니다. 실제 DOM에서 숨겨진, 분리된 DOM이라고 생각하면 됩니다.
- HTML 템플릿: `<template>` 과 `<slot>` 엘리먼트는 렌더링된 페이지에 나타나지 않는 마크업 템플릿을 작성할 수 있게 해줍니다. 그 후, 커스텀 엘리먼트의 구조를 기반으로 여러번 재사용할 수 있습니다.

웹 컴포넌트를 구현하기 위한 기본 접근법은 일반적으로 다음과 같습니다.

1. ECMAScript 2015 클래스 문법(자세한 내용은 Classes에서 확인)을 사용해 웹 컴포넌트 기능을 명시하는 클래스를 생성합니다.
2. CustomElementRegistry.define() (en-US) 메소드를 사용해 새로운 커스텀 엘리먼트를 등록하고, 정의할 엘리먼트 이름, 기능을 명시하고 있는 클래스, (선택적으로) 상속받은 엘리먼트를 전달합니다.
3. 필요한 경우, Element.attachShadow() (en-US) 메소드를 사용해 shadow DOM 을 커스텀 엘리먼트에 추가합니다. 일반적인 DOM 메소드를 사용해 자식 엘리먼트, 이벤트 리스너 등을 shadow DOM 에 추가합니다.
4. 필요한 경우, `<template>` 과 `<slot>` 을 사용해 HTML 템플릿을 정의합니다. 다시 일반적인 DOM 메소드를 사용해 템플릿을 클론하고 shadow DOM 에 추가합니다.
5. 일반적인 HTML 엘리먼트처럼, 페이지의 원하는 어느곳이든 커스텀 엘리먼트를 사용할 수 있습니다.

### Custom elements API

1. 커스텀 요소를 만들 클래스를 생성

```js
class MyComponent extends HTMLElement {
    // 항상 super를 생성자에서 먼저 호출합니다
    super();

    // 요소 관련된 로직 작성
    ...

}

// 요소 클래스 작성을 완료했으면 customElements.define(tag, class)으로 요소를 등록합니다.
customElements.define('my-component', MyComponent);
// tag이름을 설정할 때 반드시 하이픈(-)을 넣어줘야 합니다.
```

혹은 아래와 같이 작성할수도 있습니다.

```js
customElements.define('my-compoenent', class extends HTMLElement {
    ...
});
```

작성된 컴포넌트는 다음과 같이 `html`에서 사용할 수 있습니다.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <my-component> ... </my-component>
  </body>
</html>
```

만약에 자바스크립트에서 커스텀 엘리먼트로 작성된 요소들의 정보를 얻고 싶다면 다음 메서드를 사용하면 됩니다.

- `customElements.get(name)`: 주어진 이름을 가진 사용자 정의 요소에 대한 클래스를 반환합니다.
- `customElements.whenDefined(name)`: 주어진 이름을 가진 사용자 정의 요소가 정의될 때 값 없이 resolved되는 promise를 반환합니다.

또한 커스텀 엘리먼트는 컴포넌트의 생명 주기도 관리할 수 있습니다. 자세한 내용은 [여기](https://developer.mozilla.org/ko/docs/Web/API/Web_components/Using_custom_elements#%EC%83%9D%EB%AA%85_%EC%A3%BC%EA%B8%B0_%EC%BD%9C%EB%B0%B1_%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0) 참고해주세요.

또한 특정 태그를 지정해서 사용한다면 `HTMLElement`가 아닌 해당 태그를 상속받아 사용합니다. 다음은 단어를 카운트하는 예제입니다.

```js
// 커스텀 요소를 생성하기 위한 클래스를 생성한다. 기본적으로 HTMLElement를 상속받지만 이 예제에서는 HTMLParagraphElement
class WordCount extends HTMLParagraphElement {
  constructor() {
    // 항상 super를 생성자에서 먼저 호출합니다
    super();

    // count words in element's parent element
    const wcParent = this.parentNode;

    function countWords(node) {
      const text = node.innerText || node.textContent;
      return text
        .trim()
        .split(/\s+/g)
        .filter((a) => a.trim().length > 0).length;
    }

    const count = `Words: ${countWords(wcParent)}`;

    // Create a shadow root
    const shadow = this.attachShadow({ mode: 'open' });

    // Create text node and add word count to it
    const text = document.createElement('span');
    text.textContent = count;

    // Append it to the shadow root
    shadow.appendChild(text);

    // Update count when element content changes
    setInterval(function () {
      const count = `Words: ${countWords(wcParent)}`;
      text.textContent = count;
    }, 200);
  }
}

// customElements.define(tag, class)으로 요소를 등록한다.
// HTMLElement를 상속받는 경우에는 세 번째 전달 인수를 생략해도 되지만
// 특정 태그 요소의 경우 { extends: 'p' }처럼 객체를 전달해줘야 한다.
customElements.define('word-count', WordCount, { extends: 'p' });
```

그리고 `html`에서 호출할 때 해당 태그와 함께 `is` 프로퍼티를 붙여 커스텀 엘리먼트의 이름을 값으로 넘겨줍니다. 예) `<p is="word-count"></p>`

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Simple word count web component</title>
  </head>
  <body>
    <h1>Word count rating widget</h1>

    <article contenteditable="">
      <h2>Sample heading</h2>

      <p>
        Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nunc pulvinar
        sed justo sed viverra. Aliquam ac scelerisque tellus. Vivamus porttitor
        nunc vel nibh rutrum hendrerit. Donec viverra vestibulum pretium. Mauris
        at eros vitae ante pellentesque bibendum. Etiam et blandit purus, nec
        aliquam libero. Etiam leo felis, pulvinar et diam id, sagittis pulvinar
        diam. Nunc pellentesque rutrum sapien, sed faucibus urna sodales in. Sed
        tortor nisl, egestas nec egestas luctus, faucibus vitae purus. Ut elit
        nunc, pretium eget fermentum id, accumsan et velit. Sed mattis velit
        diam, a elementum nunc facilisis sit amet.
      </p>

      <p>
        Pellentesque ornare tellus sit amet massa tincidunt congue. Morbi
        cursus, tellus vitae pulvinar dictum, dui turpis faucibus ipsum, nec
        hendrerit augue nisi et enim. Curabitur felis metus, euismod et augue
        et, luctus dignissim metus. Mauris placerat tellus id efficitur ornare.
        Cras enim urna, vestibulum vel molestie vitae, mollis vitae eros. Sed
        lacinia scelerisque diam, a varius urna iaculis ut. Nam lacinia, velit
        consequat venenatis pellentesque, leo tortor porttitor est, sit amet
        accumsan ex lectus eget ipsum. Quisque luctus, ex ac fringilla
        tincidunt, risus mauris sagittis mauris, at iaculis mauris purus eget
        neque. Donec viverra in ex sed ullamcorper. In ac nisi vel enim accumsan
        feugiat et sed augue. Donec nisl metus, sollicitudin eu tempus a,
        scelerisque sed diam.
      </p>

      <p is="word-count"></p>
    </article>

    <script src="main.js"></script>
  </body>
</html>
```

### Shadow DOM

Shadow DOM은 캡슐화의 핵심이며 필요한 경우에 사용하면 됩니다. 캡슐화를 통해 마크업 구조, 스타일, 동작을 숨기고 페이지의 다른 코드로부터 분리하고 싶은 컴포넌트에 사용하면 됩니다.

### 웹 컴포넌트의 장단점

### 장점

- React, Vue, Angular 등 어떤 프레임워크와 함께 사용
- 재사용성
- 캡슐화
- 모듈화

### 단점

- 러닝 커브
- Shadow DOM은 아직 비교적 최신 기술이기 때문에 생각보다 버그가 많음.
  - input, textarea, select 와 같은 사용자 입력은 받는 폼과 연결이 되지 않을 수 있음
  - 스크린리더 같은 앱이 Shadow DOM에 있는 요소들을 제대로 인식하지 못함
- 브라우저 호환성
  - 현재 파이어폭스, 크롬, 오페라에서는 지원이 됨
  - 사파리에서는 구현은 되어 있으나 위 브라우저들만큼은 아님
  - 엣지는 구현 중...

### 결론

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fc5HsMj%2FbtrqpthVSlA%2FYR94iI9AYM6So5cX3PCygK%2Fimg.png)

웹 컴포넌트 API는 여전히 미숙한 부분이 많은 것 같습니다. 그래도 최근 해외에서는 꾸준히 웹 컴포넌트로 개발된 애플리케이션이 증가하고 생태계도 점점 커지고 있는 추세라고 하네요. 웹 컴포넌트의 경우 모든 프레임 워크에서 사용이 가능한 장점이 있는 크로스 프레임워크 기술이 있기 때문에 기존 프레임워크를 대체할 것이라는 의견도 있다고 하지만 제 생각은 아직까지는 리액트를 사용할 것 같습니다. 하지만 하나의 프로젝트 안에 뷰, 앵귤러, 리액트 등 다양한 프레임워크를 사용하는 경우가 있다면(?) 웹 컴포넌트로 UI를 구성하면 서로 공유를 할 수 있기 때문에 유용하게 쓰일 것 같네요. 하지만 리액트, 뷰와 같은 프레임워크도 (리액트는 라이브러이지만...) 현재까지는 유지 보수가 잘되고 있을지 몰라도 10년 뒤에도 사용할 것이라는 보장은 못할 것 같아요. 구글이 자사의 웹 컴포넌트 프레임워크인 '폴리머'를 밀고 있는 것도 그렇고 꾸준히 발전한다면 기존 프레임워크들을 대체할 수 있을 것 같긴 합니다.

### 레퍼런스

[웹 컴포넌트 라이브러리]

- [webcomponents.org](https://www.webcomponents.org/) — 웹 컴포넌트 예제, 튜토리얼 등의 정보를 포함하는 사이트.
- [Hybrids](https://github.com/hybridsjs/hybrids) — 오픈 소스 웹 컴포넌트 라이브러리. class 와 this 문법보다 일반 객체와 순수 함수를 선호합니다. 커스텀 엘리먼트 생성을 위한 간단한 함수형 API 를 제공합니다.
- [Polymer](https://polymer-library.polymer-project.org/3.0/docs/devguide/feature-overview) — 구글의 웹 컴포넌트 프레임워크 — 폴리필, 확장 및 예제의 집합. 현재 크로스 브라우징 웹 컴포넌트를 사용하는 가장 쉬운 방법.
- [Snuggsi.es](https://github.com/devpunks/snuggsi#readme) — 폴리필 포함 ~1kB 크기의 쉬운 웹 컴포넌트 — 브라우저와 HTML, CSS, JavaScript 클래스에 대한 기본 이해만으로 생산성을 높일 수 있습니다.
- [Slim.js](https://github.com/slimjs/slim.js) — 오픈 소스 웹 컴포넌트 라이브러리 — 빠르고 쉬운 컴포넌트 작성을 위한 고성능 라이브러리; 확장 가능하며 플러거블하고 크로스 프레임워크와 호환됩니다.

[참고사이트]

- [MDN - 웹 컴포넌트](https://developer.mozilla.org/ko/docs/Web/API/Web_components)
- [MDN - shadow DOM 사용하기](https://developer.mozilla.org/ko/docs/Web/API/Web_components/Using_shadow_DOM)
- [MDN - 다양한 예제](https://github.com/mdn/web-components-examples/blob/main/expanding-list-web-component/main.js)
- https://yozm.wishket.com/magazine/detail/2217/
- https://itchallenger.tistory.com/933
- https://daverupert.com/2023/07/why-not-webcomponents/
- https://nuli.navercorp.com/community/article/1133158
