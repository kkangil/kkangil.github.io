---
title: jest 메서드 - expect
date: 2020-02-16 14:59:50
categories:
  - Jest
tags: 
  - jest 메서드
  - expect
  - test
  
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

### expect(value)

expect 함수는 값을 테스트 하고 싶을 때 사용한다. 드물게 expect 함수만 사용해서 테스트를 할 수 있지만 값을 테스트하기 위해 `matcher` 함수를 함께 사용한다.

### expect.extend(matchers)

expect.extend 함수를 사용하여 Jest 에서 제공하는 matcher 가 아닌 직접 만들어서 사용할 수 있다.

<!-- more -->

```javascript
expect.extend({
  toBeWithinRange(received, floor, ceiling) {
    const pass = received >= floor && received <= ceiling;
    if (pass) {
      return {
        message: () =>
          `expected ${received} not to be within range ${floor} - ${ceiling}`,
        pass: true,
      };
    } else {
      return {
        message: () =>
          `expected ${received} to be within range ${floor} - ${ceiling}`,
        pass: false,
      };
    }
  },
});

test('numeric ranges', () => {
  expect(100).toBeWithinRange(90, 110);
  expect(101).not.toBeWithinRange(0, 100);
  expect({ apples: 6, bananas: 3 }).toEqual({
    apples: expect.toBeWithinRange(1, 10),
    bananas: expect.not.toBeWithinRange(11, 20),
  });
});
```

`toBeWithinRange` 라는 matcher 를 만드는 방법이다. expect 의 값이 첫번째 인자로 넘어간다. 성공 했을경우 성공 사유 message 와 pass를 true로 return 해준다. 살패 했을경우는 반대로 실패 사유 message 와 pass 를 false 로 return 해준다.

expect.extend 는 비동기 함수 호출도 지원한다.

```javascript
expect.extend({
  async toBeDivisibleByExternalValue(received) {
    const externalValue = await getExternalValueFromRemoteSource();
    const pass = received % externalValue == 0;
    if (pass) {
      return {
        message: () =>
          `expected ${received} not to be divisible by ${externalValue}`,
        pass: true,
      };
    } else {
      return {
        message: () =>
          `expected ${received} to be divisible by ${externalValue}`,
        pass: false,
      };
    }
  },
});

test('is divisible by external value', async () => {
  await expect(100).toBeDivisibleByExternalValue();
  await expect(101).not.toBeDivisibleByExternalValue();
});
```

#### Custom Matchers API

Macher 들은 항상 두개의 key 를 포함하고 있는 객체를 리턴해야한다. pass 는 성공/실패 여부이고 message 는 테스트가 실패 했을 경우 보여진다. .not() 메서드를 사용하면 실패 했을때 pass: true 의 message 가 보여진다.

### expect.anything

null 과 undefined 을 제외한 모든 값들과 일치한다. 즉, null 과 undefined 외의 모든 값들은 동일하다.

```javascript
test('map calls its argument with a non-null argument', () => {
  const mock = jest.fn();
  [1].map(x => mock(x));
  expect(mock).toBeCalledWith(1);
});
```

위의 테스트 코드는 성공할 것이다. 하지만 `expect(mock).toBeCalledWith(2);` 로 바꾸게 된다면 배열의 length 가 1 이므로 실패할 것이다. 만약 mock() 함수가 몇번 실행이 되는지 상관없이 테스트 케이스를 성공 처리 하고 싶을때 anything 을 사용하는 것이다.

```javascript
expect(mock).toBeCalledWith(expect.anything());
```

### expect.any(constructor)

expect.any(constructor) matches anything that was created with the given constructor

expect.any 는 주어진 생성자와 값이 해당 생성자에 일치하는 지를 테스트 한다. `toEqual` 과 `toBeCalledWith` 함수에 값대신 해당 메서드를 사용할 수 있다.

randocall 함수의 return 값이 Number 인지 테스트하는 코드다.

```javascript
function randocall(fn) {
  return fn(Math.floor(Math.random() * 6 + 1));
}

test('randocall calls its callback with a number', () => {
  const mock = jest.fn();
  randocall(mock);
  expect(mock).toBeCalledWith(expect.any(Number));
});
```

### expect.arrayContaining(array)

기대값이 expect.arrayContaining(array) 메서드에 주어지는 배열의 요소를 모두 포함하고 있는지 확인할 때 사용하는 메서드이다. `toEqual` 과 `toBeCalledWith` 함수에 사용할 수 있다.

```javascript
describe('arrayContaining', () => {
  const expected = ['Alice', 'Bob'];
  it('matches even if received contains additional elements', () => {
    expect(['Alice', 'Bob', 'Eve']).toEqual(expect.arrayContaining(expected));
  });
  it('does not match if received does not contain expected elements', () => {
    expect(['Bob', 'Eve']).not.toEqual(expect.arrayContaining(expected));
  });
  expect(['Alice', 'Alice', 'Bob', 'Eve']).toEqual(expect.arrayContaining(expected));
});
```

