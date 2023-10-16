
## MDX?

**MDX**는 마크다운 문서에 jsx를 활용할 수 있도록 해주는 형식입니다**. 따라서 컴포넌트가 포함된 긴 형식의 콘텐츠를 쉽게 작성할 수 있습니다.

## Nextjs에 markdown 적용하기

[next-mdx-remote](https://www.npmjs.com/package/next-mdx-remote)를 활용하여  markdown 파일을 HTML 코드로 변환합니다. 추가로 markdown 파일에서 커스텀 컴포넌트를 활용할 수 있습니다.  

```
npm i next-mdx-remote
```

해당 라이브러리 또한 앞서 소개한 remark와 rehype을 활용합니다. **Remark는 AST를 사용해 마크다운을 구문 분석하고 컴파일합니다. 구문 강조 표시를 추가하고 목차를 생성하는 등의 작업을 할 수 있는 플러그인으로 구동됩니다. 따라서 remark, rehype 플러그인을 등록하고 커스텀 설정을 할 수 있습니다.**

next-mdx-remote 라이브러리는 **함수와 컴포넌트인 serialize와 `<MDXRemote />`를 노출**합니다. 이 두 컴포넌트는 의도적으로 자체 파일로 분리되어 있습니다**. serialize는 서버 측에서 실행**되도록 설계되었으므로 서버에서/빌드 시 실행되는 getStaticProps 내에서 실행됩니다. **반면에 `<MDXRemote />`는 클라이언트 측, 즉 브라우저에서 실행되도록 설계**되었습니다.

### serialize

- serialize는 MDX 문자열을 분석합니다. 또한 선택적으로 MDX에 직접 전달되는 옵션과 mdx 범위에 포함될 수 있는 범위 개체를 전달할 수도 있습니다. **이 함수는 `<MDXRemote />`에 직접 전달하려는 개체를 반환**합니다.
    
    ```tsx
    // lib/mdx.ts
    import { serialize } from 'next-mdx-remote/serialize';
    
    export const serializeMdx = (source: string) => {
      return serialize(source, {
        mdxOptions: {
          remarkPlugins: [],
          rehypePlugins: [],
          format: 'mdx',
        },
      });
    };
    ```
    

### RemoteMdx

- serialize로 전달된 객체를 MDXRemote가 마운트되면서 HTML로 변환합니다.
    
    ```tsx {6, 11, 19}
    import { serializeMdx } from '@lib/mdx.ts'
    import { MDXRemote } from 'next-mdx-remote'
    
    import Test from '../components/test'
    
    const components = { Test }
    
    export default function TestPage({ source }) {
      return (
        <div className="wrapper">
          <MDXRemote {...source} components={components} />
        </div>
      )
    }
    
    export async function getStaticProps() {
      // MDX text - can be from a local file, database, anywhere
      const source = 'Some **mdx** text, with a component <Test />'
      const mdxSource = await serializeMdx(source)
      return { props: { source: mdxSource } }
    }
    ```
    
- 또한 **componetns prop을 활용해서 custom한 property를 적용할 수 있습니다.**  특정 태그를 커스텀 컴포넌트로 변경하여 작성하도록 변경했습니다. 이미지의 경우 nextjs에서 제공하는 Image를 pre태그에는 코드를 복사할 수 있는 컴포넌트를 추가했습니다.
    
    ```tsx
    const mdxComponents = {
      img: ({ src, alt, ...props }: { src: string; alt: string }) => {
        return (
          <Image layout="responsive" alt={alt} src={src} width={100} height={100} />
        );
      },
      pre: CodeBlock,
    };
    ```
    

### Style

- mdx 구문 분석을 통해 받은 html 정보에는 css 속성이 할당되지 않습니다. 이를 해결하는 가장 쉬운 방법은 tailwindcss의 [**@tailwindcss/typography](https://tailwindcss.com/docs/typography-plugin)를 활용하는 것입니다.**
    
    ```tsx {10}
    //tailwind.config.js
    /** @type {import('tailwindcss').Config} */
    module.exports = {
      content: [
        './src/pages/**/*.{js,ts,jsx,tsx,mdx}',
        './src/components/**/*.{js,ts,jsx,tsx,mdx}',
        './src/app/**/*.{js,ts,jsx,tsx,mdx}',
      ],
      theme: { ****},
      plugins: [require('@tailwindcss/typography')],
    };
    ```
    
    ```tsx
    // mdx에 css 적용
    // 상위 jsx태그를 활용하여 적용해야 합니다.
    <section className="prose lg:prose-xl md:prose-lg sm:prose-base prose-slate dark:prose-invert  w-full max-w-3xl">
      <Content components={mdxComponents} />
    </section>
    ```
    

## Contentlayer

nextjs에서 mdx을 사용하는 다른 방법은 라이브러리 [Contentlayer](https://contentlayer.dev/) 를 활용하는 것입니다.  Contentlayer는 Next.js와 함께 사용할 수 있는 **정적 컨텐츠 관리 도구**입니다.

Contentlayer의 가장 큰 장점은 타입스크립트를 위한 기능입니다. **Contentlayer는 콘텐츠를 데이터로 변환할 때 해당 데이터에 대한 유형 정의도 생성**합니다. 

1. **필요 라이브러리를 설치합니다.** 

```
npm install contentlayer next-contentlayer date-fns
```

2. **typescript 설정**

```json {3,5,13}
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "contentlayer/generated": ["./.contentlayer/generated"]
    }
  },
  "include": [
    "next-env.d.ts",
    "**/*.ts",
    "**/*.tsx",
    ".next/types/**/*.ts",
    ".contentlayer/generated"
  ]
}
```

3. **ignore Build Output**

```
.contentlayer
```

4. **Add Contentlayer Config**
- 커스텀한 타입 설정과 url 설정등을 수정할 수 있습니다.
    
    ```jsx
    // contentlayer.config.ts
    import { defineDocumentType, makeSource } from 'contentlayer/source-files'
    
    const Post = defineDocumentType(() => ({
      name: 'Post',
      filePathPattern: `**/*.mdx`,
      contentType: 'mdx',
      fields: {
        title: {
          type: 'string',
          description: 'The title of the post',
          required: true,
        },
        date: {
          type: 'date',
          description: 'The date of the post',
          required: true,
        },
        id: {
          type: 'string',
          description: 'the id of the post',
          required: true,
        },
        tag: {
          type: 'list',
          description: 'the tags of the post',
          required: true,
          of: { type: 'string' },
        },
        brand: {
          type: 'string',
          description: 'the type of text',
          required: true,
        },
        description: {
          type: 'string',
          description: 'description about each mdx',
          required: false,
        },
        carousel: {
          type: 'string',
          description: '글의 케러셀 포함 여부를 결정',
          required: false,
        },
      },
      computedFields: {
        url: {
          type: 'string',
          resolve: doc => {
            if (doc._raw.flattenedPath.includes('project')) {
              return `${doc._raw.flattenedPath.replace('project/', '')}`;
            }
            return `${doc._raw.flattenedPath}`;
          },
        },
      },
    }));
    ```
    
- 다른 mdx,md 변환 라이브러리와 같이 remark와 rehype을 활용합니다. 이를통해 필요한 플러그인을 사용할 수 있습니다.
    
    ```jsx
    import rehypePrettyCode from 'rehype-pretty-code';
    import rehypeAutolinkHeadings from 'rehype-autolink-headings';
    import rehypeSlug from 'rehype-slug';
    import remarkGfm from 'remark-gfm';
    import rehypeImgSize from 'rehype-img-size';
    
    export default makeSource({
      contentDirPath: 'posts',
      documentTypes: [Post],
      mdx: {
    		//support GFM(autolink literals, footnotes, strikethrough, tables, tasklists)
        remarkPlugins: [remarkGfm], 
        rehypePlugins: [
    			// plugin to add `id` attributes to headings(h1,h2,h3..)
          rehypeSlug,
          [
    				//마크다운 또는 MDX를 위한 아름다운 코드 블록을 제공합니다.
            // 코드 블록 제공 위외에 코드 hightlight 기능 또한 제공하고 있습니다. 
            rehypePrettyCode,
            {
              theme: 'github-dark', // 코드작성시 적용할 테마
            },
          ],
          [
    				//제목(h1,h2)에서 다시 자신으로 연결되는 링크를 추가할 수 있습니다.
            rehypeAutolinkHeadings,
            {
              properties: {
                className: ['anchor'],
                ariaLabel: 'anchor',
              },
            },
          ],
          // @ts-ignore
    			//플러그인을 사용하여 로컬 이미지 크기 속성을 이미지 태그에 설정할 수 있습니다.
          [rehypeImgSize, { dir: 'public' }],
        ],
      },
    });
    ```
    
    - 이중 rehypePlugins의 다양한 기능을 통해 hlight 기능 등을 적용할 수 있습니다. 공식 문서 등을 참고하면 확인할 수 있습니다. [https://rehype-pretty-code.netlify.app/](https://rehype-pretty-code.netlify.app/)

## Reference

[Rehype Pretty Code](https://rehype-pretty-code.netlify.app/)

[Getting Started – Contentlayer](https://contentlayer.dev/docs/getting-started-cddd76b7)

[@tailwindcss/typography - Tailwind CSS](https://tailwindcss.com/docs/typography-plugin)

[Next.js mdx plugin](https://bepyan.github.io/blog/nextjs-blog/3-mdx-plugin)

[Markdown/MDX with Next.js](https://nextjs.org/blog/markdown)