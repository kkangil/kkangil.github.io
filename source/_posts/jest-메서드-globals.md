---
title: jest 메서드 - globals
date: 2020-02-09 17:01:54
categories:
  - Jest
tags: 
  - jest globals 메서드
  - describe
  - test
---

Jest 제공 메서드들 중 Globals 메서드에 대해서 정리해보려 한다. Jest 공식 문서에 나와 있는 Globals 메서드 정의는 다음과 같다.

> Globals 메서드를 사용해서 전역 환경을 설정한다. 이것은 필수로 설정해 줄 필요는 없다.

<!-- more -->

### afterAll(fn, timeout)

이름에서 유추할 수 있듯이 테스트가 진행되는 동안 매번 실행되는 것이 아니라 모든 테스트가 완료 되었을 때 실행된다. 만약 인자의 함수가 Promise 이거나 generator 일 경우, Jest 는 완료를 기다린다.

다른 테스트와 공유되는 전역 환경 상태를 초기화 할때 주로 사용한다.

```
const globalDatabase = makeGlobalDatabase();

function cleanUpDatabase(db) {
  db.cleanUp();
}

afterAll(() => {
  cleanUpDatabase(globalDatabase);
});
```

descirbe 함수 내부에서 사용될 경우 describe 내부의 테스트가 모두 완료될 때 실행된다.

### afterEach(fn, timeout)

afterAll 메서드와 달리 afterEach는 하나의 테스트가 완료될 때마다 실행된다. 각 테스트에 의해 생성된 임시 상태 또는 변수를 초기화 하는 경우 주로 사용한다.

```
const globalDatabase = makeGlobalDatabase();

function cleanUpDatabase(db) {
  db.cleanUp();
}

afterEach(() => {
  cleanUpDatabase(globalDatabase);
});
```

descirbe 함수 내부에서 사용될 경우 describe 내부의 각각 테스트가 완료될 때 실행된다.

### beforeAll(fn, timeout)

afterAll과 정반대라고 생각하면 된다. 이 메서드는 테스트가 실행되기전 최초에 한번 실행된다. 각 테스트를 진행하기 위해 데이터를 설정해 줄때 주로 사용된다.

```
const globalDatabase = makeGlobalDatabase();

beforeAll(() => {
  // Clears the database and adds some testing data.
  // Jest will wait for this promise to resolve before running tests.
  return globalDatabase.clear().then(() => {
    return globalDatabase.insert({testData: 'foo'});
  });
});
```

descirbe 함수 내부에서 사용될 경우 describe 내부의 테스트를 하기전에 실행된다.

### beforeEach(fn, timeout)

하나의 테스트가 시작되기 전에 실행된다. descirbe 함수 내부에서 사용될 경우 describe 내부의 각각 테스트가 시작되기 전 실행된다.

```
const globalDatabase = makeGlobalDatabase();

beforeEach(() => {
  // Clears the database and adds some testing data.
  // Jest will wait for this promise to resolve before running tests.
  return globalDatabase.clear().then(() => {
    return globalDatabase.insert({testData: 'foo'});
  });
});
```

중복된 코드를 제거할때 유용하게 사용할 수 있다.

### describe(name, fn)

describe 메서드는 몇몇의 관계가 있는 테스트들을 그룹으로 묶어 생성할 때 사용한다. test 메서드를 최상위에서 바로 실행시킬 수 있지만, describe 메서드로 관련있는 테스트 끼리 묶어서 작성하게 되면 가독성이 높아진다.

```
const myBeverage = {
  delicious: true,
  sour: false,
};

describe('my beverage', () => {
  test('is delicious', () => {
    expect(myBeverage.delicious).toBeTruthy();
  });

  test('is not sour', () => {
    expect(myBeverage.sour).toBeFalsy();
  });
});
```

describe 메서드 함수 내부에서 다시 describe로 그룹화를 할 수 있다.

### describe.each(table)(name, fn, timeout)

