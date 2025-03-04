## Q1. const 객체 내부 값을 변경할 수 있는 이유는?

```js
const obj = { name: "Alice" };
obj.name = "Bob";
console.log(obj.name); // "Bob"
```

## A1.

- const는 "변수의 값 자체를 변경할 수 없는 불변성"을 지닌 것이 아니라 "변수에 할당된 메모리 주소를 변경할 수 없는 것"을 의미합니다.
- 객체나 배열 같은 참조 타입의 경우, 변수 자체가 아니라 객체가 위치한 메모리 주소를 가리키고 있으므로 내부 속성 변경이 가능합니다.

> 기본 타입(Primitive Type): string, number, boolean, null, undefined, symbol, bigint는 값 자체를 메모리에 저장
> 참조 타입(Reference Type): object, array, function은 메모리 주소(참조값)를 저장

- 즉, const는 객체의 변경을 막아주는 것이 아니라, 변수가 가리키는 메모리 주소(참조값)의 변경을 막아주는 것이라고 이해하자.

## Q2. ES6 모듈 스코프(Module Scope)란?

## A2.

- 전역 변수를 남용하면 스코프 오염, 변수 충돌, 예측 불가능한 동작 등의 문제가 발생할 수 있습니다.
- ES6에서는 import / export 를 사용한 모듈 시스템(Module System) 을 통해 전역 변수 없이 코드를 모듈화하고 변수 관리를 효율적으로 할 수 있습니다.
  동적 import()를 활용하여 필요할 때만 모듈을 로드 가능합니다.

## Q3. 고차 함수를 활용한 메모이제이션(Memoization)이란?

## A3.

- 메모이제이션은 한 번 계산된 값을 캐싱하여, 동일한 입력에 대해 다시 계산하지 않도록 하는 기법입니다.
- 고차 함수는 캐시를 유지하는 클로저를 활용하여 메모이제이션을 구현할 수 있습니다.

```js
function memoizedAdd() {
  let cache = {};
  return function (num) {
    if (num in cache) {
      console.log("캐시에서 가져옴!");
      return cache[num];
    }
    console.log("새로 계산!");
    cache[num] = num + 10;
    return cache[num];
  };
}

const add10 = memoizedAdd();
console.log(add10(5)); // 새로 계산! -> 15
console.log(add10(5)); // 캐시에서 가져옴! -> 15
console.log(add10(10)); // "새로 계산!" -> 20
console.log(add10(10)); // "캐시에서 가져옴!" -> 20
```

<br/>
=> 상세 설명:
	1.	memoizedAdd()를 호출하면, cache = {} 객체가 생성됨 <br/>
	•	cache는 클로저로 내부 함수에서 계속 유지됨 <br/>
    
	2.	add10(5)를 처음 실행할 때: <br/>
	•	5가 cache에 없으므로 "새로 계산!" 출력 <br/>
	•	5 + 10 = 15를 계산하고 cache[5] = 15 저장 <br/>
	•	15를 반환 <br/>

    3.	add10(5)를 다시 실행할 때: <br/>
    •	5가 이미 cache에 있으므로 "캐시에서 가져옴!" 출력 <br/>
    •	저장된 cache[5] = 15 값을 반환 (재계산 없음!) <br/>

    4.	add10(10)을 실행하면: <br/>
    •	10이 cache에 없으므로 "새로 계산!" 출력 <br/>
    •	cache[10] = 20 저장 후 20 반환 <br/>

    5.	add10(10)을 다시 실행하면: <br/>
    •	cache[10] = 20이 있으므로 "캐시에서 가져옴!" 출력 <br/>
    •	저장된 값 20 반환 <br/>

> 클로저는 고차 함수가 내부 변수를 은닉할 수 있도록 하는 기법입니다.
> 고차 함수가 함수를 반환할 때, 반환된 함수는 부모 스코프의 변수에 접근할 수 있습니다.
>
> ```js
> function multiplier(factor) {
>   return function (number) {
>     return number * factor;
>   };
> }
>
> const double = multiplier(2);
> console.log(double(5)); // 10 => double(5) 실행 시, factor = 2 값을 유지 (클로저 활용)
> ```

### +) 실제 활용 예제: Fibonacci 수열 최적화

1. 기본 재귀 방식 (비효율적)

```js
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

console.log(fibonacci(40)); // 매우 느림
```

2. 메모이제이션을 적용한 방식 (최적화됨)

- 같은 n 값이 들어오면 이미 계산된 결과를 재사용하여 성능이 대폭 향상된다. (같은 입력이 들어왔을 때 다시 계산하지 않고 기존 결과를 반환하는 방식)

```js
function memoizedFibonacci() {
  let cache = {};

  return function fib(n) {
    if (n in cache) return cache[n]; // 캐시 확인
    if (n <= 1) return n;

    cache[n] = fib(n - 1) + fib(n - 2); // 계산 후 캐시 저장
    return cache[n];
  };
}

const fib = memoizedFibonacci();
console.log(fib(40)); // 훨씬 빠름 (캐싱 활용)
```
