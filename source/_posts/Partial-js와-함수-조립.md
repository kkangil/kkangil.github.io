---
title: Partial.js와 함수 조립
date: 2019-12-01 15:35:32
categories:
  - 함수형 프로그래밍
tags:
  - javascript
  - 함수형 프로그래밍
  - 파이프라인
  - go 함수
  - 비동기와 _.go
  - _.indent
  
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

## 파이프라인

### 즉시 실행 파이프라인, _.go와 _.mr

_.go는 파이프라인의 즉시 실행 버전이다. 첫 번째 인자로 받은 값을 두 번째 인자로 받은 함수에게 넘겨주고, 두 번째 인자로 받은 함수의 결과는 세 번째 함수에게 넘겨준다. 이것을 반복하다가 마지막 함수의 결과를 리턴해 준다.

```javascript
_.go(10, // 첫번째 인자
  function (a) { return a * 10 }, // 100
  function (a) { return a - 50 }, // 50
  function (a) { return a + 10 } // 60
)
```

_.go는 Multiple Results를 지원한다. _.mr 함수를 함께 사용하면 다음 함수에게 2개 이상의 인자들을 전달할 수 있다.

<!-- more -->

```javascript
_.go(10,
  function (a) { return _.mr(a * 10, 50) },
  function (a, b) { return a - b },
  function (a) { return a + 10 }
)
// 60
```

_.go의 첫 번째 인자는 두 번째 인자인 함수가 사용할 인자이며 두 번째 부터는 파이프라인에서 사용할 함수들이다. _.go의 두 번째 인자인 함수, 즉 최초 실행될 함수에게 2개 이상의 인자를 넘끼고자 한다면 그때도 _.mr을 사용하면 된다.

```javascript
_.go(_.mr(2, 3),
  function (a, b) {
    return a + b;
  },
  function (a) {
    return a * a
  }
)

// 25
```

위의 코드처럼 _.mr로 인자들을 감싸서 넘겨주면, 다음 함수는 인자를 여러개로 펼쳐서 받게 된다. _.mr(2, 3)은 하나의 값이지만 _.go 내부에서 인자를 펼쳐서 넘겨주어, 2와 3이 첫 번째 함수의 a와 b가 된다.
_.go는 이미 정의되어 있는 함수와 조합하거나 화살표 함수와 사용할 때 특히 표현력이 좋다.

```javascript
function add(a, b) {
  return a + b;
}
function square(a) {
  return a * a;
}

_.go(_.mr(2, 3), add, square);
_.go(_.mr(2, 3), (a, b) => a + b, a => a * a)
```

### 함수를 만드는 파이프라인 _.pipe

_.go가 즉시 실행하는 파이프라인이라면 _.pipe는 실행할 준비가 된 함수를 리턴하는 파이프라인 함수다. 그 외 모든 기능은 _.go와 동일하다.

```javascript
var f1 = _.pipe(add, square);
f1(2, 3); // 25

var f2 = _.pipe((a, b) => a + b, a => a * a);
f2(2, 3) // 25
```

### 부분 커링 함수와의 조합

파이프라인 함수를, 함수를 리턴하는 함수와 함께 사용해 보자.

```javascript
var products = [
  { id: 1, name: "후드 집업", discounted_price: 6000, price: 10000 },
  { id: 2, name: "코잼 후드티", discounted_price: 8000, price: 8000 },
  { id: 3, name: "A1 반팔티", discounted_price: 6000, price: 6000 },
  { id: 4, name: "코잼 반팔티", discounted_price: 5000, price: 6000 },
]

_.go(products,
  _.filter(p => p.discounted_price < p.price), // 1
  _.sortBy('discounted_price'), // 2
  _.first, // 3
  _.val('name') // 4
)

// 코잼 반팔티
```

1. `products` 중에 할인 중인 상품만 남긴다.
2. `discounted_price` 가 낮은 순으로 정렬한다.
3. 첫 번째를 꺼낸다.
4. product.name 을 확인한다.

Partial.js의 _.filter, _.sortBy, _.val은 모두 부분 커링이 된다. 모두 인자를 하나만 넘겨 앞으로 실행될 함수를 리턴 받았다. 그렇게 만들어진 함수들은 _.go를 통해 순서대로 실행된다.

