# Section 3. 제너레이터와 이터레이터

## Index

0. [기존과 달라진 ES6에서의 리스트 순회](#기존과-달라진-ES6에서의-리스트-순회)
1. [제너레이터를 활용한 odds 메서드 예제](#제너레이터를-활용한-odds-메서드-예제)
2. [iterable 함수들을 조합하는 방법들](#iterable-함수들을-조합하는-방법들)

---

## Content

### 제너레이터와 이터레이터

- `제너레이터`: 이터레이터이자 이터러블을 생성하는 함수 (이터레이터를 Return하는 함수)

```javascript
function* gen() {
  yield 1;
  yield 2;
  yield 3;
  return 100;
}
let iter = gen(); // 이터레이터 반환
log(iter.next()); // { value: 1, done: false }
log(iter.next()); // { value: 2, done: false }
log(iter.next()); // { value: 3, done: false }
log(iter.next()); // { value: 100, done: true } -> return값을 지정안하면 value는 undefined

for (const a of gen()) log(a);
```

> 제너레이터로 반환된 이터레이터는 이터러블이기도 하다! <br>
> 즉, `제너레이터는 Well-formed 이터러블/이터레이터`이다.

그렇기 때문에 for-of문으로 접근이 가능한 것이다.

추가적으로 제너레이터에서 return 값을 지정해줄 수 있다.<br>
return값을 지정해주면 yield로 선언한 값을 넘어가면 value가 `undefined가 아닌 return으로 지정해 준 값으로 들어간다.`

하지만 return 값은 for-of문을 순회할 때 접근할 수 없다. 위 예제에서 `return 100` 코드를 작성했어도 for-of문의 결과값으로는 `1, 2, 3`이 출력될 것이다.

<br>

```javascript
function* gen() {
  yield 1;
  if (false) yield 2;
  yield 3;
  return 100;
}
let iter = gen(); // 이터레이터 반환
log(iter.next()); // { value: 1, done: false }
log(iter.next()); // { value: 2, done: false }
log(iter.next()); // { value: 3, done: false }
log(iter.next()); // { value: 100, done: true } -> return값을 지정안하면 value는 undefined

for (const a of gen()) log(a);
```

그리고 `제너레이터는 위와 같이 순회할 값을 문장으로 표현하는 것`이라고도 말할 수 있다.

> 자바스크립트에서는 어떠한 값이든 이터러블이면 순회할 수 있다. 하지만 제너레이터는 이러한 문장을 순회할 수 있는 값으로 만들 수 있다. <br>
>
> `때문에, 제너레이터를 통해서 어떠한 값이나 상태든 순회할 수 있는 상태(이터러블)로 만들 수 있다는 것이다.`
>
> 이것은 상징적이고 함수형 프로그래밍 관점에서 많이 중요한 개념이다!

<br>

### 제너레이터를 활용한 odds 메서드 예제

```javascript
function* odds(num) {
  for (let i = 1; i <= num; i++) {
    if (i % 2) {
      yield i;
    }
  }
  return;
}
```

위의 예제는 제너레이터를 이용해 홀수만을 출력하는 함수를 구현한 것이다. 여기서 이터러블 함수들을 활용해서 관심사 분리를 통해 함수로 분리할 수 있다.

```javascript
function* infinitiy(i) {
  while (true) yield i++;
}

function* odds(l) {
  for (const a of infinity(l)) {
    if (a % 2) yield a;
  }
}
```

이렇게 무한루프를 함수로 기능을 분리한 예제이다. 여기서 정말 신기한점은 while 무한 반복문은 런타임 에러가 발생할 가능성이 매우 높은데 전혀 발생하지 않는다.

`이유는 함수가 실행될 때만 yield를 생성하기 때문에 런타임 에러가 발생하지 않는다.`

위의 소스에서 limit 범위를 제한하는 기능을 함수로 또 분리를 해 보자면..

```javascript
function* infi(x = 0) {
  while (true) yield x++;
}

function* limit(l, iter) {
  for (const a of iter) {
    yield a;
    if (a === l) return;
  }
}

function* odds(num) {
  for (let i of limit(num, infi(1))) {
    if (i % 2) {
      yield i;
    }
  }
  return;
}
```

이런 작업들을 해보면서 `가장 중요한 점은 관심사를 분리시키는 것`이라고 느꼈다.<br>
그리고 `TDD와도 어느정도 연관성`이 있겠다고도 싶었다.

<br>

### iterable 함수들을 조합하는 방법들

> for-of, 전개연산자, 구조분해, 나머지 연산자

```javascript
log(...odds(10));
log([...odds(10), ...odds(20)]);

// 구조분해 + 나머지 연산자
const [head, ...tail] = odds(5);
const [a, b, ...rest] = odds(5);
```
