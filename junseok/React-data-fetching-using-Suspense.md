# 1. React Data Fetching의 종류
- 리엑트의 이전 공식문서를 참고하면, Data Fetching의 방식에 대해서 세가지 논리적 방법을 소개하고 있습니다.
    1. Fetch-on-render(fetch in useEffect): 컴포넌트 렌더링을 시작하고 useEffect로 비동기 처리를 합니다.
    2. Fetch-then-render(Relay without Suspense): useEffect로 화면을 그리는 데에 필요한 모든 데이터를 한 번에 조회 후 렌더링을 시작합니다. 
    3. Render-as-you-fetch(Relay with Suspense): 비동기 작업과 렌더링을 동시에 시작합니다. 즉시 초기 상태를 렌더링하고, 비동기 작업이 완료되면 다시 렌더링합니다.


## Fetch-on-render
React 컴포넌트들은 각각 자신만의 생명 주기를 가지기 때문에 비동기 작업들도 코드 의도와는 다르게 동작할 수 있습니다. 이를 해결하기 위해 **fetching과 동기화** 하는 작업이 이루어져야 합니다. 
**하지만 이러한 방식은 컴포넌트의 복잡도를 증가시킬 뿐만 아니라 `Waterfall Problem`을 유발할 수도 있습니다.**
```ts
// fetch 동기화 예시 
import { useState, useEffect } from "react"


type Data ={
    url:string
}


export default function RenderAsFetch(){
  const [data, setData]=useState({url: "", lendering: false})

  useEffect(()=>{
    
    (async()=>{
        const data = await waitFetchData()
        setData({url: data[0].url , lendering:true})
    })()

  },[])

  if(!data.lendering){
    console.log("pending- wating image loading one");
    return <div>Loading...</div>
  }
  console.log("Render - Rendering Cat Image one")


  return <div>
    <img src={data.url} width={200} height={200}/>
    <SecondImgae isLendering={data.lendering} />
  </div>
}


function waitFetchData(){
    return new Promise<Data[]>(resolve => setTimeout(async()=>{
        const json = await fetch("https://api.thecatapi.com/v1/images/search")
        const data: Data[] = await json.json()
        return resolve(data)
    }, 1000))
}


type Prop ={
    isLendering: boolean
}
function SecondImgae(isLendering: Prop){
    const [data, setData]=useState({url: "", lendering: false})

    useEffect(()=>{
        if(isLendering){
            (async()=>{
                const data = await waitFetchData()
                setData({url: data[0].url , lendering:true})
            })()
        }
      },[isLendering])

    if(!data.lendering){
        console.log("pending- wating image loading two");
        return <div>Loading Second</div>
    }
    console.log("Render - Rendering Cat Image two")
    return <div>
        <img src={data.url} width={300} height={300}/>
    </div>
}
```

## Fetch-then-render
- 비동기 작업들의 동시성을 보장하는 방식입니다. 필요한 비동기 작업들을 모두 한 번에 처리하고 이를 각 컴포넌트에 뿌려주는 방식입니다.
```ts
import { useState, useEffect } from "react"

type State = {
    url: string
    lendering: boolean
}

export default function RenderThenFetch(){
    const [data, setData]=useState<State>({url: "", lendering: false})
    const [secondData, setSecondData]=useState<State>({url: "", lendering: false})

  useEffect(()=>{
    Promise.all([fetchData(), fetchData()]).then(([firstImage, secondImage])=>{
        setData({url: firstImage.url, lendering: true})
        setSecondData({url: secondImage.url, lendering: true })
    })
  },[])

  if(!data.lendering){
    console.log("pending- wating image loading one");
    return <div>Loading...</div>
  }
  console.log("Render - Rendering Cat Image one")


  return <div>
    <img src={data.url} width={200} height={200}/>
    <SecondImgae imageUrl={secondData}/>
  </div>
}

function SecondImgae({imageUrl}: {imageUrl: State}){
    console.log("Render - Rendering Cat Image two")
    return <div>
        <img src={imageUrl.url} width={300} height={300}/>
    </div>
}


type Data ={
    url:string
}

function fetchData(){
    return new Promise<Data>((resolve)=>{
        const data = fetch("https://api.thecatapi.com/v1/images/search").then(json => json.json()).then(data=> data[0] as Data)
        return resolve(data)
    })
}
```
비동기 작업들을 한곳에서 처리하고 그 결과를 각 컴포넌트에게 전달하는 방식은 **컴포넌트들 간의 역할분담이 불명확하게 하고 높은 결합도**를 만들 수 있습니다.

## Render-as-you-fetch
- Render-as-you-fetch는 리엑트 16버전에서 실험적인 기능인 `Suspense`를 활용하는 방식입니다. Suspense는 리엑트 18버전에서 정식 기능으로 채택되었습니다.

- 위의 두 방식은 결과적으로 **명령형 프로그램**의 특징을 가지고 있습니다. (명령형 프로그램:  "무엇을 할 지를(How)" 결정하는 프로그래밍 ⇒ 비동기 상태값을 가지고 어떤 UI를 보여줄지에 대한 분기 로직을 JSX에 코딩)

