---
title: Storybook 마크다운으로 작성하기
toc: true
widgets:
  - type: toc
    position: right
  - type: categories
    position: right
  - type: tags
    position: right
  - type: adsense
    position: right
    client_id: ca-pub-5445993070474035
    slot_id: ''
date: 2020-05-17 16:23:45
categories:
  - Storybook
tags:
  - Storybook
  - markdown
  - mdx
  - addon-docs
---

우선 이 글에서는 리액트, 타입스크립트를 사용하는 스토리북 세팅이 다 갖춰졌다는 것을 가정하고 써보려고 한다.
스토리북은 단위 컴포넌트 테스트를 가능하게 해주고 협업에 있어 엄청난 장점을 갖는다고 생각한다.
하지만, 스토리북에서 제공해 주는 Canvas 만 사용했었다. 이로 인해 디테일한 설명, Props 의 type 등 불편한 점이
많았다. 이 문제를 어떻게 해결할 수 있을까 라는 고민을 하며 찾아보니 마크다운 문법으로 스토리북을 작성할 수 있다는
것을 알게 되었다.

<!-- more -->

## addon-docs

우선 .storybook 폴더에 main.js 파일이 존재한다. 이 파일은 스토리북에 대한 설정을 잡아주는 파일이라고 생각하면 된다.

`addons`, `webpack` 등의 설정을 해줄 수 있다.

```javascript
const path = require('path');

module.exports = {
    addons: [
        '@storybook/addon-knobs/register',
        '@storybook/addon-actions/register'
    ],

    webpackFinal: (config) => {
        config.module.rules.push({
            test: /\.(ts|tsx)$/,
            use: [
                {
                    loader: require.resolve('babel-loader'),
                    options: {
                        presets: [['react-app', { flow: false, typescript: true }]],
                    },
                },
                {
                    loader: require.resolve("react-docgen-typescript-loader"),
                    options: {
                        tsconfigPath: path.join(__dirname, "../tsconfig.json"),
                    },
                },
            ],
        });

        config.resolve.extensions.push('.js', '.ts', '.tsx');
        return config;
    }
};
```

기존 main.js 의 설정이었다. 사실 스토리북에서 마크다운으로 작성하기 위한 설정은 쉬웠다.

`@storybook/addon-docs` 를 install 해주고 `addons` 에 넣어주면 거의 끝났다고 보면 된다.

```shell script
npm i -D @storybook/addon-docs
```

```javascript
module.exports = {
    addons: [
        '@storybook/addon-docs/preset',
        '@storybook/addon-knobs/register',
        '@storybook/addon-actions/register'
    ],
    // ...
};
```

이제 마지막 한가지만 더 추가해주면 된다. 본인은 .storybook 폴더 내부에 config.js 파일이 존재하고 해당 파일에서
`path` 와 `filter` 를 설정해준다. 이때 mdx 도 `filter` 파일 조건에 추가만 해주면 된다. 

```javascript
configure(require.context('../`stories`', true, /\.stories\.(mdx|tsx)$/), module);
```

## mdx 작성

이제 기존 storybook 파일을 .jsx 또는 .tsx 에서 .mdx 로 바꾸면 된다.
addon-docs 문서를 보면 다른 좋은 기능들이 많지만 본인은 거의 아래의 것들만 사용했다.

- Meta
- Title
- Subtitle
- Story
- Preview
- Props

### Meta

```jsx harmony
<Meta title="components/Button" component={Button} decorators={[withKnobs]} />
```

- `title` 은 스토리북 상의 경로에 해당한다. component 라는 폴더 밑에 Button 을 생성한다.
- `component` 는 import 한 리액트 컴포넌트를 전달해주면 된다.
- `decorators` 는 여러개 넘겨줄 수 있는데 addon-knobs 를 사용하기 때문에 withKnobs 만 넘겨주었다.

### Title

```jsx harmony
<Title>Button</Title>
```

`Title` 은 해당 문서의 제목 즉, h1 태그가 된다. (#)

### Subtitle

```jsx harmony
<Subtitle>Button Component</Subtitle>
```


`Subtitle` 은 h2 태그에 해당한다. (##)

### Preview, Story

`Preview` 와  `Story` 는 거의 세트로 사용했다. `Preview` 를 사용하면 문서에 영역이 생기는데 이 영역에 
`Story`가 들어가게 된다.

```jsx harmony
<Preview>
    <Story name="default">
        <Button/>
    </Story>
</Preview>
```

### Props

마지막으로 Props 는 컴포넌트에 `defaultProps` 가 존재하거나 `interface` 가 정의되어 있다면 해당 컴포넌트의
타입과 필수값들을 테이블로 나열해 준다.

```jsx harmony
<Props of={Button}/>
```


## 스토리북에서 리액트 문법 사용하기

추가로 스토리북에서 리액트의 기능 `useState`, `useEffect` 또는 함수를 만들어서 사용할 수 없을까에 대한
고민을 하게 되었고 역시나 사용할 수 있었다.

Story 태그에 js 문법을 이용하여 함수를 사용하고 컴포넌트를 리턴해주면 된다. 

```jsx harmony
<Preview>
    <Story name="default">
        {() => {
            const [state, setState] = useState([]);
            const handleClick = (e) => {
                //...
            }
            return (
                <Button
                    handleClick={handleClick}
                />
            )
        }}
    </Story>
</Preview>
```

아직 풀리지 않는 이상한 점도 있다. Story 태그에 함수를 사용할때 공백줄이 있으면 에러가 발생한다.
이 문제에 대해 구글링 해본 결과 이슈로 올라오고 있는데 아직까지 해결되지는 않은것 같다.

즉 이 코드는 동작하지만,
```javascript
const [state, setState] = useState([]);
const handleClick = (e) => {
    //...
}
```

이 코드에서는 에러가 발생한다.
```javascript
const [state, setState] = useState([]);

const handleClick = (e) => {
    //...
}
```

[@storybook/addon-docs](https://www.npmjs.com/package/@storybook/addon-docs)
