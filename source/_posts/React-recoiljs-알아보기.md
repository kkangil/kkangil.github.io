---
title: React recoiljs 알아보기
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
date: 2020-05-24 16:51:45
categories:
    - React
tags:
    - react
    - recoil
    - atom
    - selector
---

최근 react 에서 새로운 상태관리 라이브러리를 발표했다. 주요 컨셉으로는 atom,selector 라는 단위를 통해
derived state 를 효과적으로 처리하고 상태의 코드 분할이 가능하게 한다고 한다.

리액트의 기본 설정은 create-react-app 을 사용하려고 한다. react-app 생성 후 recoiljs 를 설치해주면 
사용 준비가 끝난것이다.

```shell script
npm i recoil
```

<!-- more -->

## 구조

아직 나온지 얼마 되지 않은 라이브러리라서 구조화에 대한 내용이 많이 없다. 현재는 src 내부에
`recoil` 이라는 폴더를 만들고 component 단위 별로 파일을 생성하려고 한다.

## RecoilRoot

redux, mobx 등 다른 상태관리 라이브러리에도 provider 가 있듯이 recoil 도 RecoilRoot 라는 hoc 가 존재한다.
`props` 로 `initializeState` 를 전달해 줄 수 있지만 지금은 일단 넘어가려 한다.

create-react-app 으로 생성한 app 기준 index.js 에 App 을 RecoilRoot 로 감싸주면 된다.

```jsx harmony
import {RecoilRoot} from 'recoil';

ReactDOM.render(
  <React.StrictMode>
      <RecoilRoot>
        <App />
      </RecoilRoot>
  </React.StrictMode>,
  document.getElementById('root')
);
```

