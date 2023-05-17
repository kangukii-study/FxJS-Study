# Section 5. 코드를 값으로 다루어 표현력을 높이기

이전에 직접 정의한 map, filter, reduce 코드를 값으로 다루어서 전체적인 코드의 표현력을 높일 수 있다. 이를 위해서 주로 사용하는 함수들인 go, pipe, curry에 대해서 알아보려고 한다.

## Index

1. [go](#go)
2. [pipe](#pipe)
3. [curry](#curry)

---

## Content

### go

- go 함수는 매개변수를 받아서 결과를 바로 산출하는 함수이다.
- 첫번째 인자는 시작이 되는 값을 받고 나머지는 함수로 받아서 첫번째 인자가 두번째 함수로 전달이 되어 평가가 되고 이어서 다음 함수로 넘어가서 마찬가지로 평가가 되는 과정이 진행된다.

> - 함수를 위에서부터 아래로, 왼쪽에서부터 오른쪽으로 평가하면서 연속적으로 함수를 실행하고 이전에 실행된 함수의 결과를 다음 함수에 전달하는 함수이다.<br>
>
> - go함수로 표현하는 것은 함수형 프로그래밍에서 코드를 값으로 다루는 아이디어라고 생각하면 된다.<br>
>
> - go함수를 통해서 어떤 함수가 코드인 함수를 받아서 평가하는 시점을 원하는대로 다룰 수 있고 코드의 표현력도 높일 수 있다.<br>

#### 기본 동작 과정

- go 함수는 인자로 함수 리스트를 받게 된다.

```javascript
go([
    0,
    a = > a + 1,
    a = > a + 10,
    a = > a + 100,
    console.log
]);
```

- 전개 연산자를 통해서 인자를 받게된다.

```javascript
const go = (...list) => {
  console.log(list);
};

go(
    0,
    a = > a + 1,
    a = > a + 10,
    a = > a + 100,
    console.log
);   // [0, f, f, f]
```

- 첫 번째 값을 초기값으로 설정하고 두 번째 인자부터 함수의 리스트들을 받아서 반환값을 각 함수로 넘긴다.

```javascript
const reduce = (f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }

  for (const value of iter) {
    acc = f(acc, value);
  }
  return acc;
};

const go = (...args) => reduce((a, f) => f(a), args);

go(
  0,
  (a) => a + 1, // 1
  (a) => a + 10, // 11
  (a) => a + 100, // 111
  console.log
); // 111
```

#### go 함수를 사용해서 가독성 좋은 코드 작성하기

```javascript
const products = [
  { name: "반팔티", price: 15000 },
  { name: "긴팔티", price: 20000 },
  { name: "핸드폰케이스", price: 15000 },
  { name: "후드티", price: 30000 },
  { name: "바지", price: 25000 },
];

console.log(
  reduce(
    add,
    map(
      (p) => p.price,
      filter((p) => p.price < 20000, products)
    )
  )
);
```

위와 같은 코드가 있다고 가정할 때 go 함수를 사용해서 조금 더 이해하기 쉽게 코드를 작성해보자!

```javascript
go(
  products,
  (products) => filter((p) => p.price < 20000, products), // 2만원 미만인 상품들만 추림
  (products) => map((p) => p.price, products), // 가격만 뽑아옴
  (prices) => reduce(add, prices), // 가격들을 다 더해서 1개의 값으로 축약
  console.log
);
```

`이렇게 작성하게 되면 위의 코드보다는 코드양이 많아지고 간결하지는 않지만 개발자가 읽기 편해진 코드라고는 할 수 있다.`

<br>

### pipe

- go함수와는 달리 함수를 리턴하는 함수
- `go함수는 인자값으로 들어온 함수에서 즉시 실행시켜 결과값을 반환해서 다음 함수로 전달하지만 pipe함수는 함수 자체를 반환시켜서 최종적으로 인자값으로 받은 함수리스트를 합성시켜 합성된 함수를 가지고 로직을 수행한다.`

#### 기본 동작 과정

pipe 함수는 내부적으로 go함수를 사용하게 된다.

- 기본적으로 함수들을 전개 연산자를 통해 매개변수로 받음 (`(...fs)`)
- 반환된 함수에서 초기값을 매개변수로 전달함 (`a`)
- 최종적으로 go함수를 호출시킨다 (`go(a, ...fs)`)

```javascript
const pipe =
  (...fs) =>
  (a) =>
    go(a, ...fs);

const f = pipe(
  (a) => a + 1,
  (a) => a + 10,
  (a) => a + 100
); // 3개의 함수를 실행시키는 f라는 하나의 함수를 만든 것

console.log(f(0)); // 111
```

#### pipe에 기능 추가

위의 pipe에서는 최초 인자값으로 하나만 받기 때문에 처음 인자값을 여러개줄 수 없다. 따라서 위의 코드로 pipe를 사용하려고 하면 `f(add(0,1))` 이런식으로 f에 하나의 값만 들어가게 했어야 했다.

이 불편함을 해소하고자 여러개의 인자값을 받을 수 있도록 리팩토링을 해보자면...

```javascript
const pipe =
  (f, ...fs) =>
  (...as) =>
    go(f(...as), ...fs);

const f = pipe(
  (a, b) => a + b,
  (a) => a + 10,
  (a) => a + 100
);

f(0, 1);
```

이런식으로 pipe 함수 구현에서는 첫번째 함수 `f`와 나머지 함수들인 `...fs`를 따로 받는다. 그렇게 리턴된 합성함수는 여러개의 인자 `...as`를 받아서 f함수에 넣고 결과를 나머지 함수들이 받으면서 차례대로 실행된다.

<br>

### curry

- 함수를 인자로 받아서 함수를 리턴한다.
- `리턴된 함수가 실행되었을 때 인자가 2개 이상이면 받아둔 함수를 즉시 실행하고, 인자가 2개 미만이면 함수를 다시 리턴하고 그 이후에 받은 인자들을 합쳐서 실행한다.`

```javascript
const curry =
  (f) =>
  (a, ..._) =>
    _.length ? f(a, ..._) : (..._) => f(a, ..._);

const mult = curry((a, b) => a * b);
console.log(mult(3)); // 다음 인자가 올때까지 기다림
console.log(mult(3)(2)); // 6

const multi3 = mult(3);
console.log(multi3(2)); // 6
```

#### go + curry

- currying (curry함수를 사용하게 되면)을 사용하게 되면 함수를 부분적으로 사용할 수 있게 된다.
- `기존에 구현한 map, filter, reduce함수에 curry를 적용하면 go함수에서 map, filter, reduce를 좀더 축약해서 직관적으로 go함수를 사용할 수 있다.`
- `map, filter, reduce는 인자가 최소 2개여야 동작하기 때문에 curry를 감싸게 되면 부족한 인자가 올 때까지 대기하는 함수가 되기 때문이다.`

지금까지의 내용을 한번 코드로 정리해봤다.

```javascript
const curry =
  (f) =>
  (a, ..._) =>
    _.length ? f(a, ..._) : (..._) => f(a, ..._);

// map
const map = curry((f, iter) => {
  const res = [];
  for (const el of iter) {
    res.push(f(el));
  }
  return res;
});

// filter
const filter = curry((f, iter) => {
  let res = [];
  for (const a of iter) {
    if (f(a)) res.push(a);
  }
  return res;
});

// reduce
const reduce = curry((f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }
  for (const a of iter) {
    acc = f(acc, a);
  }
  return acc;
});

// add 함수
const add = (a, b) => a + b;

// go 함수
const go = (...args) => reduce((a, f) => f(a), args);

// 예제
const products = [
  { name: "반팔티", price: 15000 },
  { name: "긴팔티", price: 20000 },
  { name: "핸드폰케이스", price: 15000 },
  { name: "후드티", price: 30000 },
  { name: "바지", price: 25000 },
];

// 초기 예제코드
console.log(
  reduce(
    add,
    map(
      (p) => p.price,
      filter((p) => p.price < 20000, products)
    )
  )
);

// 위 예제 코드를 go를 사용해서 가독성 좋은 코드로 리팩토링
go(
  products,
  (products) => filter((p) => p.price < 20000, products),
  (products) => map((p) => p.price, products),
  (prices) => reduce(add, prices),
  console.log
);

// filter, map, reduce에 curry를 적용 - (1)
go(
  products,
  (products) => filter((p) => p.price < 20000)(products),
  (products) => map((p) => p.price)(products),
  (prices) => reduce(add)(prices),
  console.log
);

// filter, map, reduce에 curry를 적용 - (2)
go(
  products,
  filter((p) => p.price < 20000),
  map((p) => p.price),
  reduce(add),
  console.log
);
```

맨 마지막처럼 코드를 작성할 수 있다. `이것이 map, filter, reduce에 curry를 적용한 이유라고 보면 될 것 같다. 확실히 가독성이 좋고 결국엔 코드도 간결해진 것을 확인할 수 있다.`

#### pipe를 사용해서 함수 조합으로 함수 만들기

pipe를 사용하게 되면 함수들을 의미적으로 하나의 함수로 합쳐서 사용할 수 있다.

예제코드를 넣어봤다.

```javascript
go(
  products,
  filter((p) => p.price < 20000),
  map((p) => p.price),
  reduce(add),
  console.log
);

go(
  products,
  filter((p) => p.price >= 20000),
  map((p) => p.price),
  reduce(add),
  console.log
);
```

위의 코드를 보면 지금 map과 reduce부분이 중복이다. 이것을 pipe를 통해 간결하게 표현이 가능하다.

```javascript
const total_price = pipe(
  map((p) => p.price),
  reduce(add)
);

go(
  products,
  filter((p) => p.price < 20000),
  total_price
  console.log
);

go(
  products,
  filter((p) => p.price >= 20000),
  total_price
  console.log
);
```

여기서 더 쪼갠다면 filter와 total_price 부분을 수정할 수 있을 것 같다.

```javascript
const total_price = pipe(
  map((p) => p.price),
  reduce(add)
);

const base_total_price = predi => pipe(
  filter(predi),
  total_price
);

go(
  products,
  base_total_price((p) => p.price < 20000)
  console.log
);

go(
  products,
  base_total_price((p) => p.price >= 20000)
  console.log
);
```

---

### Reference

- https://velog.io/@younoah/functional-js-%ED%95%A8%EC%88%98%EC%9D%98-%EC%A4%91%EC%B2%A9-go-pip-curry
- https://medium.com/%EC%98%A4%EB%8A%98%EC%9D%98-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/%ED%95%A8%EC%88%98%ED%98%95-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-pipe-c80dc7b389de
- https://jinn2u.tistory.com/17