```javascript
// 할인 상품 중 가격이 가장 높은 상품의 이름
_.go(products,
  _.filter(p => p.discounted_price < p.price),
  _.sortBy('discounted_price'),
  _.last,
  _.val('name'),
  console.log
)
// 후드 집업

// 할인 상품 중 할인액이 가장 높은 상품의 이름
_.go(
  products,
  _.filter(p => p.discounted_price < p.price),
  _.sortBy(p => p.discounted_price - p.price),
  _.first,
  _.val('name'),
  console.log
)
// 후드 집업

// 할인 상품 중 할인액이 가장 낮은 상품의 이름
_.go(
  products,
  _.filter(p => p.discounted_price < p.price),
  _.max(p => p.discounted_price - p.price),
  _.val('name'),
  console.log
)
// 코잼 반팔티
```

중간중간 들어간 함수들의 조합을 약간씩 수정하여, 다른 로직으로 쉽게 변경할 수 있다. 할인이 진행 중인 상품들 중 제일 비싼 상품을 꺼낸다든지, 할인이 되지 않는 상품 중 제일 싼 상품을 꺼낸다든지 하는 로직으로 변경하는 것이 위에서 알 수 있듯 매우 쉽다.

### 보조 함수로 사용하는 파이프라인

파이프라인을 보조 함수로 만들기 위해 사용하는 것도 좋다. 파이프라인을 _.each, _.map, _.reduce 등과 같은 고차 함수들의 보조 함수로 사용해 보자.

```javascript
_.go(
  products,
  _.filter(p => p.discounted_price < p.price),
  _.map(_.pipe(_.identity, _.pick(['id', 'name']), _.values)),
  console.log
)
// [[1, "후드 집업"], [4, "코잼 반팔티"]]
```

_.filter 에서는 할인 중인 상품만 남게 된다. _.map 에서는 iteratee를 파이프라인으로 만들었다. _.pipe는 _.identity를 통해 iteratee의 첫 번째 인자인 product만 리턴하게 되고, 부분 커링된 _.pick을 통애 id와 name만 남은 객체로 만들게 된다. 마지막으로 _.values를 통해 값만 남겼다.

### 비동기와 _.go

_.go는 비동기 제어를 지원한다. Promise가 체인 방식에 함수적인 아이디어를 가미하여 비동기 상황을 제어했다면, _.go는 함수적으로만 비동기를 제어한다. _.go는 내부적으로 함수들을 순차적으로 실행해 나가면서 비동기 함수의 결과를 재귀 함수로 꺼낸 후, 다음 함수에게 이어주는 식으로 비동기 상황을 제어한다.
_.go와 _.pipe는 일반 콜백 함수와 Promise를 모두 지원한다. _.go와 _.pipe는 기본적으로 동기로 동작하지만 파이프라인의 함수들을 실행하는 도중에 Promise 객체가 리턴되거나 jQuery의 then과 같은 Deferred Object가 리턴되거나 _.callback 함수를 이용했다면, 내부적으로 비동기를 제어하는 파이프라인으로 변한다.
Promise의 경우에는 함수가 실행되면 즉시 setTimeout을 일으키고, 일단 { then: ... } 을 리턴하게 되므로 동기 함수들과 중첩 사용을 할 수 없다. 그러나 _.go의 경우에는 비동기 상황이 없을 경우 결과를 즉시 리턴하기 때문에, 이후 연속적으로 실행되는 함수들이 반드시 비동기성을 띌 필요가 없다.

```javascript
_.go(
  10,
  _.callback(function (a, next) {
    setTimeout(function () {
      next(a + 10)
    }, 100)
  }),
  function (a) { // next를 통해 받은 결과 a
    console.log(a);
  }
)
```

위 상황에서 _.go는 첫 번째 함수를 실행하면서 내부적으로 next를 만들어 함수에게 넘긴다. next에게 결과를 전달하면 그 결과는 다음 함수의 인자로 넘어가게 된다. 다시 바깥으로 나와 그 다음 함수들을 순차적으로 실행하므로 콜백 지옥에서 벗어날 수 있다.

