# Context API 의존성 주입

의존성 주입이라는 단어가 함수형 프로그래밍을 지향하는 리액트와는 전혀 연관이 없을 줄 알았습니다. 주로 객체 지향 프로그래밍(OOP)에서 많이 쓰이는 용어라고 생각했거든요…

### 의존성 주입(Dependency Injection)

의존성 주입은 말 그대로 의존성을 주입하는 것을 말합니다. 조금 더 구체적으로 말하면 클래스 A, B, C가 있습니다. A가 B에 의존 중일 때, B의 변경은 A에 영향을 미칩니다. 더군다나 B가 C에 의존 중이라면 C가 변경되었을 때, A와 B 모두 변경해야 될 가능성이 있습니다.

이처럼 서로 상호 간의 의존 관계에 놓여 있으면 결합도가 증가되며 재사용성이 떨어지게 되죠. 그래서 의존성 주입을 통해 모듈을 분리하고 재사용성을 높일 수 있습니다.

그래서 어떻게 모듈을 분리하고 재사용성을 높일 수 있는 걸까요? 의존성 주입은 필요한 객체를 직접 생성하지 않고 외부에서 넣어주는 방식입니다. 즉, 객체의 의존 관계를 내부에서 결정하는 것이 아니라 객체 외부에서, 런타임 시점에 결정하는 방식입니다.

예를 들어, 데이터를 페칭하는 상황에서 개발자의 개발 환경이 브라우저일수도 있고, 노드 환경일수도 있습니다. 또한 개발자의 성향에 따라 fetch를 사용할 수도 있고, axios를 사용할 수도 있죠.

이러한 상황일 때 의존성 주입은 정말 유용하게 사용할 수 있습니다.

```tsx
export const createHttp = (fetch = window.fetch) => {
  return (url, config) => fetch(url, config);
};

const axiosClient = createHttp(axios);
axiosClient('/todo', {
  method: 'get',
  headers: {},
  data: {},
});
```

### Context API로 의존성 주입하기

Context API로 의존성을 주입하는 경우 대부분 다음과 같은 코드 패턴으로 이루어집니다.

```jsx
// Contexts/DependencyProvider.jsx
import React, { useContext } from 'react';

const DependencyContext = React.createContext();

export function useDependency() {
  return useContext(DependencyContext);
}

export default function DependencyProvider({ myService, children }: Props) {
  return (
    <DependencyContext.Provider value={{ myService }}>
      {children}
    </DependencyContext.Provider>
  );
}
```

context를 사용할 최상위 컴포넌트에 Provider를 감싸줍니다.

```jsx
// App.jsx
import { useState } from 'react';
import MyComponent from './Components/MyComponent';
import DependencyProvider from './Contexts/DependencyProvider';
import './App.css';

function App() {
  const myService = {
    callMe: () => {
      alert('hello!');
    },
  };

  return (
    <DependencyProvider myService={myService}>
      <MyComponent />
    </DependencyProvider>
  );
}

export default App;
```

그리고 context를 사용할 컴포넌트에서 불러줍니다.

```jsx
// Components/MyComponent.jsx
import { useDependency } from '../Contexts/DependencyProvider';

export default function MyComponent() {
  const { myService } = useDependency();
  myService.callMe();

  return <div>MyComponent</div>;
}
```

대표적인 사용 예시로 react-query가 있습니다.

```jsx
import React from 'react';
import './App.css';
import MainProducts from './components/MainProducts';

import { QueryClient, QueryClientProvider } from 'react-query';
import { ReactQueryDevtools } from 'react-query/devtools';

// Create a client
const queryClient = new QueryClient();

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <MainProducts />
      <ReactQueryDevtools initialIsOpen={true} />
    </QueryClientProvider>
  );
}
```

실제로 리액트 쿼리뿐만 아니라 상태 관리 라이브러리인 Redux, Recoil, Mobx 등에서 Context API를 의존성 주입 용도로만 사용하고 있다고 합니다.

### 의존성은 교체가 가능해야 합니다.

개발 단계에서 실제 데이터 요청과 Mock 데이터 요청을 따로 만들어 두어 네트워크 비용을 줄일 수 있습니다.

유튜브 API를 받아오는 상황에서 다음과 같이 실제 데이터를 요청 후 받아온 데이터를 json 파일로 저장합니다.

이렇게 미리 저장한 파일을 가져올 fakeClient를 정의합니다.

```jsx
import axios from 'axios';

export default class FakeYoutubeClient {
  async search({ params }) {
    return params.relatedToVideoId
      ? axios.get('/data/videos/related.json')
      : axios.get('/data/videos/search.json');
  }
  async videos() {
    return axios.get('/data/videos/popular.json');
  }
  async channels() {
    return axios.get('/data/videos/channel.json');
  }
}
```

그리고 실제 데이터를 요청하는 클래스도 정의합니다.

```jsx
import axios from 'axios';

export default class YoutubeClient {
  constructor() {
    this.httpClient = axios.create({
      baseURL: 'https://www.googleapis.com/youtube/v3',
      params: { key: process.env.REACT_APP_YOUTUBE_API_KEY },
    });
  }

  async search(params) {
    return this.httpClient.get('search', params);
  }

  async videos(params) {
    return this.httpClient.get('videos', params);
  }

  async channels(params) {
    return this.httpClient.get('channels', params);
  }
}
```

