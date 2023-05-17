# Section 4. Map, Filter, Reduce

## Index

0. [Map](#Map)
1. [이터러블 프로토콜을 따른 map의 다형성](#이터러블-프로토콜을-따른-map의-다형성)
2. [Filter](#Filter)
3. [Reduce](#Reduce)

---

## Content

### Map

- `map()은 배열 각 요소에 접근해서 주어진 함수를 수행한 결과를 모아 새로운 배열을 반환하는 메서드이다.`
- 이터러블 프로토콜을 따르는 함수
- 이터러블을 순회하면서 각각의 값을 인자로 넘어온 f를 이용해 수행하고 반환함

```javascript
const map = (f, iter) => {
  let res = [];
  for (const i of iter) {
    res.push(f(i));
  }
  return res;
};
```

위 예시는 기존의 map이 아닌 custom한 map 메서드이다. <br>
우리가 평소에 사용했던 `Array.prototype.map`과 약간의 동작 차이가 있다.<br>

> 기존에 구현되어 있던 map을 사용하는 것이 아니라 커스텀하는 이유는 다른 측면에서 더욱 효율적으로 동작시키게 하기 위해서라고 생각하지만 아직은 조금 이해가 안되는 부분이 있다.
>
> 강의를 계속 들어보면서 이 부분을 해소해야 할 것 같다.

<br>

### 이터러블 프로토콜을 따른 map의 다형성

- 이터러블 프로토콜을 따르는 객체들은 모두 map을 사용할 수 있다.
- map 메서드는 이터러블 프로토콜을 따르는 for문을 사용해서 순회할 수 있다.

#### document.querySelectorAll은 Array.prototype.map이 동작할까요?

- document.querySelectorAll은 `Array를 상속받은 객체가 아니다.` (NodeList)
- `따라서 Array.prototype.map이 따로 정의되어 있지 않기 때문에 동작하지 않는다.`

```javascript
document.querySelectorAll('*').map(...);   // Error!
```

#### 하지만! 이터러블 프로토콜을 따르기 때문에 순회로직 사용이 가능하다!

- `Array.prototype.map이 아닌 우리가 위에서 커스텀한 map으로 사용이 가능하다!`
- `document.querySelectorAll이 이터러블 프로토콜을 따르고 있음!`

```javascript
log(map((el) => el.nodeName, document.querySelectorAll("*"))); // ['HTML', 'HEAD', 'SCRIPT', ...]

const iter = document.querySelectorAll("*")[Symbol.iterator]();
log(iter.next()); // { value: html, done: false }
log(iter.next()); // { value: head, done: false }
log(iter.next()); // { value: script, done: false }

// ====================================================================

function* gen() {
  yield 2;
  yield 3;
  if (false) yield 5;
  yield 4;
}

// 제너레이터도 이터러블이기 때문에 사용할 수 있다.
map((a) => a * a, gen()); // [4, 9, 16]
```

- `결론적으로 이터러블 프로토콜을 따르는 경우 위에서 커스텀한 map 메서드를 사용할 수 있다.`<br>
- `이터러블 프로토콜을 따르는 함수들을 사용하는 것은 다른 함수들과의 조합성이 좋아지고 유연성 및 다형성이 올라간다고 할 수 있다.`

#### key-value 쌍을 사용하는 Map 예시

```javascript
let m = new Map();
m.set("a", 10);
m.set("b", 20);

const it = m[Symbol.iterator](); // new Map()도 이터러블임! -> 특정 이터레이터 반환
log(map(([k, a]) => [k, a * 2], m)); // [['a', 20], ['b', 40]]
log(new Map(map(([k, a]) => [k, a * 2], m))); // 이 방법으로 다시 Map 객체로 만들 수 있다.
```

위 예시를 통해 커스텀으로 구현한 map 메서드를 가지고 새로운 Map 객체를 만든 것을 확인할 수 있다.

<br>

### Filter

- `filter()는 배열데이터가 있을 때 특정 조건에 맞는 데이터만 추려내는 함수`

```javascript
const filter = (f, iter) => {
  let res = [];
  for (const a of iter) {
    if (f(a)) res.push(a);
  }
  return res;
};
```

- map과 마찬가지로 filter를 custom해보면 거의 동일한 형태로 구현할 수 있다.

<br>

### Reduce

- `reduce()는 배열의 각 요소에 대해 주어진 Reducer 함수를 실행하고 map과 filter와는 달리 하나의 결과값을 반환하는 함수이다.`
- `보통 배열을 하나의 값으로 축약할 때 사용한다.`

우리가 알고있는 reduce는 다음과 같이 사용한다.

```javascript
const numbers = [1, 2, 3, 4];
const numbersSum = numbers.reduce(
  (accumulator, currentValue, currentIndex, array) => {
    console.log(accumulator, currentValue, currentIndex, array);

    return accumulator + currentValue;
  }
);

console.log(numbersSum);
```

하지만 우리는 reduce를 custom해서 정의를 해보도록 하겠다. `custom하는 이유는 함수의 교체가 가능해서 다양한 로직의 조합성을 실현할 수 있기 때문이다.`

```javascript
const reduce = (f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }

  for (const i of iter) {
    acc = f(acc, i);
  }

  return acc;
};

const add = (a, b) => a + b;

// 초기값 지정한 경우의 reduce
console.log(reduce(add, 0, [1, 2, 3, 4, 5]));

// 초기값 지정하지 않은 경우의 reduce
console.log(reduce(add, [1, 2, 3, 4, 5]));

// reduce 응용 예제
// 배열뿐 아니라 객체 배열의 조작도 인자로 넘어가는 f 를 어떻게 구현하느냐에 따라 가능해집니다.
console.log(
  reduce(
    (total_price, products) => (total_price += products.price),
    0,
    products
  )
);
```

reduce를 custom한 예제이다. 마찬가지로 우리가 알고있는 reduce 메서드와는 조금 다른 형태를 가지고 있다.

#### 고차함수 중첩 사용

```javascript
// 테스트 케이스
const products = [
  { name: "반팔티", price: 15000 },
  { name: "긴팔티", price: 20000 },
  { name: "핸드폰케이스", price: 15000 },
  { name: "후드티", price: 30000 },
  { name: "바지", price: 25000 },
];

const add = (a, b) => a + b;

// map, filter, reduce의 중첩 활용
console.log(
  reduce(
    add,
    map(
      (p) => p.price,
      filter((p) => p.price < 20000, products)
    )
  )
);

console.log(
  reduce(
    add,
    filter(
      (n) => n >= 20000,
      map((p) => p.price, products)
    )
  )
);
```