```javascript
function asyncCallback() {
  function add(a, b, next) {
    setTimeout(function () {
      next(a + b);
    }, 1000);
  }

  function sub(a, b, next) {
    setTimeout(function () {
      next(a - b);
    }, 1000);
  }

  function mul(a, b, next) {
    setTimeout(function () {
      next(a * b);
    }, 1000);
  }

  function log(msg, next) {
    setTimeout(function () {
      console.log(msg);
      next(msg);
    }, 1000);
  }

  _.go(
    _.mr(5, 10),
    _.callback(
      function (a, b, next) {
        add(a, b, next);
      },
      function (result, next) {
        sub(result, 10, next);
      },
      function (result, next) {
        mul(result, 10, next);
      },
      function (result, next) {
        log(result, next);
      }
    )
  )
}

asyncCallback(); // 50
```

연속적으로 비동기 함수가 사용되어야 한다면 위와 같이 _.callback 함수에 여러개의 함수를 넘겨도 되고, 아래와 같이 미리 _.callback 패턴이라고 지정해 두면 더욱 간결하게 코딩할 수 있다.

```javascript
var add = _.callback(function (a, b, next) {
  setTimeout(function () {
    next(a + b);
  }, 1000);
});

var sub = _.callback(function (a, b, next) {
  setTimeout(function () {
    next(a - b);
  }, 1000);
});

var mul = _.callback(function (a, b, next) {
  setTimeout(function () {
    next(a * b);
  }, 1000);
});

var log = _.callback(function (msg, next) {
  setTimeout(function () {
    console.log(msg);
    next(msg);
  }, 1000);
});

_.go(
  _.mr(5, 10),
  add,
  function (result) {
    return sub(result, 10);
  },
  function (result) {
    return mul(result, 100);
  },
  function (result) {
    return log(result);
  }
)
```

위 코드 처럼 _.callback 함수로 미리 정의해 둔 ad, sub 등의 함수를 실행하면 그 함수는 Promise 객체를 리턴한다. 정확히는 Promise가 지원되는 환경에서는 Promise 객체를 리턴하고, Promise가 지원되지 않는 환경에서는 Partial.js 내부에 구현된 Promise 객체를 리턴한다. Promise 객체를 파이프라인이 다시 받아도 다시 비동기를 제어한다. 따라서 Promise가 지원되지 않는 환경에서도 파이프라인식의 비동기 제어가 가능하다.

### 중간에 멈추고 나가기

일반 함수에서는 함수 중간 어디서든 return문으로 함수를 빠져 나올 수 있다. Partial.js는 파이프라인에서도 이와 같은 일이 가능하도록 지원하는데, 바로 _.stop이라는 함수를 이용하면 된다.

```javascript
_.go(
  null,
  function () { console.log(1) },
  function () { console.log(2) },
  function () { return _.stop() },
  function () { console.log(3) }
)
```

1과 2만 찍힌 후 세번째 함수는 실행되지 않고 파이프라인 전체를 나오게 된다. _.go의 최종 결과도 함께 전달하고 싶다면 _.stop 함수에게 전달하면 된다.

```javascript
var result = _.go(
  null,
  function () { console.log(1) },
  function () { console.log(2) },
  function () { return _.stop("Hi") },
  function () { console.log(3) }
);

console.log(result); // Hi
```

## 비동기

### 코드 변경없이 비동기 제어가 되는 고차 함수

Partial.js의 _.each, _.map, _.reduce 등의 주요 함수들은 _.go와 _.pipe 처럼 하나의 함수로 동기와 비동기 상황이 모두 대응되도록 되어 있다. Partial.js의 함수를 이용하면 비동기 상황에서도 동기 상황과 동일한 코드를 작성할 수 있고, 비동기 함수와 동기 함수의 조합도 가능하다.