그리고 실제 구현체가 있는 클래스를 정의합니다.

```jsx
export default class YoutubeAPI {
  constructor(apiClient) {
    // DI 의존성 주입
    this.apiClient = apiClient;
  }

  async search(keyword) {
    return keyword ? this.#searchByKeyword(keyword) : this.#mostPopular();
  }

  async channelImageURL(id) {
    return this.apiClient
      .channels({ parmas: { part: 'snippet', id } })
      .then((res) => res.data.items[0].snippet.thumbnails.default.url);
  }

  async relatedVideos(id) {
    return this.apiClient
      .search({
        params: {
          part: 'snippet',
          maxResults: 25,
          type: 'video',
          relatedToVideoId: id,
        },
      })
      .then((res) =>
        res.data.items.map((item) => ({ ...item, id: item.id.videoId }))
      );
  }

  async #searchByKeyword(keyword) {
    return this.apiClient
      .search({
        params: {
          part: 'snippet',
          maxResults: 25,
          type: 'video',
          q: keyword,
        },
      })
      .then((res) =>
        res.data.items.map((item) => ({ ...item, id: item.id.videoId }))
      );
  }

  async #mostPopular() {
    return this.apiClient
      .videos({
        params: {
          part: 'snippet',
          maxResults: 25,
          chart: 'mostPopular',
        },
      })
      .then((res) => res.data.items);
  }
}
```

그리고 의존성을 주입해줄 Context를 생성합니다.

```jsx
import { useContext } from 'react';
import { createContext } from 'react';
import FakeyoutubeClient from '../api/fakeYoutubeClient';
import Youtube from '../api/youtube';
import YoutubeClient from '../api/youtubeClient';

export const YoutubeApiContext = createContext();

const Faker = new FakeyoutubeClient();
// const client = new YoutubeClient();
const youtube = new Youtube(Faker);

export function YoutubeApiProvider({ children }) {
  return (
    <YoutubeApiContext.Provider value={{ youtube }}>
      {children}
    </YoutubeApiContext.Provider>
  );
}

export function useYoutubeApi() {
  return useContext(YoutubeApiContext);
}
```

그리고 유튜브 API가 사용될 곳에 Provider를 감싸줍니다.

```jsx
import { QueryClient, QueryClientProvider } from 'react-query';
import { Outlet } from 'react-router-dom';
import SearchHeader from './components/SearchHeader';
import { YoutubeApiProvider } from './context/YoutubeApiContext';

const queryClient = new QueryClient();

function App() {
  return (
    <>
      <SearchHeader />
      <YoutubeApiProvider>
        <QueryClientProvider client={queryClient}>
          <Outlet />
        </QueryClientProvider>
      </YoutubeApiProvider>
    </>
  );
}

export default App;
```

그리고 아래의 코드와 같이 context를 사용하면 됩니다.

```jsx
import React from 'react';
import { useQuery } from 'react-query';
import { useParams } from 'react-router-dom';
import VideoCard from '../components/VideoCard';

import { useYoutubeApi } from '../context/YoutubeApiContext';

export default function Videos() {
  const { keyword } = useParams();
  const { youtube } = useYoutubeApi();

  const {
    isLoading,
    error,
    data: videos,
  } = useQuery(['videos', keyword], () => youtube.search(keyword));

  return (
    <>
      {isLoading && <p>Loading...</p>}
      {error && <p>Something is wrong</p>}
      {videos && (
        <ul className="grid grid-colos-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 2xl:grid-cols-5 gap-2 gap-y-4">
          {videos.map((video) => (
            <VideoCard key={video.id} video={video} />
          ))}
        </ul>
      )}
    </>
  );
}
```

### Context가 상태 관리가 아닌 이유

상태 관리는 시간이 지남에 따라 상태가 변경되는 방식을 의미합니다.

다음과 같은 경우를 상태 관리라고 합니다.

- 초기 값을 저장한다.
- 현재 값을 읽을 수 있다.
- 값 업데이트가 가능하다.

React의 useState와 useReducer가 이러한 예입니다.

- Hook을 호출해서 초기 값 저장
- Hook을 호출해서 현재 값을 읽는다.
- 제공된 setState 또는 dispatch 함수를 호출해서 값을 업데이트한다.
- 구성 요소가 Re-Render 되었기 때문에 값이 업데이트 됐음을 알 수 있다.

반면에 Context는 단지 데이터를 공유하는 역할을 합니다. 즉 Context는 상태가 Context Tree 내부에 포함된 다른 컴포넌트들과 공유되는 방식이죠.

### 이 외 다양한 Context API를 활용한 예시

- 단위 테스트
- Context API를 활용한 전략 패턴
  - https://itchallenger.tistory.com/916

[참고 사이트]

[https://velog.io/@jay/react-dependency-injection#왜-이게-중요한데요](https://velog.io/@jay/react-dependency-injection#%EC%99%9C-%EC%9D%B4%EA%B2%8C-%EC%A4%91%EC%9A%94%ED%95%9C%EB%8D%B0%EC%9A%94)

https://blog.testdouble.com/posts/2021-03-19-react-context-for-dependency-injection-not-state/

https://blog.logrocket.com/dependency-injection-react/

https://itchallenger.tistory.com/370
