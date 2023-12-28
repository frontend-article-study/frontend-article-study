# 사실 그냥.. 해보면서 겪은 삽질입니다.

네...대충 쓰다가 조금 빡빡하게 써보려고하니까 문제가 터져서 디버깅하느라 고생좀 했습니다.

# zod는 뭔가요?

꽤 유명한 라이브러리이다보니 대부분 들어보셨거나 사용해본 경험이 있으실 것 같습니다.

혹시 모르시는 분들을 위해 간략히 설명드리면

스키마 밸리데이션을 도와주는 라이브러리입니다.

공식문서에서는 다음과 같이 소개를 하고있습니다.

**TypeScript-first schema validation with static type inference**

타입스크립트를 잘 지원해주면서 스키마 밸리데이션을 정적 타입추론과 함께 도와주는 라이브러리

라고 요약할 수 있는데 쉽게 말하면 유효성검사를 해주는 라이브러리입니다.

# 문법이 어떻게 되나요?

```tsx
const simpleSchema = z.object({
  name: z.string(),
  money: z.number(),
});

// 만약 이 스키마를 쓰고싶다면 ?

const exampleObj = {
  name: '우은희',
  money: 1000,
};

const validatedWooEunhe = simpleSchema.parse(exampleObj);
//이 과정을 거치면 타입이 스키마에서 정의한 타입으로 좁혀집니다.
```

# 이게 왜 좋은건가요?

타입스크립트의 절대적인 한계는 명확합니다. 개발 단계에서만 동작하고 트랜스파일을 진행하고 나면

타입과 관련된 코드는 모두 날아가고 일반적인 자바스크립트 코드가 된다는 것인데요

이것은 장점이기도 하지만 동시에 단점이기도 합니다.

바로 런타임에 변화무쌍하게 들어오는 외부의 값들을 타입스크립트가 검증해줄 수는 없다는 것인데요

이것을 해결해주기위해 zod와 같은 런타임 밸리데이션 라이브러리가 등장하는 것입니다.

# form과 사용할때는 form 법을 따라야한다.

사실 zod를 프론트엔드에서 사용하는 대부분의 경우는 폼밸리데이션을 원활히 하기 위해서일 것입니다.

근데 form의 세계에서는 거의 모든게 string이라는 점을 유의해야합니다.

<br/>

1 2 3 4와 같은 숫자 ? string입니다.

type이 number인 인풋에서 받은 숫자? string입니다.

즉 인풋 제출 값을 그대로 이용해서는 평생 아래 밸리데이션을 통과할 수 없다는 뜻입니다.

```tsx
const onChange = (event) => {
  const validate = z.number().parse(event.target.value); // 절대 통과 불가능..
};
```

유감이네요.. 그럼 어떻게 해야할까요?

```tsx
const onChange = (event) => {
  const validate = z.string().parse(event.target.value); // 통과!
};
```

이렇게 통과하도록 한뒤 세부적인 검증은 zod에서 제공하는 refine , superRefine 메서드를 이용해야합니다.

예를 들어 number 인풋에서 입력받은 값이 숫자 10을 넘을때에만 통과시켜주는 밸리데이션이 필요하다고 가정해봅시다

그렇다면 다음과 같은 코드를 작성할 수 있어요

```tsx
const onChange = (event) => {
  const validate = z
    .string()
    .refine((data) => Number(data) > 10)
    .parse(event.target.value);
};
```

refine 메서드는 내부에 콜백함수를 집어넣으며 이 콜백함수의 첫번째 인수에는 검증중인 값이 들어가있습니다.

z.string()을 통과하고 문자열이라는 것이 검증된 경우에는 refine()메서드가 동작하게되는것입니다.

zod는 이런식으로 검증을 순차적으로 실행할 수 있도록 도와줍니다.

이런 특성은 폼밸리데이션에서 매우 유용한데요 예를 들면 좀 더 섬세한 유저경험을 만들어줄수있습니다.

사용자가 아무것도 입력하지 않은 상태에서 제출을 누르면 "빈칸을 채워주세요"라는 에러를 발생시키고