```javascript
// 1
console.log(JSON.stringify(_.map([1, 2, 3], function (v) {
  return new Date();
})))

//  ["2019-12-01T08:20:00.422Z","2019-12-01T08:20:00.422Z","2019-12-01T08:20:00.422Z"]

// 2
_.map([1, 2, 3], function () {
  return new Promise(function (resolve) {
    setTimeout(function () {
      resolve(new Date())
    }, 1000);
  });
}).then(function (result) {
  console.log(JSON.stringify(result))
})
// ["2019-12-01T08:20:01.424Z","2019-12-01T08:20:02.424Z","2019-12-01T08:20:03.428Z"]
```
같은 _.map 함수지만 1은 즉시 완료되었고, 2는 3초 정도의 시간이 걸려 완료되었다. Partial.js의 _.each, _.map, _.reduce 등은 첫 번째 iteratee의 결과가 Promise 객체일 경우 내부적으로 for문 대신 재귀로 변경된다. 2의 경우 iteratee가 각각 모두 1초 정도의 시간이 걸렸기 때문에 배열에 1초 정도의 차이를 가진 값들이 담겼다.
위 코드는 둘 다 동일한 _.map 함수로 작성하기는 했지만 동기 상황의 코드와 비동기 상황의 전체적인 코드 구조가 다르다. _.go, _.pipe 등을 이용하면 두 가지 코드 모두 동일한 구조를 갖도록 만들 수 있다.

```javascript
// 1
_.go(
  [1, 2, 3],
  _.map(function () { return new Date() }),
  JSON.stringify,
  console.log
);

// 2
_.go(
  [1, 2, 3],
  _.map(function () {
    return new Promise(function (resolve) {
      setTimeout(function () {
        resolve(new Date())
      }, 1000);
    });
  }),
  JSON.stringify,
  console.log
)
```

_.map의 iteratee에서 Promise 함수가 아닌 일반 비동기 함수를 사용해야 한다면 다음과 같이 사용하면 된다.

```javascript
_.go(
  [1, 2, 3],
  _.map(_.callback(function (val, i, list, next) {
    setTimeout(function () {
      next(new Date());
    }, 1000)
  })),
  JSON.stringify,
  console.log
)
```

마지막 인자로 next가 들어오고 next를 통해 결과를 전달하면, 그 결과가 하나씩 _.map의 결과인 새로운 배열에 쌓여간다. new Promise를 사용하는 것 보다 간결하다.

Partial.js는 iteratee의 결과나 iteratee의 속성에 따라 동기 로직과 비동기 로직을 알아서 선택하도록 되어있다. 동기 로직으로 돌아가야 할 경우에는 for문을 사용하고 비동기 로직을 사용해야 할 경우에만 재귀로 변경되기 때문에 성능적으로도 문제가 없다.

### 비동기 결과를 기다리는 if문, _.if

아래 is_1, is_2 함수는 결과를 즉시 리턴하는 함수다. 아래와 같은 함수는 if문과 같은 조건절에서 사용할 수 있다.

```javascript
var is_1 = function (a) {
  return a === 1;
};
var is_2 = function (a) {
  return a === 2;
};

function test1(a) {
  if (is_1(a)) return '1입니다.'
  else if (is_2(a)) return '2입니다.'
  else return '1도 아니고 2도 아닙니다.'
}

console.log(test1(2)); // 2입니다.
```

그런데 만약 is_1과 같은 함수가 즉시 값을 리턴할 수 없고, 데이터베이스에 다녀와야 하거나 HTTP 통신이 있어야 하면 상황은 달라진다. 자바스크립트에서는 아래와 같은 상황을 if문으로 제어할 수 없다.

```javascript
var is_1_async = function (a) {
  return new Promise(function (resolve) {
    setTimeout(function () {
      resolve(a === 1);
    }, 1000);
  })
};

var is_2_async = function (a) {
  return new Promise(function (resolve) {
    setTimeout(function () {
      resolve(a === 2);
    }, 1000);
  })
};

function test2(a) {
  if (is_1_async(a)) return '1입니다.'
  else if (is_2_async(a)) return '2입니다.'
  else return '1도 아니고 2도 아닙니다.'
};

console.log(test2(2)); // 1입니다. (정상적으로 동작하지 않음)
```

is_1_async 함수의 진짜 결과는 false이겠지만  is_1_async가 즉시 Promise 객체를 리턴하기 때문에 if의 조건문 입장에서는 true가 된다. is_1_async가 콜백 패턴으로 이루어진 일반 함수 였다고 하더라고, undefined가 즉시 리턴될 것이므로 역시 정상적인 결과를 만들수 없다.