- 반면 Suspense를 활용하는 방식은 **선언형 프로그램**의 특징을 가지고 있습니다. (선언형 프로그램: 
"무엇이 될지를(What)" 결정하는 프로그래밍 ⇒ 비동기 상태값에 따른 UI를 Prop으로 주입하기)

- **Suspense는 컴포넌트의 렌더링을 어떤 작업이 끝날 때까지 잠시 중단시키고, 다른 컴포넌트를 먼저 렌더링 할 수 있도록 도와주는 기능**입니다. Suspense를 사용하면 Component Lazy Loading이나 Data Fetching 등의 비동기 처리를 할 때, 응답을 기다리는 동안 fallback UI(e.g Spinner)를 보여주고, 그 사이에 우선순위가 높은 다른 UI들을 먼저 렌더링 할 수 있습니다.

- Suspense는 Promise의 상태를 반환하는 코드가 있어야 fullback component를 진행합니다.
```ts
// App.tsx
import React from "react";
import UseSuspense from "./components/UseSuspense";

export default function App(){
  return    (
  <React.Suspense fallback={<div>Loading.....</div>}>
    <UseSuspense />
  </React.Suspense> 
  )
}

```
```ts
// UseSuspense.tsx
// react의 공식 문서를 참고하여 작성하였습니다.
/*
이는 Suspense와 이를 둘러싼 Data Fetching에 대한 개념적 모델을 구현한 프로토타입이므로 
실제로 React가 이렇게 Suspense와 Data Fetching을 처리하지는 않는다는 것을 주의해야 합니다.
*/
type Data = {
    id: string
    url: string
    width: number
    height: number
}


export default function UseSuspense(){
    const catImages = use(fetchData("https://api.thecatapi.com/v1/images/search?limit=10`"));
    console.log(catImages)
  return (
        <div>
            {catImages.map((image:Data) => (
                    <img key={image.id} src={image.url} width={500} height={500} alt={`Cat image: ID is a ${image.id}`} />
            ))}
        </div>
  );
}


// callback 함수가 반환하는 promise에 따라 Suspense가 동작합니다.
function use(promise: any ) {
    if (promise.status === 'fulfilled') {
      return promise.value;
    } else if (promise.status === 'rejected') {
      throw promise.reason;
    } else if (promise.status === 'pending') {
      throw promise;
    } else {
      promise.status = 'pending';
      promise.then(
        (result: any) => {
          promise.status = 'fulfilled';
          promise.value = result;
        },
        (reason: any) => {
          promise.status = 'rejected';
          promise.reason = reason;
        },      
      );
      throw promise;
    }
  }

// 데이터 가져오는  함수
const cache = new Map();
function fetchData(url: string) {
  if (!cache.has(url)) {
    cache.set(url, getData(url));
  }
  return cache.get(url);
}

