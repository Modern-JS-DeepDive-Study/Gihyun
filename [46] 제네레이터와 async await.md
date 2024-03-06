# 46. 제네레이터와 async/await

## 1. 제네레이터

<aside>
💡 코드 블록의 실행을 일시 중지했다가 필요한 시점에 재개할 수 있는 특수한 함수

</aside>

1. 제네레이터 함수는 함수 호출자에게 **함수 실행의 제어권을 양도**(yield)할 수 있다.
    1. 일반 함수는 호출 이후 제어권을 함수가 독점한다.
2. 제네레이터 함수는 함수 호출자와 함수의 **상태를 주고받을 수 있다**.
3. 제네레이터 함수를 호출하면 제네레이터 객체를 반환한다.
    1. 제네레이터 함수를 호출하면 일반 함수처럼 함수 코드 블록을 실행하는 것이 아니라 제네레이터 객체를 반환한다.

## 2. 제네레이터 객체

<aside>
💡 제너레이터 함수를 호출하면 이터러블이면서 동시에 이터레이터인 제네레이터 객체를 생성해 반환한다.

</aside>

- Symbol.iterator 메서드를 상속받았다.
- next 메서드를 소유하고 있다.
- 추가적으로 이터레이터에는 없는 `return`, `throw` 메서드를 가지고 있다.

### 제네레이터 객체의 메서드

- `next` 메서드를 호출하면 제네레이터 함수의 `yield` 표현식까지 코드 블록을 실행하고 yield된 값을 value 프로퍼티 값으로, false를 done 프로퍼티 값으로 갖는 `이터레이터 리절트 객체`를 반환한다.
- `return` 메서드를 호출하면 인수로 전달받은 값을 value 프로퍼티 값으로, true를 done 프로퍼티 값으로 갖는 `이터레이터 리절트 객체`를 반환한다.
- `throw` 메서드를 호출하면 인수로 전달받은 에러를 발생시키고 undefined를 value 프로퍼티 값으로, true를 done 프로퍼티 값으로 갖는 `이터레이터 리절트 객체`를 반환한다.

### 제네레이터 함수는 함수 호출자와 함수의 상태를 주고받을 수 있다?

- 이터레이터의 `next` 메서드와 달리 제네레이터 객체의 `next` 메서드에는 인수를 전달할 수 있다.
    - 그 인수는 제네레이터 함수의 `**yield` 표현식을 할당받는 변수에 할당**된다.

→ 이러한 특성을 활용하면 비동기 처리를 동기 처리처럼 구현할 수 있다.

## 그래서 제네레이터 함수는 왜 쓰는데?

### 이터러블의 구현이 간단하다.

```jsx
// 이터레이션 프로토콜을 준수해 이터러블 생성
const infiniteFibonnaci = (function(){
  let [pre, cur] = [0, 1];

  return {
		// 이터러블이면서 이터레이터인 객체 반환
    [Symbol.iterator]() {
      return this;
    },
    next() {
      [pre, cur] = [cur, pre + cur]
      return { value: cur };
    }
  }
}());

// 제너레이터를 사용해 이터러블 생성
const infiniteFibonnaci = (function* (){
  let [pre, cur] = [0, 1];

  while (true) {
    [pre, cur] = [cur, pre+cur];
    yield cur;
  }
}());

for (const num of infiniteFibonnaci) {
  if (num > 10000) break;
  console.log(num);
}
```

- 이터레이션 프로토콜을 준수해 이터러블을 생성하는 방식보다 간단하다.
    - 제네레이터 함수가 반환하는 제네레이터 객체는 이미 이터러블이면서 이터레이터이기 때문이다.
        - Symbol.iterator 메서드를 상속받았다.
        - next 메서드를 소유하고 있다.
    - 즉 프로토콜을 준수하기 위한 코드처럼 보이기보다, `yield` 키워드를 활용해 좀 더 “코드”처럼 보이게 구현할 수 있다.

→ 즉 이터러블의 장점을 똑같이 갖추고 있으면서, 가독성은 좋다.

- 무한 이터러블 구현 가능
- 지연 평가

### 비동기 처리를 동기처럼 구현할 수 있다.

