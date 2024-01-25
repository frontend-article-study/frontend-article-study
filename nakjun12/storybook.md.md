# storybook

![Untitled](storybook%2011ab7e6bcf53411cb663c002329c3018/Untitled.png)

**가이드라인**

[https://storybook.js.org/tutorials/intro-to-storybook/react/ko/test/](https://storybook.js.org/tutorials/intro-to-storybook/react/ko/test/)

**피그마에서 말하는 개발자를 위한 디자인 시스템** 

[https://storybook.js.org/tutorials/design-systems-for-developers/react/ko/introduction/](https://storybook.js.org/tutorials/design-systems-for-developers/react/ko/introduction/)

**React CSS 라이브러리 연동하는 법**

[https://storybook.js.org/docs/get-started/setup](https://storybook.js.org/docs/get-started/setup)

**정적 자산 제공하는 법**

[https://storybook.js.org/docs/configure/images-and-assets](https://storybook.js.org/docs/configure/images-and-assets)

**스토리 작성하는 법** 

[https://storybook.js.org/docs/writing-stories#working-with-react-hooks](https://storybook.js.org/docs/writing-stories#working-with-react-hooks)

**msw 1 버전 지원 (업로드 3개월 전)**

[https://github.com/mswjs/msw-storybook-addon#configuring-msw](https://github.com/mswjs/msw-storybook-addon#configuring-msw)

[https://storybook.js.org/addons/storybook-addon-react-router-v6/](https://storybook.js.org/addons/storybook-addon-react-router-v6/)

### Storybook 설치 및 기본 설정

[https://storybook.js.org/docs/get-started/install](https://storybook.js.org/docs/get-started/install)

1. **Storybook 설치하기:**
먼저, 프로젝트 디렉토리로 이동한 후, `pnpm`을 사용하여 Storybook을 설치합니다.
    
    ```bash
    pnpm add @storybook/react -D   # Storybook을 개발 의존성으로 설치
    ```
    
2. **Storybook 초기 설정하기:**
Storybook은 초기 설정 스크립트를 제공하여 쉽게 설정할 수 있습니다. 이 스크립트는 필요한 설정 파일과 예제 스토리들을 생성해줍니다.
    
    ```bash
    npx sb init   # Storybook 초기 설정 스크립트 실행
    ```
    
    스크립트 실행 후, 진행 여부를 확인하는 질문에 'y'를 입력합니다. 
    
    ![Untitled](storybook%2011ab7e6bcf53411cb663c002329c3018/Untitled%201.png)
    
    필요한 경우, 추가적으로 ESLint 설정을 진행할 수 있습니다.
    
    **자동화된 설정과 안내:**
    
    Storybook은 설치 과정이 자동화되어 있어 사용자에게 편리합니다. 설치 완료 후, Storybook은 사용 방법에 대한 가이드라인을 제공합니다.
    
    ![Untitled](storybook%2011ab7e6bcf53411cb663c002329c3018/Untitled%202.png)
    
    ![Untitled](storybook%2011ab7e6bcf53411cb663c002329c3018/Untitled%203.png)
    
    위와 같이 친절하게 설명해주니 처음해본다면 꼭 하면 좋을 것 같습니다.
    
3. **Storybook 스크립트 추가하기:**`package.json` 파일에 Storybook을 실행하는 스크립트를 추가할 수 있습니다. 
    
    ```json
    {
      "scripts": {
        "storybook": "start-storybook -p 6006",
        "build-storybook": "build-storybook"
      }
    }
    
    ```
    
    `**storybook**` : **`start-storybook`**은 Storybook을 실행하는 명령어입니다. -p는 포트 번호입니다.
    
    `**build-storybook` : `build-storybook`** 명령어는 개발 환경이 아닌, 프로덕션 환경에서 사용할 수 있도록 Storybook의 정적 버전을 생성합니다.
    
4. **커스텀 설정 변경 (선택사항):**
필요에 따라 `.storybook` 폴더 내의 `main.js`, `preview.js` 등의 설정 파일을 수정하여 Storybook을 커스터마이즈할 수 있습니다.

---

### **리액트(React)를 위한 스토리북(Storybook) 튜토리얼**

[https://storybook.js.org/tutorials/intro-to-storybook/react/ko/get-started/](https://storybook.js.org/tutorials/intro-to-storybook/react/ko/get-started/)

스토리 북은 일부 한국어를 지원합니다.

1. **간단한 컴포넌트 활용하기**

```jsx
//Button.jsx

import PropTypes from "prop-types";
import "./button.css";

/**
 * Primary UI component for user interaction
 */
export const Button = ({ primary, backgroundColor, size, label, ...props }) => {
  const mode = primary
    ? "storybook-button--primary"
    : "storybook-button--secondary";
  return (
    <button
      type="button"
      className={["storybook-button", `storybook-button--${size}`, mode].join(
        " "
      )}
      style={backgroundColor && { backgroundColor }}
      {...props}>
      {label}
    </button>
  );
};
```

위와 같은 button 컴포넌트가 있습니다.

props로 `{primary, backgroundColor, size, label, ...props}`를 받습니다.

### Controls

![Untitled](storybook%2011ab7e6bcf53411cb663c002329c3018/Untitled%204.png)

이는 그대로 `Controls` 탭으로 설정합니다.

![Untitled](storybook%2011ab7e6bcf53411cb663c002329c3018/Untitled%205.png)

Docs가 있는데 여기에는 `controls (arguments)` 에 관한 설명이 적혀있습니다.

### Control Docs 설정

![Untitled](storybook%2011ab7e6bcf53411cb663c002329c3018/Untitled%206.png)

이는 js 같은 경우 propTypes으로 문서를 작성할 수 있습니다.

```jsx

Button.propTypes = {
  /**
   * Is this the principal call to action on the page
   */
  primary: PropTypes.bool,
  /**
   * What background color to use
   */
  backgroundColor: PropTypes.string,
  /**
   * How large should the button be?
   */
  size: PropTypes.oneOf(["small", "medium", "large"]),
  /**
   * Button contents
   */
  label: PropTypes.string.isRequired,
  /**
   * Optional click handler
   */
  onClick: PropTypes.func
};

Button.defaultProps = {
  backgroundColor: null,
  primary: false,
  size: "medium",
  onClick: undefined
};
```

### 스토리북 파일 작성

![Untitled](storybook%2011ab7e6bcf53411cb663c002329c3018/Untitled%207.png)

Doc 하단에 보면 여러 스토리가 보입니다.

위와 같이 여러 스타이별로 컴포넌트를 만드는 방법을 보겠습니다.

**default 설정**

```jsx
//Button.stories.js

import { Button } from "./Button";

// More on how to set up stories at: https://storybook.js.org/docs/writing-stories#default-export
export default {
  title: "Example/Button",
  component: Button,
  parameters: {
    // Optional parameter to center the component in the Canvas. More info: https://storybook.js.org/docs/configure/story-layout
    layout: "centered"
  },
  // This component will have an automatically generated Autodocs entry: https://storybook.js.org/docs/writing-docs/autodocs
  tags: ["autodocs"],
  // More on argTypes: https://storybook.js.org/docs/api/argtypes
  argTypes: {
    backgroundColor: { control: "color" }
  }
};
```

스토리북 파일을 들어가보면, default로 내보내는 것이 기본 설정입니다.

### title

먼저 `title`은 아래 그림처럼 특정 그룹을 의미합니다.

![Untitled](storybook%2011ab7e6bcf53411cb663c002329c3018/Untitled%208.png)

title에서 Example을 제거해보겠습니다

```jsx
title : "Button"
```

![Untitled](storybook%2011ab7e6bcf53411cb663c002329c3018/Untitled%209.png)

EXAMPLE을 제거하자 해당 그룹에 벗어나 보여집니다.

```jsx
title: "SD/Button",
```

![Untitled](storybook%2011ab7e6bcf53411cb663c002329c3018/Untitled%2010.png)

앞에 SD를 붙혔더니 해당 그룹이 다시 생깁니다.

Title은 위처럼 파일, 폴더 별로 그룹을 나누어줍니다.

### 간단 설명

```jsx
export default {
  title: 'Example/Button',
  component: Button, // 이 설정은 Storybook에 어떤 React 컴포넌트를 보여줄 것인지를 정의
  parameters: {
    // 'parameters'는 Storybook의 렌더링 및 동작 방식을 커스터마이즈하기 위한 설정입니다.
    layout: 'centered', // 'layout' 파라미터는 컴포넌트를 캔버스 중앙에 배치합니다.
  },
  tags: ['autodocs'], //상단에 있는 propsTypes를 doc로 만들어줍니다, 
  argTypes: {
    // 'argTypes'는 Storybook에서 컨트롤할 수 있는 각 prop의 동작을 정의합니다.
    backgroundColor: { control: 'color' }, 
		// 'backgroundColor' prop을 color picker로 조절할 수 있도록 설정합니다.
  },
};
```

자세한 내용은 storybook이 링크로 달아놨으니 궁금하시면 참고하시면 될 것 같습니다.

### 링크를 포함한 코드

```jsx
// More on how to set up stories at: https://storybook.js.org/docs/writing-stories#default-export
export default {
  title: "SD/Button",
  component: Button,
  parameters: {
    // Optional parameter to center the component in the Canvas. More info: https://storybook.js.org/docs/configure/story-layout
    layout: "centered"
  },
  // This component will have an automatically generated Autodocs entry: https://storybook.js.org/docs/writing-docs/autodocs
  tags: ["autodocs"],
  // More on argTypes: https://storybook.js.org/docs/api/argtypes
  argTypes: {
    backgroundColor: { control: "color" }
  }
};
```

---

### 컴포넌트 스토리 만드는 법

다음으로 컴포넌트 스토리를 만드는 법을 보겠습니다.

```jsx
// Button.stories.js
export default {
  title: 'Example/Button',
  component: Button, // 이 설정은 Storybook에 어떤 React 컴포넌트를 보여줄 것인지를 정의
  parameters: {
    // 'parameters'는 Storybook의 렌더링 및 동작 방식을 커스터마이즈하기 위한 설정입니다.
    layout: 'centered', // 'layout' 파라미터는 컴포넌트를 캔버스 중앙에 배치합니다.
  },
  tags: ['autodocs'], //상단에 있는 propsTypes를 doc로 만들어줍니다, 
  argTypes: {
    // 'argTypes'는 Storybook에서 컨트롤할 수 있는 각 prop의 동작을 정의합니다.
    backgroundColor: { control: 'color' }, 
		// 'backgroundColor' prop을 color picker로 조절할 수 있도록 설정합니다.
  },
};

export const Primary = {
  args: {
    primary: true,
    label: "Button",
    word: "have"
  }
};

export const Secondary = {
  args: {
    label: "Button"
  }
};

export const Large = {
  args: {
    size: "large",
    label: "Button"
  }
};
```

위처럼 args를 정해주고 내보내면 controls에 해당 값이 설정되며 아래처럼 나옵니다.

![Untitled](storybook%2011ab7e6bcf53411cb663c002329c3018/Untitled%2011.png)

**Secondary**

Secondary는 현재 label이 button으로 설정되어있기때문에 label은 button 이며 다른 것은 default와 동일합니다.

![Untitled](storybook%2011ab7e6bcf53411cb663c002329c3018/Untitled%2012.png)

**컴포넌트 스토리 설정 재사용하기**

```jsx
export const Primary: Story = {
  args: {
    primary: true,
    label: 'Button',
  },
};

export const PrimaryLongName: Story = {
  args: {
    ...Primary.args,
    label: 'Primary with a really long name',
  },
};
```

---

### 컴포넌트 스토리 활용하여 args 작성하기

```jsx
import { ButtonGroup } from '../ButtonGroup';

//👇 Imports the Button stories
import * as ButtonStories from './Button.stories';

export default {
  component: ButtonGroup,
};

export const Pair = {
  args: {
    buttons: [{ ...ButtonStories.Primary.args }, { ...ButtonStories.Secondary.args }],
    orientation: 'horizontal',
  },
};
```

위처럼 * 사용하여 가져올 수 있습니다.

### 두 개 이상의 구성 요소가 있는 경우

```jsx
import { List } from './List';
import { ListItem } from './ListItem';

export default {
  component: List,
};

export const Empty = {};

/*
 *👇 Render functions are a framework specific feature to allow you control on how the component renders.
 * See https://storybook.js.org/docs/api/csf
 * to learn how to use render functions.
 */
export const OneItem = {
  render: (args) => (
    <List {...args}>
      <ListItem />
    </List>
  ),
};

export const ManyItems = {
  render: (args) => (
    <List {...args}>
      <ListItem />
      <ListItem />
      <ListItem />
    </List>
  ),
};
```

**자식요소 컴포넌트 스토리 활용하기**

컴포넌트 스토리를 통해 args까지 사용할 수 있습니다.

```jsx
import React from 'react';

import { List } from './List';
import { ListItem } from './ListItem';

//👇 We're importing the necessary stories from ListItem
import { Selected, Unselected } from './ListItem.stories';

export default {
  component: List,
};

/*
 *👇 Render functions are a framework specific feature to allow you control on how the component renders.
 * See https://storybook.js.org/docs/api/csf
 * to learn how to use render functions.
 */

export const ManyItems = {
  render: (args) => (
    <List {...args}>
      <ListItem {...Selected.args} />
      <ListItem {...Unselected.args} />
      <ListItem {...Unselected.args} />
    </List>
  ),
};
```

### 컴포넌트 스토리 재사용법

```jsx
import PageLayout from './PageLayout';
import Document from './Document';
import SubDocuments from './SubDocuments';
import DocumentHeader from './DocumentHeader';
import DocumentList from './DocumentList';

export interface DocumentScreenProps {
  user?: {};
  document?: Document;
  subdocuments?: SubDocuments[];
}

export function DocumentScreen({ user, document, subdocuments }: DocumentScreenProps) {
  return (
    <PageLayout user={user}>
      <DocumentHeader document={document} />
      <DocumentList documents={subdocuments} />
    </PageLayout>
  );
}
```

위처럼 여러 컴포넌트를 import 하고 props로 내려주는 상황이 있습니다.

아래처럼 사용할 수 있습니다.

**자식 컴포넌트 스토리**

```jsx
// DocumentHeader.stories.js

.
const meta: Meta<typeof DocumentHeader> = {
  title: 'Components/DocumentHeader',
  component: DocumentHeader,
};
export default meta;

type Story = StoryObj<typeof DocumentHeader>;

// Simple 스토리는 DocumentHeader 컴포넌트의 기본적인 사용 예시를 보여줍니다.
export const Simple: Story = {
  args: {
    title: 'Sample Document Title', // 예시로 제공되는 제목
    user: 'John Doe', // 예시 사용자 이름
  },
};
```

**부모 컴포넌트 스토리**

```jsx
// Replace your-framework with the name of your framework
import type { Meta, StoryObj } from '@storybook/your-framework';

import { DocumentScreen } from './YourPage';

// 👇 Imports the required stories
import * as PageLayout from './PageLayout.stories';
import * as DocumentHeader from './DocumentHeader.stories';
import * as DocumentList from './DocumentList.stories';

const meta: Meta<typeof DocumentScreen> = {
  component: DocumentScreen,
};

export default meta;
type Story = StoryObj<typeof DocumentScreen>;

export const Simple: Story = {
  args: {
    user: PageLayout.Simple.args.user,
    document: DocumentHeader.Simple.args.document,
    subdocuments: DocumentList.Simple.args.documents,
  },
};
```

위처럼 import 해서 재사용할 수 있습니다.

```jsx
import type { Meta, StoryObj } from '@storybook/react';

import { List } from './List';
import { ListItem } from './ListItem';

//👇 Imports a specific story from ListItem stories
import { Unchecked } from './ListItem.stories';

const meta: Meta<typeof List> = {
  /* 👇 The title prop is optional.
   * Seehttps://storybook.js.org/docs/configure/#configure-story-loading
   * to learn how to generate automatic titles
   */
  title: 'List',
  component: List,
};

export default meta;
type Story = StoryObj<typeof List>;

//👇 The ListTemplate construct will be spread to the existing stories.
const ListTemplate: Story = {
  render: ({ items, ...args }) => {
    return (
      <List>
        {items.map((item) => (
          <ListItem {...item} />
        ))}
      </List>
    );
  },
};

export const Empty = {
  ...ListTemplate,
  args: {
    items: [],
  },
};

export const OneItem = {
  ...ListTemplate,
  args: {
    items: [{ ...Unchecked.args }],
  },
};
```

### 크로마틱 배포 및 활용하기

크로마틱(Chromatic)을 사용하여 스토리북을 배포하고 관리하는 방법을 단계별로 정리해보겠습니다.

1. **크로마틱 설치 및 설정**

```bash
yarn add -D chromatic
```

1. **크로마틱 계정 설정**
    - GitHub 계정을 사용하여 크로마틱에 로그인합니다. 이 단계에서는 크로마틱이 등록 권한만 요청합니다.
        
        [chromatic-setup-learnstorybook.mp4](storybook%2011ab7e6bcf53411cb663c002329c3018/chromatic-setup-learnstorybook.mp4)
        
2. **GitHub 레포지토리 선택**
    - 크로마틱에서 'Choose GitHub repo'를 클릭하여 프로젝트에 연결할 GitHub 레포지토리를 선택합니다.
3. **프로젝트 토큰 복사**
    - 크로마틱에서 생성된 고유 프로젝트 토큰(`project-token`)을 복사합니다.
4. **스토리북 빌드 및 배포**
    - 다음 명령어를 실행하여 스토리북을 빌드하고 크로마틱에 배포합니다.`<project-token>` 부분에 복사한 프로젝트 토큰을 붙여넣습니다.
        
        ```bash
        yarn chromatic --project-token=<project-token>
        ```
        
        ![Untitled](storybook%2011ab7e6bcf53411cb663c002329c3018/Untitled%2013.png)
        
5. **스토리북 링크 공유**
    - 배포된 스토리북의 링크(예: `https://random-uuid.chromatic.com`)를 팀과 공유하여 피드백을 받습니다.
6. **자동 배포 설정**
    - 스토리북을 수동으로 배포하는 대신, 코드를 푸시할 때마다 자동으로 최신 버전의 스토리북을 배포하기 위해 GitHub Actions를 설정합니다.
7. **GitHub Actions 설정**
    - 프로젝트의 기본 폴더에 `.github/workflows` 디렉토리를 생성하고, 그 안에 `chromatic.yml` 파일을 만듭니다.
    - `chromatic.yml` 파일에는 다음과 같은 내용을 포함합니다.
    이 스크립트는 GitHub에 코드를 푸시할 때마다 자동으로 스토리북을 빌드하고 크로마틱에 배포합니다.
        
        ```yaml
        # Workflow name
        name: 'Chromatic Deployment'
        
        # Event for the workflow
        on: push
        
        # List of jobs
        jobs:
          test:
            # Operating System
            runs-on: ubuntu-latest
            # Job steps
            steps:
              - uses: actions/checkout@v1
              - run: yarn
              #👇 Adds Chromatic as a step in the workflow
              - uses: chromaui/action@v1
                # Options required for Chromatic's GitHub Action
                with:
                  #👇 Chromatic projectToken
                  projectToken: ${{ secrets.CHROMATIC_PROJECT_TOKEN }}
                  token: ${{ secrets.GITHUB_TOKEN }}
        
        ```
        
8. **GitHub Secrets 설정**
    - GitHub 리포지토리의 'Settings > Secrets'에 `CHROMATIC_PROJECT_TOKEN`을 추가합니다. 이렇게 하면 `project-token`을 직접 입력할 필요 없이 안전하게 관리할 수 있습니다.

![Untitled](storybook%2011ab7e6bcf53411cb663c002329c3018/Untitled%2014.png)

이 과정을 통해 크로마틱을 사용하여 스토리북을 빌드하고 자동으로 배포하는 환경을 설정할 수 있습니다.

**크로마틱 비교 기능**

[visual-test-hero.mp4](storybook%2011ab7e6bcf53411cb663c002329c3018/visual-test-hero.mp4)

**크로마틱 리뷰 기능**

[ui-review-hero.mp4](storybook%2011ab7e6bcf53411cb663c002329c3018/ui-review-hero.mp4)