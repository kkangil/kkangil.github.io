---
title: '리액트 Hooks: 3. 사용방법2'
date: 2019-04-19 12:54:04
categories:
  - React
tags:
  - React-Hooks
  - Hooks
  - hooks 사용방법
  - useReducer
  - useMemo
  - useCallback
  - useRef
  
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
---

## useReducer
redux와 같이 action을 사용하여 useState보다 다양하게 state 를 조작할수 있게 해준다.

```jsx harmony
import React, { useReducer } from "react";

const reducer = (state, action) => {
  switch (action.type) {
    case "INCREMENT":
      return { ...state, count: state.count + 1 };
    case "DECREMENT":
      return { ...state, count: state.count - 1 };
    case "HANDLE_CHANGE":
      return { ...state, [action.target.name]: action.target.value };
    default:
      return state;
  }
};


const CountReducer = () => {
  const [state, dispatch] = useReducer(reducer, {
    count: 0,
    name: "",
    nickname: ""
  });

  const handleChange = e => {
    dispatch({ type: "HANDLE_CHANGE", target: e.target });
  };

  return (
    <div>
      <p>current count: {state.count}</p>
      <button onClick={() => dispatch({ type: "INCREMENT" })}>increase</button>
      <button onClick={() => dispatch({ type: "DECREMENT" })}>decrease</button>
      <div>
        <input name="name" value={state.name} onChange={handleChange} />
        <input name="name" value={state.nickname} onChange={handleChange} />
      </div>
    </div>
  );
};

export default CountReducer;

```

<!-- more -->
- 이전에 useState로 구현했었던 count와 input 컴포넌트를 위의 코드와 같이 reducer를 사용해서 구현할 수 있다.
- useState 또는 useReducer 어떤것이 더 좋다 안좋다는 없지만, 상황에 따라 유연하게 사용할 줄 알아야 할것같다.


## useMemo
두번째 파라미터 배열 값을 전달받아 해당 값이 변경 될때만 첫번째 파라미터 함수를 실행한다. 해당 기능을 사용하면 불필요한 함수 호출을 줄일 수 있다.

```jsx harmony
import React, { useState, useMemo } from "react";

const Sum = () => {
  const [num, setNum] = useState("");
  const [numList, setNumList] = useState([]);

  const handleClickAdd = () => {
    if (+num) {
      const newNumList = [...numList];
      newNumList.push(+num);
      setNumList(newNumList);
      setNum("");
    }
  };

  const sumValue = useMemo(() => {
    if (!numList.length) return 0;
    const sum = numList.reduce((a, b) => a + b);
    return sum;
  }, [numList]);

  return (
    <div>
      Sum
      <div>
        <input value={num} onChange={e => setNum(e.target.value)} />
        <button onClick={handleClickAdd}>add</button>
      </div>
      <div>number list: {numList.join(", ")}</div>
      <div>number total: {sumValue}</div>
    </div>
  );
};

export default Sum;
```

- sumValue 함수에 `useMemo`를 사용하지 않으면, input 값이 변경될때도 해당 로직이 불필요하게 실행된다.
- useMemo를 사용해 줌으로써 불필요하게 실행되지 않도록 하고, `numList`의 값이 변경될때마다 해당 로직이 실행된다.

## useCallback
useMemo와 거의 비슷한 기능으로 렌더링 성능을 최적화 하기위해 주로 사용된다. 쓰지 않아도 문제되지는 않지만 렌더링 해야할 컴포넌트 개수나 로직이 많아진다면 고려해 봐야한다. 해당 기능은 이벤트 함수를 필요할때만 생성해 주는 기능이다.

```jsx harmony
const handleChange = useCallback(e => {
    setNum(e.target.value);
  }, []);

  const handleClickAdd = useCallback(() => {
    if (+num) {
      const newNumList = [...numList];
      newNumList.push(+num);
      setNumList(newNumList);
      setNum("");
    }
  }, [numList, num]);
```

- 위의 onChange 함수와 handleClickAdd 함수를 useCallback을 사용하는 함수로 변경했다.
- 두번째 파라미터 배열은 해당 값이 변경될때만 함수를 생성해 주는것이다. 빈 배열이라면 최초에만 생성해주고 이후 다시 생성되지 않는다.
- handleChange 는 빈배열, handleClickAdd 은 두개의 값을 판단하고 있는데, 이는 handleChange는 단순히 `num` 의 값만 변경해 주고 있어 초기에만 생성해 줘도 되는것이다. handleClickAdd 함수는 이벤트가 실행 될때마다 값을 가져와서 사용해 주고 있기때문에 배열에 해당 값들을 명시해줘야한다.

## useRef
useRef는 함수형 컴포넌트에서도 ref 기능을 사용할 수 있게 해준다.

```jsx harmony
const inputElement = useRef(null);
const handleClickAdd = useCallback(() => {
    if (+num) {
      const newNumList = [...numList];
      newNumList.push(+num);
      setNumList(newNumList);
      setNum("");
      inputElement.current.focus();
    }
  }, [numList, num]);
<input value={num} onChange={handleChange} ref={inputElement} />
```

- useRef는 다른 Hook과 다르게 최초 선언을 배열 형태로 선언해 주지 않는다.
- ref 값이 변경되어도 리렌더링 되지 않는다.