```javascript
var test4 =
  _.if(is_1_async, function () { return '1입니다.' })
    .else_if(is_2_async, function () { return '2입니다.' })
    .else(function () { return '1도 아니고 2도 아닙니다. ' });

test4(2).then(console.log); // 2입니다.
```

위 함수는 동기 함수를 사용했던 test1과 유사한 구조를 가지고 있다. _.constant나 화살표 함수를 사용하면 더욱 간결해진다. _.go와 함께 사용한 버전도 확인해 보자.

```javascript
var test5 =
  _.if(is_1_async, _.constant('1입니다.'))
    .else_if(is_2_async, _.constant('2입니다.'))
    .else(_.constant('1도 아니고 2도 아닙니다. '));

test5(2).then(console.log);

// 화살표함수
var test6 =
  _.if(is_1_async, () => '1입니다.')
    .else_if(is_2_async, () => '2입니다.')
    .else(() => '1도 아니고 2도 아닙니다. ');
test6(1).then(console.log)

// _.go
_.go(
  3,
  _.if(is_1_async, _.constant('1입니다.'))
    .else_if(is_2_async, _.constant('2입니다.'))
    .else(_.constant('1도 아니고 2도 아닙니다. ')),
  console.log
)
```

동기와 비동기 상황을 동시에 대응하는 함수를 쉽게 만드는 팁이 있다. 해당 함수의 로직을 _.go나 _.pipe로 구현해 두는 것이다. 그렇게 하면 자동으로 동기 상황과 비동기 상황에 맞춰 로직을 알아서 변경시키기 때문에 추가적인 다른 작업을 하지 않아도 된다.

## 고차 함수

### _.all, _.spread

이 두 함수는 파이프라인과 함께 사용할 때 유용한 함수다. _.all과 _.spread 모두 받은 인자를 받은 함수들에게 전달하는 함수들인데, _.all은 받은 모든 인자를 모든 함수들에게 동일하게 전달하고, _.spread는 받은 인자들을 하나씩 나눠 준다.

```javascript
_.all(10, 5, [
  function (a, b) { return a + b },
  function (a, b) { return a - b },
  function (a, b) { return a * b }
])

_.spread(10, 5, [
  function (a) { return a * a },
  function (b) { return b * b }
])
```

_.all은 받아 둔 모든 인자를 마지막 인자로 들어온 배열 안에 있는 모든 함수들에게 동일하게 전달한다. _.spread는 받은 인자를 하나씩 나눠서, 받은 함수들에게 하나씩 전달하는 함수다.
_.all과 _.spread에게 인자 전달 없이 함수들만 전달하면 함수를 리턴하는 함수로 동작하여 파이프라인 등에서 사용하기 좋다.

```javascript
_.go(
  10,
  _.all(
    function (a) { return a + 5 },
    function (a) { return a - 5 },
    function (a) { return a * 5 }
  ),
  _.spread(
    function (a) { return a + 1 },
    function (b) { return b + 2 },
    function (c) { return c + 3 }
  ),
  console.log
)

// 16 7 53
```

## 파이프라인2

### _.go에서 this 사용

Partial.js의 _.pipe가 this를 지원하는 것을 확인했었다. _.go에서도 this 지원이 가능하다.

```javascript
var user = { name: "Cojamm" };
_.go.call(user, 32,
  function (age) {
    this.age = age;
  },
  function () {
    console.log(this.name);
  },
  function () {
    this.job = "Rapper";
  }
)

console.log(user); // {name: "Cojamm", age: 32, job: "Rapper"}
```

_.go.call의 call은 Fucntion.prototype.call 이다. _.go는 일반 함수이므로 call을 사용하는게 자연스럽다. _.go 는 call을 통해 전달받은 this를 파이프라인 내 함수들에게 전달한다. apply 역시 사용할 수 있다.

### 또 다른 파이프라인, _.indent

자바스크립트에서는 부모 스코프와 자식 스코프라는 개념이 있다. 자식 스코프는 부모 스코프의 지역 변수를 참조할 수 있다. 이러한 중첩 구조의 접근 방식이 파이프라인 내에서도 필요했는데, 이를 위한 함수가 바로 _.indent다. _.indent는 이름처럼 _.indent가 중첩될 때마다 this와 arguments를 한 단계씩 안으로 들여쓰는 콘셉트를 가지고 있다.

