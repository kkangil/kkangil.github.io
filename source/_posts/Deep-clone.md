---
title: Deep clone
date: 2019-04-18 00:01:31
categories:
  - Javascript
tags:
  - Javascript
  - syntax
  - deep-clone
  
  
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

## 이슈
1. react에서 setState를 하지 않았음에도 state가 변경되는 현상
2. api 구축중 객체를 변경하지 않았음에도 객체 값이 변경되는 현상

react 프로젝트 개발중 위의 이슈가 자주 발생하고 있었다.
해당 이슈가 왜 발생하는지, 어떻게 해결할 수 있는지 기록해두려고 한다.
해당 글은 react 뿐만 아니라 javascript 언어를 사용한다면 꼭 알아둬야 된다고 생각한다.

<!-- more -->

```javascript
state = { a: 1, b: 2 }
handleChange = e => {
  const state = {...this.state};
  state[e.target.name] = e.target.value;
  this.setState(state);
}
```
위 코드는 본인이 input 의 onChange 이벤트 함수로 거의 복붙하면서 변형없이 사용하는 이벤트 함수이다.
최근 state 변수 선언 즉, state 객체를 복사하고 사용하는 과정에 있어 동일한 이슈가 많이 발생하고 있어, 해당 함수를 단순히 복사하는것이 아닌 이해가 필요하고 상황에 따라 변형해서 사용해야할 필요있다.

## Object copy
```javascript
const state = {...this.state}
```
state 변수는 react state 객체를 복사한것이다. 해당 문법은 es9 문법으로 특정 값만 바꿀때 유용하다.
해당 문법의 es5 버전은 .assign() 메소드이다.

```javascript
const object1 = {
  a: 1,
  b: 2,
};

const object2 = Object.assign({}, object1, {a: 100});
const object3 = Object.assign({ c: 3 }, object1, {a: 100});
const object4 = Object.assign({}, object1);
object4.a = 100

const object5 = {...object1}
object5.a = 100

console.log(object2.a); //100
console.log(object2.b); //2
console.log(object3.a); //100
console.log(object3.b); //2
console.log(object3.c); //3
console.log(object4.a); //100
console.log(object5.a); //100
```

- object2 : 빈 객체에 object1을 복사한 후 a 의 값을 변경
- object3: c: 3 값이 담겨있는 객체에 object1을 복사한 후 a 의 값을 변경
- object4: 현재 우리 회사에서 사용하는 handleChange를 es5로 바꾼 방법
- object5: Spread syntax(…) es8 문법사용
하지만 state 내부에 객체를 변경하려고 할때 문제가 발생한다.

```javascript
state = {
  a: 1,
  b: {
    c: 2
  },
  d: {
    c: 2
  }
}

handleChange = e => {
  const state {...this.state}
  state.b.c = 3
  this.setState(state)
}
```

2단계 이상 깊이의 object는 복사가 원활하지 않는것을 확인할 수 있다. 복사본의 값을 변경했는데 원본의 값도 변경되는 알수 없는 현상이 발생한다.
MDN: 깊은 클로닝에 대해서, Object.assign() 은 속성의 값을 복사하기때문에 다른 대안을 사용해야합니다. 출처 값이 객체에 대한 참조인 경우, 참조 값만을 복사합니다.

```javascript
const org = { a : {b : 2}};

const obj = Object.assign({}, org);
obj.a.b = 100; 
console.log(obj.a.b);  //expected: 2 but actual: 100

const obj2 = {...org};
obj2.a.b = 100; 
console.log(org.a.b);  //expected: 2 but actual: 100
```

해당 현상은 깊은 클로닝(deep cloning) 방식을 사용해야한다.
객체 복사 방식 변경 : 객체를 string 으로 변경하고 다시 파싱해주는 방법

```javascript
const org = { a : {b : 2}};
const obj = JSON.parse(JSON.stringify(org)); obj.a.b = 100;
console.log(org.a.b); // 2
lodash 메소드 사용
import _ from 'lodash'; const org = { a : {b : 2}};
const obj = _.cloneDeep(org); obj.a.b = 100;
console.log(org.a.b); // 2
```

## Array copy

2단계 이상 깊이의 배열을 복사할때는 맵을 사용해서 객체 하나하나 복사해준 후 return 해주는 방식을 사용해서 에러발생을 막아야합니다.
```javascript
const org = {
  obj: {
    a: [{
      b: 1
      },{
      b: 2
    }]
  }
}


const obj1 = {...org}
const arr = org.obj.a // x

const arr2 = org.obj.a.map(row => {
	return {...row}
})
```

참고
https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/assign

