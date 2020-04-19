---
title: '리액트 Hooks: 2. 사용방법1'
date: 2019-04-19 12:52:51
categories:
  - React
tags:
  - React-Hooks
  - Hooks
  - hooks 사용방법
  - useState
  - useEffect
  - useContext
---

## useState

### 사용방법
useState 는 이름에서 유추하는것과 같이 함수형 컴포넌트에서도 state를 사용할 수 있게 해준다.

사용방법은 해당 기능을 import 한 후 [state 이름, state이름을 변경시켜줄 setState와 같은 이름] = useState(초기값) 이다.

<!-- more -->

```
import React, { useState } from "react";

const Counter = props => {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>current count: {count}</p>
      <button onClick={() => setCount(count + 1)}>increase</button>
      <button onClick={() => setCount(count - 1)}>decrease</button>
    </div>
  );
};

export default Counter;
```
- useState를 사용하여 count state를 생성해 주었으며 초기값은 0으로 맞춰 주었다.
- state 사용은 기존의 class에서 사용하던 this.state.count가 아닌 count로만 사용해야 한다.
- setCount 파라미터로 값을 넘겨주면 해당 값으로 state가 변경된다.

### 여러개의 state 사용
여러개의 state를 사용해야 한다면, 기존 class 방식에서는 state 객체 안에 키와 값을 설정해 주면 됐지만, 함수형 컴포넌트에서는 useState를 여러번 써줘야 한다.

```
const Counter = props => {
  const [count, setCount] = useState(0);
  const [name, setName] = useState("");
  return (
    <div>
      <input value={name} onChange={e => setName(e.target.value)} />
      <p>
        {name ? `${name}'s` : ""} current count: {count}
      </p>
      <button onClick={() => setCount(count + 1)}>increase</button>
      <button onClick={() => setCount(count - 1)}>decrease</button>
    </div>
  );
};
```

### 재사용 가능한 useState 함수 컴포넌트
동일한 stateful 로직을 사용한다면 컴포넌트를 분리해서 사용하는것이 좋다.

```
const useInputOption = props => {
  const [value, setValue] = useState(props || "");
  const onChange = e => {
    setValue(e.target.value);
  };
  return {
    value,
    onChange
  };
};

const Counter = props => {
  const [count, setCount] = useState(0);
  const nameInputOption = useInputOption("");
  const nickInputOption = useInputOption("");

  return (
    <div>
      <input {...nameInputOption} />
      <input {...nickInputOption} />
      <p>
        {nameInputOption.value ? `${nameInputOption.value}'s` : ""} current
        count: {count}
      </p>
      <button onClick={() => setCount(count + 1)}>increase</button>
      <button onClick={() => setCount(count - 1)}>decrease</button>
    </div>
  );
};
```

- input의 value와 onChange 이벤트 함수를 return 해주는 `useInputOption` 컴포넌트를 만들었다.
- 코드가 간결해지고 재사용성이 높아진다.

## useEffect
useEffect는 함수형 컴포넌트에서도 라이프사이클과 같은 기능을 사용할 수 있게 해준다.
`componentDidMount` 와 `componentDidUpdate`, `componentWillUnmount` 를 합쳐놓은 것이다.
useEffect 함수는 state 변수를 하나만 관리하는것이 좋다. 즉 여러개의 useEffect 함수를 사용하는것을 추천해주고 있다.

### 사용방법
```
import React, { useState, useEffect } from "react";

const User = () => {
  const [name, setName] = useState("");

  useEffect(() => {
    console.log("completed render", name);
  });

  return <input value={name} onChange={e => setName(e.target.value)} />;
};

export default User;
```
- 해당 코드를 확인해보면 render 가 될때마다 console.log 가 찍히는것을 확인할 수 있다. 이것은 `componentDidMount` 와 `componentDidUpdate` 라이프사이클과 일치한다.


### componentDidMount
```
useEffect(() => {
    console.log("completed render", name);
  }, []);
```
- useEffect 함수에 두번째 파라미터로 빈배열을 넣으면 최초 렌더링이 될때만 실행되고 name이 변경되더라도 실행되지 않는다.


### componentDidUpdate
```
useEffect(() => {
    console.log("completed render", name);
  }, [name]);
```

- 배열에 원하는 값을 넣어주면 해당 값이 변할때만 실행된다.

### componentWillUnmount
```
useEffect(() => {
    console.log("completed render", name);
    return () => {
      console.log("unmount", name);
    };
  }, [name]);
```
- return 함수는 렌더링이 될때마다 실행되는 함수이다. name을 확인해보면 변경되기 이전의 값이 찍히는것을 확인할 수 있다.
- 두번째 파라미터에 빈 배열을 담아서 실행하면 unmount 될때만 return 함수가 실행된다.

## Promise 처리
useEffect와 useState를 사용하여 promise 컴포넌트 또한 재사용이 가능하도록 할 수 있다.
```
import { useState, useEffect } from "react";

const usePromise = ({ promise, initialData, arr }) => {
  const [data, setData] = useState(initialData);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(false);

  const fetchData = async () => {
    setLoading(true);
    try {
      const result = await promise();
      setData(result);
    } catch (err) {
      setError(true);
    }
    setLoading(false);
  };

  useEffect(() => {
    fetchData();
  }, arr || []);

  return { data, loading, error };
};

const getNames = async () => {
  return new Promise(resolve =>
    setTimeout(() => resolve([{ name: "kkangil" }, { name: "kkangil2" }]), 1000)
  );
};

const Names = () => {
  const { data: names, loading, error } = usePromise({
    promise: getNames,
    initialData: []
  });

  if (loading) return <div>로딩중...</div>;
  if (error) return <div>에러</div>;

  return (
    <>
      {names.map((row, index) => (
        <div key={index}>{row.name}</div>
      ))}
    </>
  );
};
```
### usePromise
- 파라미터로 호출해야하는 `promise` 함수, 최초 초기화 데이터 `initialData`, useEffect 함수 2번째 파라미터 배열 `arr`을 객체로 받는다.
- `useEffect` 함수에서 async/await을 사용하면 warning이 발생한다. 그렇기 때문에 async/await 함수를 따로 만들어 해당 함수를 useEffect에서 실행 시켜준다.

### Names
- initialData에 빈배열을 넣어주지 않으면 return의 `names.map` 에서 에러가 발생한다. 해당 에러를 막기 위해 빈 배열로 초기화해줬다. `names` 값이 존재할때만 map 을 사용하는 방법도 있지만, 추후 재사용을 위해 데이터 초기값을 설정해줄 수 있도록 했다.

## useContext
useContext는 함수형 컴포넌트에서 Context를 쉽게 사용할 수 있게 해준다.
```
import React, { createContext, useContext } from "react";

const backgroundColorContext = createContext("black");

const Context = () => {
  const backgroundColor = useContext(backgroundColorContext);
  const style = {
    width: "50px",
    height: "50px",
    borderRadius: "50%",
    background: backgroundColor
  };
  return <div style={style} />;
};

export default Context;
```