```javascript
var f1 = _.indent(
  function () {
    console.log(this, arguments); // 1
    return 'hi';
  },
  function () {
    console.log(this, arguments) // 2
  }
)

f1(1, 2);
```

_.indent는 _.pipe처럼 함수를 리턴하는 파이프라인이다. 함수들의 연속 실행이 준비된 함수를 리턴하여 f1에 담았고 실행했다.
1과 2의 결과에는 차이가 있다. console.log 에게 두 번째 인자로 넘긴 arguments에 담긴 값이 다르다. 1은 파이프라인의 첫 번째 함수이기에 f1을 통해 넘겨진 인자 [1,2]가 arguments에 들어온다. 2는 두 번째 함수이고, 이전 함수에서 'hi'를 리턴했기에 ['hi']가 arguments에 담겨 있다.
파이프라인으로 코딩하다 보면 파이프라인의 최초 함수가 아닌 중간에 있는 함수에서 파이프라인이 최초로 받은 인자를 알고 싶을 때가 있다. 이것은 클로저를 사용하는 것으로도 자연스럽게 해결된다.
_.indent의 해결법은 파이프라인 내부의 함수들은 중간 어디서든 this.arguments로 파이프라인의 최초 인자들에 접근할 수 있다.

_.indent의 this는 한 번의 파이프라인 사용 후 휘발되는 this다. 파이프라인 전체에서 공유하고자 하는 값이 있을 때 값을 이어주는 식으로 사용할 수도 있다. 또한 인스턴스를 만들어 두고 계속 사용하는 this가 아닌, 함수를 실행할 때마다 새로 생겼다 사라지는 this이므로 부수효과가 최소화되고 메모리 사용에도 문제가 없다.

```javascript
var f2 = _.indent(
  function (a) { this.b = a + 10; },
  function () { },
  function () { },
  function () { console.log(this.b) },
)

f2(5); // 15
f2(7); // 17
```

_.indent에서 this는 이번에 실행할 때 열린 부모 스코프를 대신하는 역할이므로, 그 this의 key/value를 확장한다는 것은 새로운 변수를 할당하는 것 이라고 볼 수 있다. this에 값을 단 후 값을 변경하지 않는다면 부수 효과로부터 자유로워진다. 하나의 큰 함수 안에서 변수에 담고 그 변수에 담긴 값을 변경하지 않는 것과 동일하다.

```javascript
var f3 = _.indent(
  function (a) {
    this.b = a + 10;
  },
  _.indent(
    function () {
      this.b = 20;
      console.log(this.b); // 20
      console.log(this.parent.b) // 15
    },
    function () {
      console.log(this.parent.arguments); // [5]
    }
  ),
  function () {
    console.log(this.b);
  }
)

f3(5);
```

parent를 통해 부모를 찾아갈 수 있고, this.parent.arguments로 파이프라인의 최초 인자도 찾아갈 수 있다. 자바스크립트에서 값에 접근할 수 있다는 것은 값을 변경할 수도 있다는 의미이므로 위험하다. 이것은 모든 값 혹은 변수에 해당하는 이야기다. 그러나 값을 참조하는 식으로 접근한다면, 여기저기에서 값에 접근할 수 있다는 것 자체는 문제가 되지는 않는다.

### 무조건 비동기로 동작하는 _.async

_.go, _.pipe, _.indent는 파이프라인 내부의 함수에서 비동기 결과가 나올 경우 비동기 제어를 시작하도록 구현되어 있다. 그러므로 비동기 상황이 생기지 않는다면 즉시 일반 값이 리턴되고, 비동기 상황이 생겼을 때만 Promise 객체가 리턴된다. 때로는 내부 함수의 결과와 상관없이 무조건 비동기를 일으키고 싶을 때가 있다. _.async는 이럴 때 사용하기 적합한, 무조건 비동기로 동작하는 파이프라인이다.

```javascript
_.go.async(1, function (a) {
  return a;
}).then(console.log);
console.log(2);

// 2
// 1
```

참조: [함수형 자바스크립트 프로그래밍](http://www.yes24.com/Product/Goods/56885507)