[RecoilRoot 공식 문서](https://recoiljs.org/docs/api-reference/core/RecoilRoot)

## atom

리액트의 state 와 같다라고 생각해도 된다. 즉, 상태 값이라는 것이다. 기본적으로 `atom` 함수를 실행하고
이때 인자로 option 을 넘겨준다. option 중 `key` 와 `default` 는 필수 값이다.

key 는 unique 한 id 여야 하고 default 는 이름에서 유추할 수 있듯이 해당 상태 값의 초기값이다.

count component 를 만들면서 확인해 보자.

우선 recoil 폴더에 count.js 파일을 만든 후 아래와 같이 atom state 를 생성해 줬다.

```javascript
import {atom} from "recoil";

export const countState = atom({
    key: "countState",
    default: 0
});
```

이후 Count Component 를 아래와 같이 생성했다.

```jsx harmony
import React from "react";
import {useRecoilState} from "recoil";
import {countState} from "../recoil/count";

function Counter() {
    const [count, setCount] = useRecoilState(countState);
    const incrementByOne = () => setCount(count + 1);

    return (
        <div>
            Count: {count}
            <br />
            <button onClick={incrementByOne}>Increment</button>
        </div>
    );
}

export default Counter;
```

위의  `useRecoilState` 훅은 밑에서 자세히 설명하도록 하겠다.

## selector

`selector` 도 `atom` 과 마찬가지로 값으로 쓰인다. 차이점은 atom 은 오로지 현재 값만 가져오고 
setState 할때도 넘겨주는 값만을 사용할 수 있다. 하지만 selector 는 option 에 `get` 과 `set` 을 넘겨줘서
사용할 수 있다.

option 에 역시 key 는 필수이며 unique 해야한다. get 또한 필수로 넘겨줘야 하며 set 은 optional 이다.

```javascript
export const countEvenState = selector({
    key: "countEvenState",
    get: ({get}) => get(countState) % 2 === 0,
    set: ({set}, newValue) => set(countState, newValue)
});
```

먼저 selector recoil state 를 생성했다. 위 값을 불러올때 짝수면 true, 홀수면 false 를 가져온다.

`get` 은 함수를 전달해줘야한다. 이때 인자 객체에 get 이라는 함수가 있는데, 이 함수를 사용하여 상태값을
불러와 사용한다. 이때 get 에 전달해줘야 하는 값은 recoil state 여야 한다.

위의 예제에서는 `set` 이 필요 없을 수 있으나, 어떤 식으로 사용해야 하는지 알 수 있게 추가한 것이다.
set 은 첫번째 인자로 객체, 두번째 인자로 새로운 값이 넘어오고, 첫번째 인자 객체의 set 을 사용하여
state 값을 바꾼다. 이때 첫번째 인자는 역시 recoil state 여야 하고 두번째 인자로 바뀔 값을 넘겨준다.

```jsx harmony
function Counter() {
    const [count, setCount] = useRecoilState(countState);
    const [evenCount, setEvenCount] = useRecoilState(countEvenState);
    const incrementByOne = () => setCount(count + 1);
    const incrementByOneEvenCount = () => setEvenCount(count + 1);

    return (
        <div>
            Count: {count}
            <br />
            <button onClick={incrementByOne}>Increment</button>
            <br />
            Even Count: {evenCount ? '짝수' : '홀수'}
            <br />
            <button onClick={incrementByOneEvenCount}>Even Increment</button>
        </div>
    );
}

```

Count component 를 위와 같이 변경하였다. Even Increment 버튼을 클릭하면 count 의 값도 같이 바뀌는 것을
확인할 수 있다. set 에서 count 의 값을 변경해 주기 때문이다.

## recoil hooks

### recoil state 값 사용

recoil state 는 훅을 이용하여 사용해야 한다. 위 예제의 `useRecoilState` 같은 훅이다. 현재까지 세가지 방법의
recoil state 사용법이 있다.

- useRecoilValue: 값만을 불러올 수 있다. 즉 이 훅은 set 함수를 반환하지 않는다.
- useSetRecoilState: set 함수만을 불러올 수 있다.
- useRecoilState: 값, set 함수 두가지 다 불러올 수 있다.

위 예제에서는 useRecoilState 만을 사용하였는데 현재 setCount 는 사용하지 않으므로 `useRecoilValue` 로 리팩토링
하는 것이 좋을 것 같다.

```jsx harmony
function Counter() {
    const count = useRecoilValue(countState);
    const [evenCount, setEvenCount] = useRecoilState(countEvenState);
    const incrementByOneEvenCount = () => setEvenCount(count + 1);

    return (
        <div>
            Count: {count}
            <br />
            Even Count: {evenCount ? '짝수' : '홀수'}
            <br />
            <button onClick={incrementByOneEvenCount}>Even Increment</button>
        </div>
    );
}
```

### useResetRecoilState

recoil state 를 default 값으로 초기화 시킬때 사용한다.

```jsx harmony
const resetCount = useResetRecoilState(countState);
<button onClick={resetCount}>reset</button>
```

### useRecoilValueLoadable

밑에서 다시 언급하겠지만 이 훅은 주로 비동기 selector 를 쓸때 사용된다. React.Suspense 로 loading 처리를 할 수 있지만
위 훅을 사용하면 현재 상태(state) 와 값(contents)을 반환해준다.

```javascript
const countLoadable = useRecoilValueLoadable(countEvenState);
console.log(countLoadable);
```

### useRecoilCallback

recoil state 를 불러오지 않았을때도 callback 함수를 전달해주어 해당 callback 에서 recoil state 에 접근 가능하도록
해준다.

아래는 공식 문서의 예제이다.

```jsx harmony
import {atom, useRecoilCallback} from 'recoil';

const itemsInCart = atom({
  key: 'itemsInCart',
  default: 0,
});

function CartInfoDebug() {
  const logCartItems = useRecoilCallback(async ({getPromise}) => {
    const numItemsInCart = await getPromise(itemsInCart);
    console.log('Items in cart: ', numItemsInCart);
  });

  return (
    <div>
      <button onClick={logCartItems}>Log Cart Items</button>
    </div>
  );
}
```

## 비동기 처리

프로젝트를 진행함에 있어 API 호출과 같은 비동기 처리가 중요하다. recoil 에서도 역시 비동기 처리에 대한
가이드를 전달해준다.

recoil 폴더에 name.js 라는 파일을 추가한 후 아래와 같이 recoil state 를 생성했다.

```javascript
import {atom, selector} from "recoil";

const getName = name => new Promise(resolve => {
    window.setTimeout(() => {
        resolve({name});
    }, 1000);
});

export const currentUserNameState = atom({
    key: 'currentUserNameState',
    default: "Kkangil",
});

export const currentUserName = selector({
    key: 'currentUserName',
    get: async ({get}) => {
        const response = await getName(get(currentUserNameState));
        return response.name;
    },
});

```

`selector` 의 get option 에서 async/await 처리가 가능하다. 이후 UserName 이라는 Component 를 생성해줬다.

```jsx harmony
import React from "react";
import {useRecoilValue} from "recoil";
import {currentUserName} from "../recoil/name";

function UserName() {
    const userName = useRecoilValue(currentUserName);
    return <div>{userName}</div>;
}

export default UserName;

```

이렇게 Component 를 생성해 준 후 확인해보면 React.Suspense 를 사용해야 한다는 에러가 발생한다.
index.js 에 React.Suspense 를 추가해주자.

```jsx harmony
ReactDOM.render(
  <React.StrictMode>
      <RecoilRoot>
          <React.Suspense fallback={<div>Loading...</div>}>
            <App />
          </React.Suspense>
      </RecoilRoot>
  </React.StrictMode>,
  document.getElementById('root')
);
```

## 유틸리티

아쉽지만 현재 버전 0.0.7 에서는 제공하고 있지 않은것 같다. 추후 버전 업이 됐을때 기대해봐도 좋을것 같다.

### atomFamily

atom 을 사용하다가 한가지 의문이 들었다. 초기값을 설정해 주는것은 알겠는데 동적으로 초기값을 설정해 줄 수는
없을까? 역시 존재했다.

```typescript
function atomFamily<T, Parameter>({
  key: string,

  default:
    | RecoilValue<T>
    | Promise<T>
    | T
    | (Parameter => T | RecoilValue<T> | Promise<T>),

  dangerouslyAllowMutability?: boolean,
}): RecoilState<T>
```

공식 문서에서 제공해주고 있는 atomFamily 의 type 정의이다. default 부분을 보면 RecoilValue, Promise 그리고 함수로
정의되어 있는것을 확인할 수 있다.

```jsx harmony
export const countStateByFamily = atomFamily({
    key: "countState",
    default: defaultValue => defaultValue
});

function Count2({number}) {
  const count = useRecoilValue(countStateByFamily(number));
  return (
    <div>
      Count: {count}
    </div>
  );
}
```

### selectorFamily

atom 과 마찬가지로 selector 도 값을 넘겨주어 사용할 수 있다. atom 과 다른점은 selector 의 get 과 set 은
이미 함수를 사용하고 있었다. 값을 넘겨주기 위해 함수가 함수를 리턴해주는 형식이 된다.

```jsx harmony
const myNumberState = atom({
  key: 'MyNumber',
  default: 2,
});

const myMultipliedState = selectorFamily({
  key: 'MyMultipliedNumber',
  get: (multiplier) => ({get}) => {
    return get(myNumberState) * multiplier;
  },

  set: (multiplier) => ({set}, newValue) => {
    set(myNumberState, newValue / multiplier);
  },
});

function MyComponent() {
  const number = useRecoilValue(myNumberState);
  const multipliedNumber = useRecoilValue(myMultipliedState(100));

  return <div>{number} / {multipliedNumber}</div>;
}
```

## 결론

아직 버전이 0.0.7 이고 개발이 더 필요해 보인다. 언제 정식 출시 될지도 모르곘지만 기존에 주로 사용되던
redux, mobx 와 비교를 해보자면 훨씬 더 간단하고 간결하게 사용할 수 있을것 같다. redux 에서의 action, reducer, 
middleware 등 작업에 걸리는 시간을 줄일 수 있을것 같다. 그리고 함수형 컴포넌트와 잘 어울릴것 같다.

참고: [RecoilRoot 공식 문서](https://recoiljs.org/docs/introduction/installation)
Github: [kkangil](https://github.com/kkangil/react-recoiljs)