```jsx
const async = (generatorFunc) => {
  const generator = generatorFunc();

  const onResolved = (arg) => {
    const result = generator.next(arg);

    return result.done ? result.value : result.value.then((res) => onResolved(res));
  };

  return onResolved;
};

(async(function* fetchTodo(){
  const url = "https://jsonplaceholder.typicode.com/todos/1";
  const response = yield fetch(url);
  const todo = yield response.json();
  console.log(todo);
})());
```

### 예시: 페이지네이션이 되어있는 API를 호출해 처리하는 과정

1. 지연 평가 활용
- worst

```jsx
async function fetchAllPages(url) {
  let allData = [];
  let nextUrl = url;

  do {
    const response = await fetch(nextUrl);
    const json = await response.json();
    allData.push(...json.data);
    nextUrl = json.next_page_url;
  } while (nextUrl);

  return allData;
}

(async () => {
  const allData = await fetchAllPages('https://api.example.com/data');
  console.log(allData);
})();
```

- good

```jsx
async function* fetchPages(url) {
  let nextUrl = url;
  do {
    const response = await fetch(nextUrl);
    const json = await response.json();
    nextUrl = json.next_page_url;
    yield json.data;
  } while (nextUrl);
}

(async () => {
  for await (const page of fetchPages('https://api.example.com/data')) {
    console.log(page);
  }
})();
```

1. 함수 호출자에게 함수 실행의 제어권을 양도
- 데이터를 받아온 후 후속처리인  `console.log` 는 logAllPages에 종속된다.

```jsx
async function logAllPages(url) {
  let nextUrl = url;

  while (nextUrl) {
    const response = await fetch(nextUrl);
    const json = await response.json();
    console.log(json.data); // 각 요청 후에 데이터를 바로 로깅
    nextUrl = json.next_page_url;
  }
}

(async () => {
  await logAllPages('https://api.example.com/data');
})();
```

- 데이터를 받아온 후 후속처리인  `console.log` 는 자유롭다.

```jsx
async function* fetchPages(url) {
  let nextUrl = url;
  do {
    const response = await fetch(nextUrl);
    const json = await response.json();
    nextUrl = json.next_page_url;
    yield json.data;
  } while (nextUrl);
}

async function logAllPages(url) {
  for await (const page of fetchPages(url)) {
    console.log(page);
  }
}

logAllPages('https://api.example.com/data');
```

- 제너레이터를 쓰고 싶지 않다면?
    - 콜백 패턴을 사용해볼 수 있다.

```jsx
async function fetchAllPages(url, callback) {
  let nextUrl = url;

  while (nextUrl) {
    const response = await fetch(nextUrl);
    const json = await response.json();
    callback(json.data);
    nextUrl = json.next_page_url;
  }
}

async function logAllPages(url) {
  await fetchAllPages(url, (pageData) => {
    console.log(pageData);
  });
}

logAllPages('https://api.example.com/data');
```

## async, await

- 제너레이터보다 간단하고 가독성 좋게 비동기 처리를 동기 처리처럼 동작하도록 도입되었다.
- 프로미스를 기반으로 동작한다.

### async 함수

<aside>
💡 async 함수는 언제나 프로미스를 반환한다. async 함수가 명시적으로 프로미스를 반환하지 않더라도 암묵적으로 반환값을 resolve하는 프로미스를 반환한다.

</aside>

- 인스턴스를 반환해야하는 클래스의 constructor 함수를 제외하고는 모든 함수에 적용이 가능하다.
    - 함수 선언문
    - 함수 표현식
    - 화살표 함수
    - 메서드
    - 클래스 메서드

### await 키워드

<aside>
💡 대기하다가 프로미스가 settled 상태가 되면 프로미스가 **resolve한 처리 결과**를 반환한다.

</aside>

### 에러처리

- 비동기 함수의 콜백 함수를 호출한 것은 비동기 함수가 아니기 때문에 에러를 캐치할 수 없는 문제가 있었다.
- setTimeout과는 달리 프로미스를 반환하는 비동기 함수는 명시적으로 호출할 수 있기 때문에 호출자가 명확하다.

→ 따라서 HTTP 통신에서 발생한 네트워크 에러 뿐 아니라 try 코드 블록 내의 모든 문에서 발생한 에러까지 캐치할 수 있다.

- async 함수 내에서 catch 문을 사용해서 에러 처리를 하지 않으면 async 함수는 발생한 에러를 reject하는 프로미스를 반환한다.