10 미만의 값을 입력한 상태로 제출을 누르면 "10 이상의 값을 넣어주세요"와 같이 에러를 섬세하게 발생시킬수있게되는거죠!

### 이스터에그

```tsx
const validated = z
  .string()
  .refine((data) => Number(data) >= 0)
  .parse('');
```

위 밸리데이션은 통과됩니다.

왜냐하면 "" 와 같은 빈문자열이 Number() 안에 들어가면 반환값은 0이거든요.. ㅎㅎ;;

## 그런데 실제로 이 값을 사용할때에는 number여야하는 경우에는요?

그런 경우를 지원하는 transform 메서드가 존재합니다.

이렇게 사용할 수 있어요

```tsx
const hi = z
  .string()
  .refine((data) => Number(data) > 10)
  .transform((data) => Number(data))
  .parse('100');
// 이때 hi의 타입은 number입니다.
```

이렇듯 transform 을 이용하여 적절히 값을 변환시켜주는 작업도 수행할 수 있습니다.

# react-hook-form과의 통합

앞서 설명드렸듯이 zod를 프론트엔드에서 사용하는 경우에는 주로 폼밸리데이션을 위한 경우가 많습니다.

간단한 예시 코드를 통해 어떻게 사용하게되는지 보여드릴게용

```tsx
npm i react-hook-form zod @hookform/resolvers
```

hookform과 zod 그리고 그 둘을 통합시켜줄 리졸버를 설치합니다.

```tsx
import * as z from 'zod';
import ZodResolver from '@hookform/resolvers/zod';
import { useForm } from 'react-hook-form';

const zunSchema = z.object({
  sung: z.string(),
  zi: z.string().optional(),
  hyun: z.string().refine((data) => Number(data) >= 1, {
    message: '0이상의 숫자를 입력하세요',
  }),
});

type ZunSchemaType = z.infer<typeof zunSchema>;

const ZunComponent = () => {
  const zunForm = useForm<ZunSchemaType>({
    resolver: zodResolver(zunSchema),
  });
};

return (
  <div>
    <input {...zunForm.register('sung')} />
    <input {...zunForm.register('zi')} />
    <input {...zunForm.register('hyun')} />
  </div>
);
```
이런 형태로 사용해줄 수 있습니다.

?? 아니 react-hook-form에서도 밸리데이션 지원해주잖아요 라고 생각할수도 있지만

이렇게 리졸버를 통해 런타임 밸리데이터 라이브러리와 훅폼을 결합시키는것은

밸리데이션 논리를 명확하게 외부로 분리시킬 수 있도록 도움을 주고

또한 더욱 강력한 검증 방법과 검증 이후 값 변환, 디폴트값 설정등의 편의기능을 지원해준다는 점이 주목할만합니다.


## 와 zod 짱이다 당장 써야지

하지만 zod는 번들사이즈가 필요성에 비해서는 조금 크다는 단점이 있습니다.

unpackedsize 기준으로

@hookform/resolver는 555 kB

zod는 628 kB에 달하는 번들사이즈를 가집니다.


감이 안온다구요?

react의 unpackedsize는 316 kB 입니다

zustand, redux ,jotai 등등 주로 사용되는 전역 상태관리라이브러리들의 평균

unpackedsize 역시 300~400 kb 선이고요!


폼을 많이 쓰지 않는 프로젝트에서 단순 폼밸리데이션을 위해서 사용하기에는 조금 무겁다는 인상이 들 수 있어요

(그래도 저는 애용합니다.)

# 마치며

이외에도 커스텀에러메시지를 만드는 방법, enum인 경우 에러메시지를 설정하는 법 등등 여러가지 사용법이 있는데

관심 있으신 분들은 따로 더 찾아보시면 좋습니다.

리액트 훅폼과 결합하여 사용하면 정말 빡빡한 유효성검사도 선언적으로 잘 관리할 수 있게되어

개인적으로 애용하는 라이브러리에요!
