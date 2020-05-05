---
title: jest 메서드-toBe
date: 2020-02-23 15:46:45
categories:
  - Jest
tags: 
  - toBe
  - toEqual
  - toHaveBeenCalled
  - toHaveReturned
  - toError
  
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

## .toBe(value)

.toBe(value) 메서드는 expect 의 값과 비교할때 사용한다. 이것은 Object.is() 를 사용하여 비교하는데 === 연산자를 사용하는 것보다 테스트하기에 더 좋다.

```javascript
const can = {
  name: 'pamplemousse',
  ounces: 12,
};

describe('the can', () => {
  test('has 12 ounces', () => {
    expect(can.ounces).toBe(12);
  });

  test('has a sophisticated name', () => {
    expect(can.name).toBe('pamplemousse');
  });
});
```

<!-- more -->

.toBe(value) 메서드에는 소수를 쓰지 않는것이 좋다. 예를 들면 

```javascript
expect(0.1 + 0.2).toBe(0.3)
```

위와 같은 경우이다. 자바스크립트에서는 0.1 + 0.2 가 0.3이 아니기 때문이다.

## .toHaveBeenCalled()

mock 함수가 실행됐는지를 테스트하는 메서드이다.
mock 함수가 실행됐으면 테스트에 성공하는데 반대로 해당 함수를 실행하지 않는 것이 원하는 결과라면 .not 을 사용해서 테스트 코드를 구성하면 된다.

```javascript
function drinkAll(callback, flavour) {
  if (flavour !== 'octopus') {
    callback(flavour);
  }
}

describe('drinkAll', () => {
  test('drinks something lemon-flavoured', () => {
    const drink = jest.fn();
    drinkAll(drink, 'lemon');
    expect(drink).toHaveBeenCalled();
  });

  test('does not drink something octopus-flavoured', () => {
    const drink = jest.fn();
    drinkAll(drink, 'octopus');
    expect(drink).not.toHaveBeenCalled();
  });
});
```

## .toHaveBeenCalledTimes(number)

mock 함수가 몇번 실행됐는지를 확인할때 사용하는 메서드이다. 인자의 number 만큼 실행됐으면 테스트는 성공한다.

```javascript
test('drinkEach drinks each drink', () => {
  const drink = jest.fn();
  drinkEach(drink, ['lemon', 'octopus']);
  expect(drink).toHaveBeenCalledTimes(2);
});
```

## toHaveBeenCalledWith(arg1, arg2, ...)

mock 함수가 실행됐을때 인자값을 테스트할때 사용한다.

```javascript
test('registration applies correctly to orange La Croix', () => {
  const beverage = new LaCroix('orange');
  register(beverage);
  const f = jest.fn();
  applyToAll(f);
  expect(f).toHaveBeenCalledWith(beverage);
});
```

## toHaveBeenLastCalledWith(arg1, arg2, ...)

mock 함수가 여러번 실행되었을때 마지막 실행의 인자값을 비교할때 사용한다.

```javascript
test('applying to all flavors does mango last', () => {
  const drink = jest.fn();
  applyToAllFlavors(drink);
  expect(drink).toHaveBeenLastCalledWith('mango');
});
```

## .toHaveBeenNthCalledWith(nthCall, arg1, arg2, ....)

mock 함수가 여러번 실행되었을때 순서를 지정하여 인자값을 비교한다.

```javascript
test('drinkEach drinks each drink', () => {
  const drink = jest.fn();
  drinkEach(drink, ['lemon', 'octopus']);
  expect(drink).toHaveBeenNthCalledWith(1, 'lemon');
  expect(drink).toHaveBeenNthCalledWith(2, 'octopus');
});
```

## .toHaveReturned()

mock 함수가 실행되었고 return 이 되었는지를 테스트하는 메서드이다.

```javascript
test('drinks returns', () => {
  const drink = jest.fn(() => true);

  drink();

  expect(drink).toHaveReturned();
});
```

## .toHaveReturnedTimes(number)

mock 함수가 실행되었고 몇번 return 이 되었는지를 테스트하는 메서드이다.

```javascript
test('drink returns twice', () => {
  const drink = jest.fn(() => true);

  drink();
  drink();

  expect(drink).toHaveReturnedTimes(2);
});
```

## .toHaveReturnedWith(value)

mock 함수가 실행되었고 어떤 값이 리턴되었는지 테스트하는 메서드이다.

```javascript
test('drink returns La Croix', () => {
  const beverage = {name: 'La Croix'};
  const drink = jest.fn(beverage => beverage.name);

  drink(beverage);

  expect(drink).toHaveReturnedWith('La Croix');
});
```

## .toHaveLastReturnedWith(value)

마지막으로 실행된 mock 함수의 return 값을 테스트하는 메서드이다.

