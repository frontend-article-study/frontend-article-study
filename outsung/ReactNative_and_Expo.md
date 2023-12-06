# React Native 그리고 Expo

## React Native

React Native는 React를 사용해서
android, ios의 앱을 만들 수 있는 프레임워크 입니다.

특이사항으로는
- React에 의존하고 있으며, 기본적인 코드는 React와 동일합니다.
js 기반 라이브러리를 사용 할 수 있습니다.

- 단지 Web APIs, Window 객체 요런 부분들은 사용할 수 없습니다.
div, image 등의 태그를 사용하지 않고,
React Native에서 제공하는 View, Image, Text를 사용해야합니다.

- js에서 android, ios의 네이티브 기능을 사용 할 수도 있습니다.

- android, ios 앱으로 빌드 할 수 있습니다.

React를 알고 있다고 가정했을 때
java 또는 kotlin 과 object-c 또는 swift를 배우지 않고 뻐르게 개발할 수 있는 방법중 하나라고 생각합니다.


## Expo

React Native를 처음 개발 할 때 어느정도 서치를 해보면, Expo라는 키워드가 자주 나옵니다

Expo란 React Native로 개발 할 때 사용할 수 있는 라이브러리라고 할 수 있습니다.

Expo를 사용하고, 안하고 개발 방식에 차이가 있습니다만,
React Native의 기능을 더 단순화 시켰다고 생각하시면 됩니다.

기존 React Native 개발
- 네이티브 빌드를 필수적으로 처리해야함.
=> 실제 Android Studio, Xcode의 사용법을 알아야함.
- Gradle, CocoaPod 같은 네이티브 패키지 매니저도 관리해야함.
=> 똑같은 프로젝트고, npm install 까지 했어도, 네이티브 패키지에 따라 다른 결과가 나올 수 있음.
- 네이티브 폴더 android, ios 폴더를 직접 관리해야함.

Expo를 사용하면,
이러한 네이티브 부분을 신경쓰지 않고, 좀 더 js스럽게 관리할 수 있습니다.


## 비교

그림과 비유로 간단하게 React & Expo의 전체적인 개발 프로세스 그리고 차이점을 설명드리겠습니다.

<img width="559" alt="스크린샷 2023-12-06 오후 11 31 22" src="https://github.com/outsung/frontend-article-study/assets/40460655/543e4de0-346f-49dd-ab07-792789adfdda">

React Native로 만들어진 앱입니다.
Expo도 결국 React Native 기반이기 때문에 요런 구조는 동일합니다.

<img width="828" alt="스크린샷 2023-12-06 오후 11 37 11" src="https://github.com/outsung/frontend-article-study/assets/40460655/d25be03a-b035-4c86-a2cc-83b38afc6946">

js 부분에서 View 태그를 사용해 레이아웃을 잡았다고 가정하면,
해당 View는 android, ios에 *미리 정의된* `UIView` (ios), `android.view` (android) 으로 변경됩니다.

View 뿐만이 아니라, 핸드폰 후레쉬 키는 함수, 카메라 키는 함수, 파일 가져오는 함수 ... 등등 네이티브 부분에 먼저 정의된 기능을 js에서 불러와서 사용할 수 있습니다.
그래서 네이티브한 부분이 변경되면, 다시 빌드를 해서 *미리 정의*를 해둬야 합니다.


### 개발 프로세스도 웹과는 차이가 납니다.
일단 앱 개발이니까 시뮬,에뮬레이터 또는 실제 기기에서 확인해야합니다.

- android, ios를 dev 모드로 빌드 (Android Studio, Xcode)
- js 부분 빌드 (metro)

이렇게 하고 합쳐서 확인할 수 있습니다.

웹도 번들 서버를 켜고, HMR로 바로바로 확인하는 것 처럼
React Native도 js 부분을 metro라는 번들러를 사용합니다.

<img width="1016" alt="스크린샷 2023-12-06 오후 11 54 46" src="https://github.com/outsung/frontend-article-study/assets/40460655/0113e8c0-d96f-45c1-a98c-38d472e4797b">

실제로 js 부분은 바로 변경 후 확인이 가능합니다.

<img width="1067" alt="스크린샷 2023-12-06 오후 11 55 41" src="https://github.com/outsung/frontend-article-study/assets/40460655/f018ec89-29ca-4420-9441-0ffb4f67dcfd">

네이티브 부분이 변경되면 다시 빌드해서 네이티브 부분을 바꿔야 합니다.
js에 비해 오래 걸립니다.


### Expo Go

Expo 이러한 네이티브 부분을 다시 빌드하는 것을 최소화 하기위해

Expo에서 선정한 몇몇의 네이티브 라이브러리를 포함한 상태로 만들어두었습니다. 그것이 ExpoGo 입니다.

<img width="1079" alt="스크린샷 2023-12-07 오전 12 20 28" src="https://github.com/outsung/frontend-article-study/assets/40460655/03952c0f-0ae2-463b-9ee0-a11487fb989e">

그래서 *Expo에서 선정한 몇몇의 네이티브 라이브러리* 만 사용한다면 네이티브 빌드를 할 필요없이,
metro로 js 부분만 빌드만 하고, 바로 확인 할 수 있습니다.


그럼 선정받지 못한 네이티브 라이브러리를 사용하고 싶다면?
Expo의 dev-client를 사용하시면됩니다.

dev-client는 ExpoGo에 나만의 네이티브 모듈을 추가한 Custom Expo Go 입니다.

네이티브 부분이 수정되었다면, 그때 딱 한번 dev-client를 사용해서 네이티브 부분을 빌드하면 ExpoGo 사용하시는 것과 동일하게 사용이 가능합니다.
