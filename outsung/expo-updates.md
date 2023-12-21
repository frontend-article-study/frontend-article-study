
## react-native-code-push

일반적인 ios, android 앱을 개발 하고, 배포까지 했다고 가정하자.
글자 하나, 이미지 하나 수정하려면, 수정 후 다시 빌드, 배포 (심사 포함) 해야한다.

이러한 부분을 해결하기 위해서 microsoft가 react-native-code-push를 만들었다.

React Native는 native부분과, js부분으로 이루어져있는데,

react-native-code-push를 사용하면, 여기서 js부분을 동적으로 바꿀 수 있다. (이미지, 폰트등 에셋 파일도 가능)

code push를 통한 업데이트의 플로우는 대략 아래와 같다.

1. 코드 수정
2. js 부분 빌드 (메트로를 사용해 번들링)
3. 업데이트 서버에 번들 업로드 => 버전, 플랫폼, 릴리즈네임(production, stage, dev 등)
---
4. 앱에서 **버전, 플랫폼, 릴리즈네임**으로 업데이트 서버에 요청 함.
5. 업데이트 서버는 **버전, 플랫폼, 릴리즈네임**의 가장 최신의 버전을 응답 함.
6. 해당 버전을 사용해도 되는지 확인 (생성 시간 비교, signing 검증)
7. 새로운 버전을 내부에 저장 및 사용하도록 설정
8. 앱 재시작


react-native-code-push는 기본적으로 App Center 라는 서비스에 가입 후 사용이 가능하다.
여기서 App Center가 업데이트 서버의 역할이다.

새로운 버전을 업로드 할때 App Center에 업로드하고, 해당 버전을 요청하는 서버도 App Center이다.

react-native-code-push는 여러 기능을 제공한다.

- 롤백 기능: 모든 버전을 저장하고 있기에, 최신 버전으로 업데이트 이후, 바로 앱이 충돌한다면, 이전 버전으로 되돌릴수 있다.
- 버전별로 몇명이 사용하고 있고, 몇명이 롤백 했는지 확인 할 수 있다.
- 릴리즈네임을 사용하고있어서 production, stage등 여러 환경 및 테스트에 활용할 수 있다.
- App Center cli를 통해 업로드를 할 수 있다.
- ... 등등

단점은 아래와 같다.
- App Center에 의존적이다. 실제로 가끔 서버가 느리거나, 멈추는 경우가 있음.
- 흠 조금 복잡하다 ? 


## expo-updates
expo에서 물론 react-native-code-push도 사용할 수 있다.

하지만 expo에서는 expo가 제공하고 있는 expo-updates가 있다.

요걸 쓰는것이 간편하다.
expo에는 실제로 이미 적용이 되어있기도 하고
expo(eas) cli를 사용해 간단하게 업로드도 가능하다.

### EAS가 뭐죠
EAS => Expo Application Services

요 녀석이 App Center라고 보면 되는데,
EAS는 빌드, 업데이트(code push), 배포까지 관리해준다.

잘만 활용하면, 내 컴퓨터말고 EAS를 통해 빌드, 배포까지 해버리고 나는 업데이트만 쓱쓱할 수 있다.

## expo-updates 장단점 (vs react-native-code-push)

다시 expo-updates로 돌아와서

expo-updates 장단점을 react-native-code-push와 비교하면서 설명하겠다.

내부 로직은 제외하고, 실제 사용자에 영향을 받는 장단점으로는 아래와 같다.

- eas에서 관리되는 update들은 일정 사용량 이후 유료다. (App Center는 무료)
- eas서버는 잘 안죽는다
---
- Expo Updates Protocol을 명시적으로 사용하고, 사용자들도 해당 프로토콜만 지킨다면, update 서버를 만들수 있다: 공식 문서에서도 자세히 나와있음. 예시 레포도 있음.


### expo-updates 라이브러리 한계점 및 해결 방안

- 유료 
- 릴리즈 네임 동적으로 변경
- 사용량 판단

해결 방법: 커스텀 서버를 만들자!

-> custom-expo-updates-server