```javascript
test('drink returns La Croix (Orange) last', () => {
  const beverage1 = {name: 'La Croix (Lemon)'};
  const beverage2 = {name: 'La Croix (Orange)'};
  const drink = jest.fn(beverage => beverage.name);

  drink(beverage1);
  drink(beverage2);

  expect(drink).toHaveLastReturnedWith('La Croix (Orange)');
});
```

## .toHaveNthReturnedWith(nthCall, value)

mock 함수가 여러번 실행되었을때 순서를 지정하여 return 값을 비교한다.

```javascript
test('drink returns expected nth calls', () => {
  const beverage1 = {name: 'La Croix (Lemon)'};
  const beverage2 = {name: 'La Croix (Orange)'};
  const drink = jest.fn(beverage => beverage.name);

  drink(beverage1);
  drink(beverage2);

  expect(drink).toHaveNthReturnedWith(1, 'La Croix (Lemon)');
  expect(drink).toHaveNthReturnedWith(2, 'La Croix (Orange)');
});
```

## .toHaveLength(number)

length 속성을 가진 객체에서 길이를 비교할때 사용한다. array 와 string 의 길이를 비교할 때 유용하다.

```javascript
expect([1, 2, 3]).toHaveLength(3);
expect('abc').toHaveLength(3);
expect('').not.toHaveLength(5);
```

## .toHaveProperty(keyPath, value?)

객체에 keyPath 의 속성이 있는지 테스트할때 사용한다. 두번째 인자 value 는 인자값으로 keyPath 만 사용했을 경우에는 해당 속성이 있는지만 확인하지만 value 까지 사용하면 keyPath 속성의 값도 같이 비교한다.

```javascript
const houseForSale = {
  bath: true,
  bedrooms: 4,
  kitchen: {
    amenities: ['oven', 'stove', 'washer'],
    area: 20,
    wallColor: 'white',
    'nice.oven': true,
  },
  'ceiling.height': 2,
};

test('this house has my desired features', () => {
  // Example Referencing
  expect(houseForSale).toHaveProperty('bath');
  expect(houseForSale).toHaveProperty('bedrooms', 4);

  expect(houseForSale).not.toHaveProperty('pool');

  // Deep referencing using dot notation
  expect(houseForSale).toHaveProperty('kitchen.area', 20);
  expect(houseForSale).toHaveProperty('kitchen.amenities', [
    'oven',
    'stove',
    'washer',
  ]);

  expect(houseForSale).not.toHaveProperty('kitchen.open');

  // Deep referencing using an array containing the keyPath
  expect(houseForSale).toHaveProperty(['kitchen', 'area'], 20);
  expect(houseForSale).toHaveProperty(
    ['kitchen', 'amenities'],
    ['oven', 'stove', 'washer'],
  );
  expect(houseForSale).toHaveProperty(['kitchen', 'amenities', 0], 'oven');
  expect(houseForSale).toHaveProperty(['kitchen', 'nice.oven']);
  expect(houseForSale).not.toHaveProperty(['kitchen', 'open']);

  // Referencing keys with dot in the key itself
  expect(houseForSale).toHaveProperty(['ceiling.height'], 'tall');
});

```

## .toBeCloseTo(number, numDigits?)

소수를 비교할때 근사치로 비교한다. numDigits 를 사용하면 소수 몇번째 자리까지만 비교할 수 있다.
자바스크립트에서 0.1 + 0.2 는 0.3 이 아닌 0.30000000000000004 이므로 이 메서드를 사용하면 테스트가 가능하다.

```javascript
test('adding works sanely with decimals', () => {
  expect(0.2 + 0.1).toBeCloseTo(0.3, 5);
});
```

## toBeDefined()

expect 의 값이 정의 되어있는지를 확인한다. 즉, undefined 이 아닌지 확인한다. (null 도 true)
.not 을 붙이면 undefined 인지 확인할 수 있다.

```javascript
test('toBeDefined', () => {
  expect(null).toBeDefined();
  expect(undefined).not.toBeDefined();
});
```

## .toBeFalsy()

자바스크립트에서 다음 6개의 부정값에 대해서 테스트하는 메서드이다.
`false`, `0`, `''`, `null`, `undefined`, `NaN`

```javascript
test(".toBeFalsy()", () => {
  expect(0).toBeFalsy();
  expect(false).toBeFalsy();
  expect(null).toBeFalsy();
  expect(undefined).toBeFalsy();
  expect('').toBeFalsy();
  expect(NaN).toBeFalsy();
})
```

## .toBeTruthy()

.toBeFalsy 에서 사용되는 6개의 부정값 이외의 값은 truthy 하다.

## .toBeGreaterThan(number | bigint)

expect 의 값이 number | bigint 보다 큰지 확인하는 메서드다.

```javascript
test("toBeGreaterThan", () => {
  expect(11).toBeGreaterThan(10);
})
```

## .toBeGreaterThanOrEqual(number | bigint)

expect 의 값이 number | bigint 보다 크거나 같은지 확인하는 메서드다.

