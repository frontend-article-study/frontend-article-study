# Next.js blog에 markdown

# MarkDown 적용하기

서버리스 블로그를 NextJs를 활용해서 만들고 있습니다. 이러한 과정에서 각 post를 markdown 형식으로 우선 만들고 있었습니다. **Markdown은 텍스트 기반의 마크업언어로 2004년 존그루버에 의해 만들어졌으며 쉽게 쓰고 읽을 수 있으며 HTML로 변환이 가능하다. 특수기호와 문자를 이용한 매우 간단한 구조의 문법을 사용하여 웹에서도 보다 빠르게 컨텐츠를 작성하고 보다 직관적으로 인식할 수 있습니다**. 

Markdown으로 포스트를 관리하는 이유는 쉽게 html 태그로 변환할 수 있기 때문입니다. 이러한 점에 착안해서 Markdown으로 포스트를 관리하였고 라이브러리를 활용해서 Markdowm 문서를 html로 쉽게 변환하고 디자인을 수정하기로 했습니다. 이때 사용한 라이브러리는 [**리엑트 마크다운**](https://github.com/remarkjs/react-markdown) 입니다.  사용 방법은 매우 간단합니다. `**ReactMarkdown**` 컴포넌트를 통해 md 파일의 content를 전달하면 모든 자동으로 html 형식에 맞게 파싱이 진행됩니다.

```
npm i next-mdx-remote
```

```jsx
import React from 'react'
import ReactMarkdown from 'react-markdown'
import ReactDom from 'react-dom'
import remarkGfm from 'remark-gfm'

ReactDom.render(
  <ReactMarkdown remarkPlugins={[[remarkGfm, {singleTilde: false}]]}>
    This ~is not~ strikethrough, but ~~this is~~!
  </ReactMarkdown>,
  document.body
)
```

## **Unified, Remark, Rehype**

[**remark](https://remark.js.org/)는 마크다운을 처리할 때 사용하는 라이브러리**이고, **[rehype](https://github.com/rehypejs/rehype)은 HTML을 처리할 때 사용하는 라이브러리**입니다. 이 두 개의 라이브러리는 다양한 형식의 텍스트의 범용 처리를 지원하고 있는 [**unified](https://unifiedjs.com/)라는 상위 프로젝트에 소속**되어 있습니다.

**unified 라이브러리를 사용하면 Markdown이든 HTML이든 형식에 구애받지 않고 동일한 API(applicaiton programming interface)를 통해서 텍스트를 처리**할 수 있다는 큰 이점이 있습니다. **unified 라이브러리의 API는 `use()` 함수를 연쇄적으로 호출하여 원하는 작업을 순차적으로 처리하도록 설계**되어 있습니다. 

`react-markdown`의 `package.json`을 확인해보면 undefined를 활용해서 구현되어 있다는 것을 확인할 수 있습니다.

```json
"dependencies": {
    "@types/hast": "^2.0.0",
    "@types/prop-types": "^15.0.0",
    "@types/unist": "^2.0.0",
    "comma-separated-tokens": "^2.0.0",
    "hast-util-whitespace": "^2.0.0",
    "prop-types": "^15.0.0",
    "property-information": "^6.0.0",
    "react-is": "^18.0.0",
    "remark-parse": "^10.0.0",
    "remark-rehype": "^10.0.0",
    "space-separated-tokens": "^2.0.0",
    "style-to-object": "^0.4.0",
    "unified": "^10.0.0",
    "unist-util-visit": "^4.0.0",
    "vfile": "^5.0.0"
  },
```

- **remark-parse**
    - remark 플러그인을 추가하여 마크다운에서 구문 분석을 지원합니다.
- **remark-rehype**
    - 마크다운을 HTML로 변환.

```jsx
// 활용 예시
import unified from "unified";
import markdown from "remark-parse";
import remark2rehype from "remark-rehype";
// html을 문자열로 반환하는 플러그인
import html from "rehype-stringify";

const mdText = `
# Our Project

Hello, **Markdown**!
`;

const html_text = unified()
  .use(markdown)
  .use(remark2rehype)
  .use(html)
  .processSync(mdText);

console.log(html_text.toString());
```
최종 변환된 텍스트를 출력해보면 다음과 같이 HTML을 얻을 수 있습니다.
html
<h1>Our Project</h1>
<p>Hello, <strong>Markdown</strong>!</p>

```
```

## 추가적으로 활용한 라이브러리

### remark-gfm

remark 플러그인을 추가하여 GFM(리터럴 자동 링크, 각주, 취소선, 표, 작업 목록)을 지원합니다.

### react-syntax-highlighter

[hightlight.js](https://highlightjs.readthedocs.io/en/latest/index.html)를 활용하여 react에 맞는 code highlt 기능을 제공하고 있습니다.

react markdown을 nextJS에서 활용하는 방식은 다음과 같습니다. 

1. **특정 파일의 정보를 가지고 옵니다.** 
    
    ```tsx
    import { promises as fs } from 'fs';
    import path from 'path';
    import GetBlog from '@/server/getBlog';
    import { PostData } from '@/type/type';
    
    export default async function getDetail(url: string): Promise<PostData> {
      const dir = path.join(process.cwd(), 'data', 'posts', `${url}.md`);
      const metadata = await GetBlog().then(posts =>
        posts.find(post => post.path === url),
      );
      if (!metadata) {
        throw new Error(`${url}에 해당하는 포스트를 찾을 수 없습니다.`);
      }
      const content = await fs.readFile(dir, 'utf-8');
      return { ...metadata, content };
    }
    ```
    
2. **react Markdown을 활용하는 Component를 생성합니다.**
    
    ```tsx
    'use client';
    import Image from 'next/image';
    import ReactMarkdown from 'react-markdown';
    import { Prism as SyntaxHighlighter } from 'react-syntax-highlighter';
    import { materialDark } from 'react-syntax-highlighter/dist/esm/styles/prism';
    import remarkGfm from 'remark-gfm';
    
    export default function PostMarkDown({ post }: { post: string }) {
    
      return (
        <ReactMarkdown
          className="prose lg:prose-xl max-w-none"
          remarkPlugins={[remarkGfm]}
          components={{
            code({ node, inline, className, children, ...props }) {
              const match = /language-(\w+)/.exec(className || '');
              return !inline && match ? (
                <SyntaxHighlighter
                  {...props}
                  style={materialDark}
                  language={match[1]}
                  PreTag="div">
                  {String(children).replace(/\n$/, '')}
                </SyntaxHighlighter>
              ) : (
                <code {...props} className={className}>
                  {children}
                </code>
              );
            },
            img: image => (
              <Image
                src={image.src || ''}
                alt={image.alt || ''}
                width={500}
                height={300}
              />
            ),
          }}>
          {post}
        </ReactMarkdown>
      );
    }
    ```
    
3. **필요 페이지에서 두 함수를 활용하여 page를 생성합니다.**
    
    ```tsx
    import { Metadata } from 'next';
    import Image from 'next/image';
    import PostMarkDown from '@/components/PostMarkDown';
    import GetBlog from '@/server/getBlog';
    import getDetail from '@/server/getDetail';
    
    type Prop = {
      params: { slug: string };
    };
    
    export const generateMetadata = async ({ params }: Prop): Promise<Metadata> => {
      const data = await GetBlog();
      const blog = data.find(item => item.path === params.slug);
      return { title: blog?.title, description: blog?.description };
    };
    
    export default async function PostDetailPage({ params }: Prop) {
      const file = await getDetail(params.slug);
    
      return (
        <article>
          <Image
            className="w-full h-1/5"
            src={`/posts/${file.path}.png`}
            alt={file.title}
            width={750}
            height={300}
          />
          <h1 className="font-bold text-4xl my-2">{file.title}</h1>
          <p className="text-xl font-bold">{file.description}</p>
          <div className="w-44 border-2 border-sky-600 my-4"> </div>
          <PostMarkDown post={file.content} />;
        </article>
      );
    }
    ```
    

## Reference

[https://github.com/react-syntax-highlighter/react-syntax-highlighter](https://github.com/react-syntax-highlighter/react-syntax-highlighter)

[Markdown을 HTML로 변환 (unified, remark, rehype)](https://www.daleseo.com/unified-remark-rehype/#unified-remark-rehype)