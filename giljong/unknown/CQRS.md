# CQRS?

명령 질의 책임 분리이다. (고마워요 스피드웨건)

(Command Query Responsibility Segregation)의 준말

디자인 패턴의 한종류라고 인식할 수도 있는데요 여러가지 시스템과 추상화 수준에서 적용될 수 있는 말이기에

단순히 CQRS 만 보면 안되고 해당 단어가 사용된 맥락까지 모두 살펴보아야 좋습니다.

# CQS

버트란드 마이어는 1988년 (Command Query Separation)을 제안했다고 합니다.

이 때 CQS는 객체 수준의 디자인을 의미했는데요.

이 당시 이 사람은 메서드를 "함수"와 "프로시저" 두가지 타입으로 나누어야 한다고 제안했습니다.

1. 함수 ? -> 결과물을 생산하지만 상태에 영향을 주지는 않는다.

2. 프로시저 ? -> 상태에 영향을 주지만 결과물을 제공하지 않는다

즉 이것은 질문하는 행위가 상태를 바꾸면 안된다는 원칙을 공고히합니다.

이 관점에서 함수는 질문을 던지고 그에 대한 답을 받는 행위라고 할 수 있으며

프로시저는 상태를 바꾸는 행위라고 정의할 수 있겠네요

# CQRS와 CQS의 차이

Domain Driven Development라고 일컫는 도메인 주도 개발의 개념과 CQS의 개념이

오늘 다룰 CQRS 패턴이 탄생하는 것에 기여했다고 합니다.

CQRS 라는 단어는 2009년 언저리에 등장했다고 하는데

2012년 Fitzgerald는 CQRS를 "애플리케이션이나 시스템의 책임을 쓰기와 읽기로 분리하는데, 객체 수준이 아니라 전체 아키텍처 수준에서 분리하는 것"이라고 정의하였어요

즉 CQRS 와 CQS를 가르는 개념적 정의는 "객체 수준" 에서 이루어지느냐 아니면 "아키텍처" 수준에서 이루어지느냐로 나누어 볼 수 있겠네요

# CQRS 이거.. 좋은거에요?

# 레퍼런스

https://myeongjae.kim/blog/2022/02/03/fundamental-cqrs

http://lup.lub.lu.se/student-papers/record/4864802

https://learn.microsoft.com/ko-kr/azure/architecture/patterns/cqrs

https://docs.aws.amazon.com/ko_kr/prescriptive-guidance/latest/modernization-data-persistence/cqrs-pattern.html

https://lup.lub.lu.se/luur/download?func=downloadFile&recordOId=4864802&fileOId=4864803

https://github.com/yerinadler/typescript-event-sourcing-sample-app
