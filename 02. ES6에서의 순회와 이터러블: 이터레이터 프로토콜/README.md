# Section 2. ES6에서의 순회와 이터러블: 이터레이터 프로토콜

## Index

0. [기존과 달라진 ES6에서의 리스트 순회](#기존과-달라진-ES6에서의-리스트-순회)
1. [Array,Map,Set을 통한 이터러블/이터레이터 프로토콜](#Array,Map,Set을-통한-이터러블/이터레이터-프로토콜)
2. [사용자 정의 이터러블 및 이터레이터 프로토콜](#사용자-정의-이터러블-및-이터레이터-프로토콜)

---

## Content

### 기존과 달라진 ES6에서의 리스트 순회

자바스크립트가 ES6으로 바뀌게 되면서 리스트 순회쪽에 많은 변화가 있었다. 상세하게 알아볼 필요가 충분히 있다!

```javascript
const list = [1, 2, 3];
for (var i = 0; i < list.length; i++) {
  log(list[i]);
}

const str = "abc";
for (var i = 0; i < str.length; i++) {
  log(str[i]);
}
```

과거에는 이런식으로 리스트를 순회했었다. 물론 지금도 많이 사용할 수도 있는 방법이다.<br>
하지만 ES6이 도입된 시점으로부터 순회를 하는 방법이 조금 달라졌다.

```javascript
for (const a of list) {
  log(a);
}
for (const a of str) {
  log(a);
}
```

코드가 훨씬 간결해진 것을 확인할 수 있을 것이다. `여기서 중요한점은 단순히 간결하게 사용하려고 바꿨다는 것이 아닌 깊은 의미가 있다는 것이다.`

<br>

### Array,Map,Set을 통한 이터러블/이터레이터 프로토콜

JavaScript에는 Array, Set 그리고 Map이라는 내장 객체들을 가지고 있다. 이 3개를 모두 반복문을 통해 순회할 수 있다.

```javascript
const arr = [1, 2, 3];
for (const a of arr) log(a);

const set = new Set([1, 2, 3]);
for (const a of set) log(a);

const map = new Map([
  ["a", 1],
  ["b", 2],
  ["c", 3],
]);
for (const a of map) log(a);
```

참고로, Array에서는 index를 통해 값으로 접근할 수 있다.<br>
`하지만 Map과 Set에서는 index를 통해 값으로 접근할 수 없다.`

`이 의미는 맨 위의 for-of문이 내부적으로 맨 위의 for문(index를 통한 접근)처럼 설계되어 있지 않다는 의미이다.`

#### Symbol.iterator

> Symbol은 ES6에서 추가된 원시 타입 중 하나인데 객체 Property에 대한 식별자로 보통 사용되곤 한다.
>
> 참고로 모든 심볼값은 고유하고 중복이 되지 않으며 한번 생성되면 그 값은 변경되지 않기 때문에 충돌 위험이 없는 타입이다.
>
> 심볼은 new 연산자를 사용한 문법을 지원하지 않기 때문에 생성자 측면에서 불완전한 내장 객체 클래스라고 볼 수 있다.

`Array, Map, Set 모두 Symbol.iterator라는 key값을 암묵적으로 가지고 있다.`

```javascript
const arr = [1, 2, 3];
log(arr[Symbol.iterator]); // 어떤 함수가 들어있음
arr[Symbol.iterator] = null; // 안에 들어있던 함수를 null처리 => 에러 발생
```

위의 코드와 같이 Symbol.iterator를 key로 가지고 있던 함수를 null처리하게 되면 iterable하지 않다고 에러가 발생하게 된다.

**이 의미는 for-of문과 Symbol.iterator가 연관이 있다는 것이다.**

#### 이터러블, 이러레이터 프로토콜

- **이터러블: 이터레이터를 리턴하는 `[Symbol.iterator]()`메서드를 가진 값**

  - Array는 이터러블이다. Symbol.iterator 메서드를 가지고 있는 객체이기 때문이다.
  - 방금 위에서 Symbol.iterator를 null처리하고 실행하면 이터러블이 아니다라고 에러가 발생했음.
    <br>

- **이터레이터: { value, done } 객체를 리턴하는 next() 메서드를 가진 값**

  - `let iterator = arr[Symbol.iterator]()`
  - `iterator.next()` ==> `{ value: 1, done: false }`
    <br>

- 이터러블/이터레이터 프로토콜: 이터러블을 for-of, 전개 연산자 등과 함께 동작하도록 정한 규약을 의미

```javascript
const arr = [1, 2, 3];
let iter1 = arr[Symbol.iterator](); // 이터레이터
iter1.next(); // arr[0]에 접근 { value: 1, done: false }

for (const a of iter1) log(a); // 2 3
```

위의 코드와 같이 이터레이터로 먼저 접근을 하고 for-of문을 사용하게 되면 접근한 data를 제외한 나머지 데이터가 출력된다. (Set과 Map도 마찬가지)

<br>

### 사용자 정의 이터러블/이터레이터 프로토콜 정의

```javascript
const iterable = {
  [Symbol.iterator]() {
    let i = 3;
    return {
      next() {
        return i == 0 ? { done: true } : { value: i--, done: false };
      },
    };
  },
};

let iterator = iterable[Symbol.iterator]();
log(iterator.next()); // { value: 3, done: false }
log(iterator.next()); // { value: 2, done: false }
log(iterator.next()); // { value: 1, done: false }
log(iterator.next()); // { done: true }

for (const a of iterable) log(a);
```

- `iterable 객체 안에 Symbol.iterator가 구현되어 있기 때문에 for-of문에 사용될 수 있는 것이다.`
- `그리고 내부적으로 next 메서드를 실행하면서 a에 value를 담아주고 있다.`

하지만 이터러블/이터레이터 프로토콜의 모든 속성을 아직 구현하지 못했다.

```javascript
const arr2 = [1, 2, 3];
let iter2 = arr2[Symbol.iterator](); // 이터레이터
log(iter2[Symbol.iterator]() == iter2); // true (자기자신과 동일)
for (const a of iter2) log(a);
```

**iter2도 Symbol.iterator를 가지고 있고 Symbol.iterator를 실행시키면 자기 자신을 반환하게 된다.<br>
이것을 Well-formed 이터러블/이터레이터라고 한다.**

```javascript
const iterable = {
  [Symbol.iterator]() {
    let i = 3;
    return {
      next() {
        return i == 0 ? { done: true } : { value: i--, done: false };
      },
      [Symbol.iterator]() {
        return this;
      },
    };
  },
};
```

`위 코드와 같이 Symbol.iterator를 다시 또 실행시키게 되면 자기 자신을 반환시키도록 코드를 추가했습니다.`

`이렇게 자기 자신을 반환하도록 해서 이전까지 진행되었던 상태를 기억하고 남은 값을 소비할 수 있게 하기 위해서 Well-formed 이터러블/이터레이터를 구현한다고 생각하면 될 것 같다.`

```javascript
const iterable = {
  [Symbol.iterator]() {
    let i = 3;
    return {
      next() {
        return i == 0 ? { done: true } : { value: i--, done: false };
      },
    };
  },
};

let iterator = iterable[Symbol.iterator]();
for (const a of iterator) log(a);
```

만약 위와 같이 Well-formed 이터러블/이터레이터가 아닌 경우에는 for-of에서 에러가 반환될 것이다. `iterator가 iterable하지 않기 때문이다.`

`따라서 Well-formed 상태에서는 iterator든 iterable이던 for-of에 접근해서 사용될 수 있다.`

---

### Reference

Symbol 자료형에 대한 자세한 설명이 궁금하시면 아래 링크로 확인해주세용!

- https://moon-ga.github.io/javascript/symbol/