첫번째 테스트의 경우 기대값 요소에 'Alice', 'Bob' 이 포함되어 있기 때문에 성공한다. 두번째의 경우는 'Alice' 를 포함하고 있지 않기 때문에 실패한다. 세번째 테스트 경우처럼 배열 요소에 중복이 있어도 상관없이 테스트는 성공한다.

expect.not.arrayContaining(array) 메서드도 사용 가능하다.


```javascript
describe('not.arrayContaining', () => {
  const expected = ['Samantha'];
  it('matches if the actual array does not contain the expected elements', () => {
    expect(['Alice', 'Bob', 'Eve']).toEqual(
      expect.not.arrayContaining(expected),
    );
  });
});
```

### expect.objectContaining(object)

객체의 key 와 value 를 포함하고 있는지 테스트할때 사용한다.

```javascript
describe('objectContaining', () => {
  const expected = { foo: 'bar' };

  it('matches if the actual object does contain expected key: value pairs', () => {
    expect({ bar: 'baz' }).toEqual(expect.objectContaining(expected));
  });
});
```

`foo` 라는 key 가 없어서 위 테스트는 실패한다. 만약 { bar: 'baz' } 를 { foo: 'baz' } 객체로 변경해서 테스트 해보면 실패하는것을 확인할 수 있다. objectContaining 는 key/value 쌍으로 동일해야 성공한다. 

```javascript
test('onPress gets called with the right thing', () => {
  const onPress = jest.fn();
  simulatePresses(onPress);
  expect(onPress).toBeCalledWith(
    expect.objectContaining({
      x: expect.any(Number),
      y: expect.any(Number),
    }),
  );
});
```

objectContaining 객체의 value 에 특정한 값이 아니라 any, anything 도 사용할 수 있다. expect.not.objectContaining(object) 메서드도 사용 가능하다.

### expect.stringContaining(string)

expect.stringContaining(string) 은 주어지는 문자를 포함하고 있는지 테스트한다.

```javascript
describe('stringContaining', () => {
  const expected = 'Hello world!';

  it('matches if the received value does contain the expected substring', () => {
    expect('Hello world!').toEqual(expect.stringContaining(expected));
  });
});
```

expect.not.stringContaining(string) 도 사용 가능하다.

### expect.stringMatching(string | regexp)

주어진 string 이나 정규식에 일치하는지 테스트할 때 사용한다. string 이 주어지는 경우 완전하게 일치 해야하며 정규식을 사용하는 경우는 해당 정규식에 일치하면 된다.

```javascript
describe('stringMatching in arrayContaining', () => {
  const expected = [
    expect.stringMatching(/^Alic/),
    expect.stringMatching(/^[BR]ob/),
  ];
  it('matches even if received contains additional elements', () => {
    expect(['Alicia', 'Roberto', 'Evelina']).toEqual(
      expect.arrayContaining(expected),
    );
  });
  it('does not match if received does not contain expected elements', () => {
    expect(['Roberto', 'Evelina']).not.toEqual(
      expect.arrayContaining(expected),
    );
  });
});
```


### expect.assertions(number)

test 함수의 callback 함수 내부에서 테스트가 몇번이 일어나는지 확인하는 메서드이다.

```javascript
test('assertions count', () => {
  expect.assertions(2);
  expect(true).toBeTruthy();
  expect(null).toBeFalsy();
});
```

위 테스트코드에서 expect.assertions 을 제외하고 두개의 테스트를 하고있다.

### expect.hasAssertions()

expect.hasAssertions() 함수는 test 함수의 callback 함수 내부에서 테스트가 최소 한번 실행되고 있는지 테스트하는 함수다.

```javascript
test('has assertions', () => {
  expect.hasAssertions();
  expect(null).toBeFalsy();
});
```

### resolves

성공된 Promise 의 value 를 가져올때 사용한다. resolves 를 사용하지 않으면 received value 는 {} 가 된다.

```javascript
test('resolves to lemon', () => {
  expect(Promise.resolve('lemon')).resolves.toBe('lemon');
});
```

async/await 과도 사용할 수 있다.

```javascript
test('async/await resolves to lemon', async () => {
  await expect(Promise.resolve('lemon')).resolves.toBe('lemon');
  await expect(Promise.resolve('lemon')).resolves.not.toBe('octopus');
});
```

### rejects

rejects 는 resolves 와는 반대로 실패된 Promise 의 reason 을 가져올 때 사용한다.

```javascript
test('rejects to octopus', () => {
  // make sure to add a return statement
  return expect(Promise.reject(new Error('octopus'))).rejects.toThrow(
    'octopus',
  );
});

test('rejects to octopus', async () => {
  await expect(Promise.reject(new Error('octopus'))).rejects.toThrow('octopus');
});
```

참조: [Jest 공식 문서](https://jestjs.io/docs/en/expect)
