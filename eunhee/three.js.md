# three.js 정의와 기본구조 👷‍♀️
## Three.js
Three.js란 웹에 3D객체를 쉽게 렌더링하도록 도와주는 <span style="color:red">자바스크립트 3D 라이브러리</span> 입니다.  WebGL과 비교한다면 WebGL은 2D/3D객체를 렌더링하는 일을 하고, Three.js은 어려운 WebGL을  3D 그래픽에 관련된 수학 지식들이 없이 직관적인 코드를 짤 수 있도록 도와주는 3D 라이브러리라고 할 수 있겠네요!

* SVG와 CSS를 통해 3D를 만들 수 있지만 매우 제한적이고 어렵습니다.

||종류|하는 일|언어|
|------|---|---|---|
|WebGL|레스터화 엔진|웹에서 2차원 그래픽과 인터랙티브 3차원 그래픽을 렌더링|js|
|Three.js|3D 라이브러리	|수학지식없이  WebGL을 사용하도록 도와주는 라이브러리|자바스크립트|

OpenGL 과 WebGL

OpenGL은 Open Graphics Library이며, WebGL의 원조라고 알아둡시다. C언어가 기반인 OpenGL을 웹상에서도 사용할 수 있도록 나온 것이 WebGL입니다.

## Three.js의 기본 구조.
Three.js 에서 가장 기본이 되는 것은 3 가지.<span style="color:blue"> Renderer, Scene, Camera </span> 입니다. (당연하게도 3ds MAX 와 기본 개념이 비슷하네요.)

1. Scene이라는 **3D 공간**을 꾸미고,
2. 그 공간을 Camera라는 **시점**에서 바라보는 것.
3.  그 시점을 Renderer를 통해 HTML Canvas안에 **렌더링**하여 보여주는 것. 
이게 전부입니다. 