```javascript
test("toBeGreaterThanOrEqual", () => {
  expect(11).toBeGreaterThanOrEqual(11);
})
```

## .toBeLessThan(number | bigint)

expect 의 값이 number | bigint 보다 작은지 확인하는 메서드다.

```javascript
test("toBeLessThan", () => {
  expect(9).toBeLessThan(10);
})
```

## .toBeLessThanOrEqual(number | bigint)

expect 의 값이 number | bigint 보다 작거나 같은지 확인하는 메서드다.

```javascript
test("toBeLessThanOrEqual", () => {
  expect(11).toBeLessThanOrEqual(11);
})
```

## .toBeInstanceOf(Class)

instance 객체를 비교할때 사용한다.

```javascript
class A {}

expect(new A()).toBeInstanceOf(A);
expect(() => {}).toBeInstanceOf(Function);
expect(new A()).toBeInstanceOf(Function); // Error
```

## .toBeNull()

expect 값이 null 인지 확인하는 메서드이다.
.toBe(null) 을 사용하는 것과 동일하지만 테스트가 실패했을때 에러 메시지가 조금 더 보기 좋다.

```javascript
function bloop() {
  return null;
}

test('bloop returns null', () => {
  expect(bloop()).toBeNull();
});
```

## .toBeUndefined()

expect 값이 undefined 인지 확인하는 메서드이다.
toBeNull 과 마찬가지로 toBe(undefined)를 사용할 수도 있지만 에러 메시지가 조금 더 좋다고 한다.

## .toBeNaN()

expect 값이 NaN 확인하는 메서드이다.

## .toContain(item)

배열 또는 문자에서 item 을 포함하고 있는지 확인하는 메서드이다.

```javascript
test("toContain", () => {
  const a = ["foo", "bar"];
  const b = "foo_bar"

  expect(a).toContain("bar");
  expect(a).not.toContain("baz");
  expect(b).toContain("bar")
});
```

## toContainEqual(item)

배열에서 일반 특별한 구조를 가진 value (ex. JSON) 를 포함하고 있는지 확인할때 사용한다.

```javascript
test("toContain", () => {
  const a = ["foo", "bar", { delicious: true, sour: false }];

  expect(a).toContain({ delicious: true, sour: false })
});
```

toContain 메서드를 사용하면 위의 테스트는 실패되지만 이 경우 toContainEqual 메서드를 사용하면 성공한다.

```javascript
test("toContainEqual", () => {
  const a = ["foo", "bar", { delicious: true, sour: false }];
  
  expect(a).toContainEqual({ delicious: true, sour: false })
});
```

## toEqual(value)

toBe 는 정확하게 테스트하기 위해 Object.is 를 사용한다. 만약 오브젝트의 값을 체크하기를 원한다면 대신 toEqual 를 사용해야한다. toEqual 는 오브젝트 또는 배열의 모든 필드 값을 재귀적으로 체크한다.

```javascript
const can1 = {
  flavor: 'grapefruit',
  ounces: 12,
};
const can2 = {
  flavor: 'grapefruit',
  ounces: 12,
};

describe('the La Croix cans on my desk', () => {
  test('have all the same properties', () => {
    expect(can1).toEqual(can2);
  });
  test('are not the exact same can', () => {
    expect(can1).not.toBe(can2);
  });
});
```

toBe 를 사용한 expect(can1).not.toBe(can2); 의 경우 can1 의 객체와 can2 의 객체가 동일한 객체가 아니기 때문에 테스트에 성공한다.

## .toMatch(regexpOrString)

정규식에 대해 문자열을 테스트 할 수 있다.

```javascript
describe('an essay on the best flavor', () => {
  test('mentions grapefruit', () => {
    expect(essayOnTheBestFlavor()).toMatch(/grapefruit/);
    expect(essayOnTheBestFlavor()).toMatch(new RegExp('grapefruit'));
  });
});
```

## .toMatchObject(object)

객체 부분 속성에 대해 일치하는지 테스트한다.
```javascript
const houseForSale = {
  bath: true,
  bedrooms: 4,
  kitchen: {
    amenities: ['oven', 'stove', 'washer'],
    area: 20,
    wallColor: 'white',
  },
};
const desiredHouse = {
  bath: true,
  kitchen: {
    amenities: ['oven', 'stove', 'washer'],
    wallColor: expect.stringMatching(/white|yellow/),
  },
};

test('the house has my desired features', () => {
  expect(houseForSale).toMatchObject(desiredHouse);
});
```

## .toThrow(error?)

특정 함수가 호출될 때 에러를 던진다는 것을 테스트하려면 toThrow를 사용하면 된다.

```javascript
test('throws on octopus', () => {
  expect(() => {
    drinkFlavor('octopus');
  }).toThrow();
});
```

참조: [Jest 공식 문서](https://jestjs.io/docs/en/expect#tobevalue)