async function getData(url: string) {
    try{
        await new Promise(resolve => {
            setTimeout(resolve, 3000);
        });
        const json = await fetch(url)
        const data = await json.json()
        return data
    }
    catch(err){
        return err
    }
}
```
결과적으로 **Suspense는 개발자가 Data Loading에 대한 상태를 선언적으로 관리할 수 있는 방식을 제공**하며, 이로 인해 메인 컴포넌트가 “데이터가 성공적으로 로드되었을 때”에 대한 상태만 신경 쓸 수 있게 도와주는 역할을 한다는 것을 알 수 있습니다.

- 실제로 Suspense가 Child Component의 데이터 로딩에 대한 상태를 확인하는 방법은 다음과 같습니다.
    1. render method에서 캐시로부터 값을 읽기를 시도한다
    2. value가 캐시 되어 있으면 정상적으로 렌더 한다
    3. value가 캐시 되어 있지 않으면 캐시는 “Promise를 throw 한다
    4. promise가 resolve 되면, React는 “Promise를 throw 한 곳으로부터” 재시작한다.

중요한 포인트는 데이터가 로딩 중일 때, "컴포넌트는 Suspense(가장 가까운 Parent에 위치한)에게 promise를 throw" 한다는 것과, 데이터가 "로딩되고 나면(Promise가 Resolve 되고 나면), Suspense는 Fallback UI를 보여주는 것을 멈추고, 다시 정상적으로 컴포넌트를 로딩한다" 는 점입니다. 

promise를 throw 했을 때는 Suspense에서 이를 잡아서 resolve 될 때까지 fallback UI를 보여주다가 resolve가 되면, 다시 throw 한 곳으로부터 정상적인 컴포넌트 로딩을 처리할 수 있게 되는 것입니다.

# TIP 대수적 효과(Algebraic Effect)의 영향을 받은 Suspense
- suspense는 컴포넌트가 throw 하는 promise의 상태에 따라서 동작을 하게 됩니다. 이러한 모델은 "Algebraic Effect(대수적 효과)"는 아니지만, 이로부터 영향을 받았다는 의미로 해석됩니다.
> **대수?**
>
> 어떠한 수학적인 문제를 해결할 때, 수들의 연산으로 이를 해결하는 것이 아닌 문자와 문자들 사이의 추상적 관계(함수)들을 통해 이를 해결하는 방식을 ‘대수적 방식’이라고 하고, 이는 변수와 함수를 사용해서 문제를 해결하는 프로그래밍과도 관련

### 대수적 효과?
프로그래밍에서 대수적 효과는 다음과 같이 정의가 됩니다. 
**대수적 효과란 exception throw등을 포함하는 연산들의 집합(set of operations)으로부터 순수하지 못한 행위(impure behavior)들이 발생할 수 있다는 전제를 바탕으로 Computational Effect에 대해 접근하는 방식이다.**

- Computationla Effect?
 프로그램이나 함수가 실행될 때 가질 수 있는 관찰 가능하고 잠재적으로 부작용을 일으킬 수 있는 동작을 의미합니다. 이러한 효과는 종종 두 가지 주요 범주로 분류됩니다.
  1. Pure function: 순수 함수는 동일한 입력이 주어졌을 때 관찰 가능한 부작용을 일으키지 않고 **항상 동일한 출력을 생성하는 함수**입니다. 순수 함수는 결정적이며 프로그램의 외부 상태에 영향을 주지 않습니다. 예측 가능성이 높으며 쉽게 테스트하고 추론할 수 있습니다. 함수형 프로그래밍 언어는 순수 함수의 사용을 권장합니다.
  2. Impure function: 불순한 함수는 **동일한 입력에 대해 다른 출력을 생성할 수 있거나 관찰 가능한 부작용이 있는 함수입니다**. 이러한 부작용에는 전역 변수 수정, 파일 읽기/쓰기, I/O 작업 수행 또는 네트워크 요청 만들기가 포함될 수 있습니다. 불순한 함수는 예측 가능성이 낮고 프로그램을 이해하고 테스트하기 어렵게 만들 수 있습니다.

- 대수적 효과를 조금 더 javascript적이고 react적으로 해석하면 다음과 같이 설명할 수 있습니다. 이는 정확한 대수 효과를 보장하지는 않습니다. 개념에 대한 인사이트를 얻는 것으로 충분합니다.
> **“Algebraic Effect는 부모 컴포넌트(Suspense)와 자식 컴포넌트(UserList) 사이의 interaction에서 발생한 SideEffect(throw Promise)를 적절한 Handler(catch Promise in Suspense)를 사용해 해결하는 접근 방식을 의미하는 것이고, 이 Handler는 sub-expression(UserList)에서 발생시킨 Effect(throw Promise)를 central-control(Suspense)에서 잡아 적절하게 처리한 뒤, 다시 sub-expression(UserList)이 멈췄던 곳으로 흐름을 되돌려 주는 것입니다.”**

- 이러한 대수적 효과를 통한 코드 작성 방법은 **선언적으로** 코드를 작성할 수 있는 관점을 제공하며, 이는 react가 지향하는 방향과 일치합니다. Suspense는 Algebraic Effects를 사용해서 부모 컴포넌트에게 로딩 상태에 대한 처리의 “책임”을 넘겼으며, 따라서 메인 컴포넌트에서 로딩 UI 표시의 책임을 “분리”하게 됩니다.


# 결론
- React Suspense는 컴포넌트가 무언가를 렌더하기 위해 “wait”할 수 있도록 도와주며, 여기서의 “wait”은 데이터 로딩을 포함한 모든 비동기 요청이 해당될 수 있다.

- 따라서 (Error Boundary와 함께) Suspense를 사용하면 개발자는 컴포넌트에서 사용할 데이터가 “정상적으로 로드되었을 때”의 UI만 고려하면 되고, 로딩 중이나 에러의 상태에서의 UI는 위임하면 됩니다. (*하나의 컴포넌트에서 각각의 상태에 대한 관리를 명령적으로 관리할 필요가 없다*)

- Suspense는 Fallback UI를 보여주는 동안에도 Child Component를 렌더한다(단지 보이지 않을 뿐.) 따라서 보다 Concurrent하게 데이터를 요청할 수 있고, 결과적으로 그렇지 않을 때 보다 더 나은 사용자 경험을 제공한다.

# Reference
- [카카오 기술 블로그 Suspense](https://fe-developers.kakaoent.com/2021/211127-211209-suspense/)
- [리엑트 공식 문서 Suspense](https://react-ko.dev/reference/react/Suspense)
- [리엑트 구 공식 문서](https://17.reactjs.org/docs/concurrent-mode-suspense.html)
- [콴다 기술 블로그 Suspense - Suspense의 개념적 이해](https://fe-developers.kakaoent.com/2021/211127-211209-suspense/)
- [콴다 기술 블로그 대수적 영향을 받은 Suspense](https://blog.mathpresso.com/algebraic-effects-of-react-suspense-157b49807ea0)
- [우아한 테크코스 - 서스펜스와 애러 바운드리](https://www.youtube.com/watch?v=yQDsd1OVR08)