여러가지 다른 데이터로 중복되는 테스트를 수행할때 describe.each를 활용할 수 있다. 하나의 테스트 케이스로  값이 다른 데이터들로 테스를 수행하는 것이다.

- table: 인자로 배열을 넘기면 fn 함수의 인자로 사용 가능하다.
- name: 테스트의 이름 printf formatting 기법을 사용할 수 있다.

```
describe.each([
  [1, 1, 2],
  [1, 2, 3],
  [2, 1, 3],
])('.add(%i, %i)', (a, b, expected) => {
  test(`returns ${expected}`, () => {
    expect(a + b).toBe(expected);
  });

  test(`returned value not be greater than ${expected}`, () => {
    expect(a + b).not.toBeGreaterThan(expected);
  });

  test(`returned value not be less than ${expected}`, () => {
    expect(a + b).not.toBeLessThan(expected);
  });
});
```

describe.each`table` 형식으로도 사용 가능하다.

```
describe.each`
  a    | b    | expected
  ${1} | ${1} | ${2}
  ${1} | ${2} | ${3}
  ${2} | ${1} | ${3}
`
```

### describe.only(name, fn)

오직 하나의 describe 그룹의 테스트만 수행하고 싶을 때 사용한다. 다른 describe 테스트들은 skip 된다.
`describe.only.each` 메서드도 사용할 수 있다.

```
describe.only('my beverage', () => {
  test('is delicious', () => {
    expect(myBeverage.delicious).toBeTruthy();
  });

  test('is not sour', () => {
    expect(myBeverage.sour).toBeFalsy();
  });
});

describe('my other beverage', () => {
  // ... will be skipped
});
```

### describe.skip(name, fn)

특정한 describe 그룹을 테스트 하고 싶지 않을때 사용한다.
```
describe('my beverage', () => {
  test('is delicious', () => {
    expect(myBeverage.delicious).toBeTruthy();
  });

  test('is not sour', () => {
    expect(myBeverage.sour).toBeFalsy();
  });
});

describe.skip('my other beverage', () => {
  // ... will be skipped
});
```

`describe.skip.each(table)(name, fn)` 메서드도 사용가능하다.

### test(name, fn, timeout)

test 메서드를 사용하여 테스트를 수행할 수 있다. Promise나 비동기 방법을 지원한다.

```
test('did not rain', () => {
  expect(inchesOfRain()).toBe(0);
});

test('has lemon in it', () => {
  return fetchBeverageList().then(list => {
    expect(list).toContain('lemon');
  });
});
```

### test.each(table)(name, fn, timeout)

describe.each(table)(name, fn, timeout)와 개념은 동일하다. 하지만 describe는 그룹화이기 때문에 여러가지 테스트를 동시에 수행할 수 있지만 test.each는 단일 테스트에 대한 each 이다.

### test.only(name, fn, timeout)

describe.only 와 개념은 동일하다.
```
test.only('it is raining', () => {
  expect(inchesOfRain()).toBeGreaterThan(0);
});

test('it is not snowing', () => {
  expect(inchesOfSnow()).toBe(0);
});
```

오직 한가지의 test만 수행되며 describe 내부의 test들도 수행되지 않는다.

`test.only.each(table)(name, fn)` 메서드도 사용할 수 있다.

### test.skip(name, fn)

수행하고 싶지 않은 테스트를 건너뛰게 할 수 있다.

```
test('it is raining', () => {
  expect(inchesOfRain()).toBeGreaterThan(0);
});

test.skip('it is not snowing', () => {
  expect(inchesOfSnow()).toBe(0);
});
```

`test.skip.each(table)(name, fn)` 도 사용가능하다.

### test.todo(name)

추가되어야할 테스트 TODO를 남겨놓을 수 있다. 해당 메서드를 사용하면 다른 테스트들과 달리 강조되어 표시된다. 인자로 callback을 넘기면 에러가 발생한다.

```
const add = (a, b) => a + b;

test.todo('add should be associative');
```

참조: [Jest 공식 문서](https://jestjs.io/docs/en/api)