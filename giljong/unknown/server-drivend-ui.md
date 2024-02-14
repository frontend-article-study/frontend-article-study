# Server Driven UI

Server Driven UI 란 클라이언트 애플리케이션의 UI 구성 요소 / 레이아웃을 서버에서 동적으로 제어하는 방식을 말합니다.

주로 서버의 api에서 내려주는 데이터에 의존하는 형태로 화면을 구성하는 식으로 구현을 하게되는데요

화면이 어떻게 그려져야 할지를 선언하고 클라이언트측에서 해당 선언내용을 가지고 화면을 그린다는 점에서

html과 비슷하기도 합니다.

# 왜 씀?

server driven ui의 장점은 "유연함"이라고 요약할 수 있습니다.

예컨대 우리는 화면의 내용을 바꾸기 위해서는 프론트엔드측 코드를 수정하고 빌드한뒤 다시 배포하는 프로세스를 거칩니다.

반면 server driven ui는 프론트엔드측의 코드를 수정할 필요가 없습니다.

왜냐하면 server 측에서 내려주는 데이터를 수정하는 것으로 화면을 바꿀 수 있기 때문이에요

이러한 server driven ui는

1. 한번한번의 배포에 오랜 시간과 리소스를 들여야하는 앱 배포,

2. 프론트엔드 개발자의 리소스가 한정된 상황에서 최대한의 생산성이 필요한 경우

3. 각 그룹에게 서로 다른 UI를 제공할 필요가 있는 A / B 테스트

4. 비개발자 / 비프론트엔드 개발자도 화면을 다뤄야하는 경우

등의 상황에서 매우 유용하게 사용할 수 있습니다.

# 근데 왜 안씀?

1. 초기에 server driven ui를 위해 구성해야할 아키텍처가 복잡함

2. 유연하고 많은 케이스에 대응하게 하기위해서는 그만큼 설계에 많이 투자해야함 중도를 지키기 힘듬

3. 컴포넌트로 명확히 구현해야함 (디자인 시스템을 구축하는것부터가 시작임;)

# 근데 어케 씀?

server driven ui의 데이터부분은 주로 언어에 구애받지않는 JSON 혹은 YAML 형태로 구성합니다.

예컨대 이런 경우를 생각할 수 있겠죠?

```json
{
  "name": "WooEunheInput",
  "type": "text",
  "required": true,
  "placeholder": "우은희는 전설이다"
}
```

그런데 server driven ui라고 부르려면 레이아웃까지 그러니까 하나의 페이지를 제대로 그려낼 수 있어야겠죠?

당연히 조금 더 복잡해집니다. 간략화한 예시를 하나 보고가봅시다

```json
{
  "type": "componentScreen",
  "title": "Leonard Bernstein",
  "header": {
    "image": { "url": "..." },
    "title": "Leonard Bernstein",
  },
  "sections": [
    {
      "heading": {
         "title": "Popular Works",
         "button": { "title": "Show all", ...}
      },
      "components": [
         {
           "type": "detailRow",
           "title": "West Side Story",
           "disclosureText": "258",
         },
         {
           "type": "detailRow",
           "title": "Candide",
           "disclosureText": "116",
         },
         {
           "type": "detailRow",
           "title": "Symphonic Dances",
           "disclosureText": "70",
         }
       ]
     },
     {
       "heading": {
          "title": "Biography",
       },
       "components": [
          {
            "type": "text",
            "paragraphs": "Leonard Bernstein was an American pianist...",
          }
        ]
      }
    }
  ]
}
```

이제 저런 데이터를 토대로 그릴 수 있게 코드 짜시면 됩니다.

# 마치며

당연하지만 이런 건 mvp 단계에서 고민한다기보다는 엄청나게 빨리 성장하면서도 일손이 부족한 경우에 도입을 고려해야할 것입니다.

잘 만들어진 시스템이 필요할 정도로 바쁜 경우에 의미가 있지 당장 살아남아야하는 상황에서는 그냥 노가다하는게 맞을 것 같네요

server driven ui를 사용하는 대표적인 기업은 카카오 스타일 , 토스, 에어비앤비, 구글 등이 있습니다.

# 레퍼런스

https://devblog.kakaostyle.com/ko/2021-12-16-1-server-driven-ui/

https://www.youtube.com/watch?v=3wxG1WLDONI&t=646s

https://www.youtube.com/watch?v=EuDOn7255gI

https://deview.kr/data/deview/session/attach/7_%E1%84%82%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%87%E1%85%A5%20%E1%84%80%E1%85%A5%E1%86%B7%E1%84%89%E1%85%A2%E1%86%A8%E1%84%8B%E1%85%B4%20Server%20Driven%20UI%20-%20LAPIN.pdf

https://techblog.pet-friends.co.kr/

next-js%EB%A1%9C-%EC%9D%B4%EC%A0%84%ED%95%98%EA%B3%A0-server-driven-ui%EB%A1%9C-%EB%B3%80%ED%99%94%ED%95%98%EB%8B%A4-3954be8b981d

https://brunch.co.kr/@advisor/37

https://medium.com/airbnb-engineering/a-deep-dive-into-airbnbs-server-driven-ui-system-842244c5f5
