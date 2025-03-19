# 1. useEffect에서 setTimeout 문제

- React의 useEffect에서 setTimeout을 사용할 때 흔히 발생하는 문제는 this가 아니라, 클로저 문제로 인해 오래된 상태를 참조하는 것입니다.

## 1-1. setTimeout이 초기 상태를 참조하는 문제

```js
import { useState, useEffect } from "react";

function MyComponent() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    setTimeout(() => {
      console.log(count); // ❌ 항상 0을 출력 (최신 상태를 참조하지 않음)
    }, 1000);
  }, []);

  return <button onClick={() => setCount(count + 1)}>Increase</button>;
}
```

**🔍 원인**

- useEffect의 의존성 배열이 []이므로, 첫 렌더링 시점의 count 값(즉, 0)을 클로저로 캡처합니다.
- 이후 setCount를 사용하여 상태를 변경해도, setTimeout 내에서 여전히 초기 값인 0만 출력됩니다.

**✅ 해결 방법**

1. 의존성 배열에 count 추가하기: count가 변경될 때마다 useEffect가 재실행되므로, 최신 값을 사용할 수 있습니다.
   ```js
   useEffect(() => {
     setTimeout(() => {
       console.log(count); // 최신 count 값이 반영됨
     }, 1000);
   }, [count]);
   ```
2. 함수형 업데이트 (prevState 사용): setCount(prevCount => {...})를 사용하면, 항상 최신 상태를 참조할 수 있습니다.
   ```js
   useEffect(() => {
     setTimeout(() => {
       setCount((prevCount) => {
         console.log(prevCount); // 최신 count 값을 반영
         return prevCount;
       });
     }, 1000);
   }, []);
   ```

---

## 1-2. 기술 면접 정리

### 1. useEffect에서 setTimeout을 사용할 때 클로저 문제(Old State Reference)란?

**질문**

- useEffect에서 setTimeout을 사용할 때, 왜 이전 상태 값이 출력되는 경우가 있나요?
  <br/>

**답변**

- useEffect의 의존성 배열을 []로 설정하면, 컴포넌트가 처음 렌더링될 때만 실행되므로 setTimeout 내부에서 클로저로 초기 상태 값을 캡처하게 됩니다.
- 즉, 이후 상태가 변경되어도 setTimeout 내부에서는 여전히 초기 상태 값을 참조하기 때문에, 오래된 상태가 출력될 수 있습니다.

### 2. useEffect에서 setTimeout이 언마운트된 후에도 실행되는 문제

**질문**

- useEffect에서 setTimeout을 사용할 때, 컴포넌트가 언마운트되면 어떤 문제가 발생할 수 있나요? 이를 어떻게 해결할 수 있을까요?
  <br/>

**답변**

- 컴포넌트가 언마운트되었는데도 setTimeout이 실행되면 불필요한 상태 업데이트가 발생할 수 있음.
  React 18+에서는 Strict Mode에서 개발 환경에서 컴포넌트가 두 번 마운트되는 문제가 발생할 수 있어 setTimeout의 실행이 의도치 않게 여러 번 발생할 수도 있음.
  이를 방지하려면 useEffect의 cleanup function(정리 함수)에서 clearTimeout()을 호출해야 함

```js
useEffect(() => {
  const timeoutId = setTimeout(() => {
    setCount(count + 1);
  }, 1000);

  return () => clearTimeout(timeoutId); // ✅ 정리 함수에서 타이머 제거
}, [count]);
```

### 3. setTimeout과 React Strict Mode의 관계

**질문**

- React Strict Mode가 setTimeout과 관련된 문제를 유발할 수 있나요?
  <br/>

**답변**

- React 18의 Strict Mode에서는 개발 환경에서 컴포넌트가 두 번 마운트됨.
  따라서 useEffect 내부에서 setTimeout을 설정하면, 의도하지 않게 두 개의 타이머가 설정될 수 있음.
  이를 해결하려면 Strict Mode 환경에서도 정상적으로 동작하도록 정리(cleanup) 함수를 올바르게 설정해야 함.

```js
import { useEffect, useRef } from "react";

function MyComponent() {
  const hasRun = useRef(false); // ✅ useRef를 활용하여 최초 실행만 허용

  useEffect(() => {
    if (!hasRun.current) {
      setTimeout(() => {
        console.log("Hello, world!");
      }, 1000);
      hasRun.current = true; // 두 번째 실행 방지
    }
  }, []);
}
```

### 4. 자바스크립트에서 정확한 지연 기능(Precise Delay)을 제공하지 않는 이유

- 자바스크립트는 **싱글 스레드 기반 비동기 언어**로 이벤트 루프 모델을 사용합니다. 즉, 한 번에 하나의 작업만 처리 가능하며, 특정 실행 중에는 다른 코드 실행이 블로킹될 수 있습니다.
- setTimeout이나 setInterval은 정확한 타이밍 보장을 하지 않음. 이벤트 루프가 현재 실행 중인 코드(콜 스택)처리를 먼저 끝내야 실행된다.
  즉, 정확한 시간이 되면 실행되는 것이 아니라, 지정된 시간이 지난 후 태스크 큐에 추가된다. 실행되려면 콜 스텍이 비어있어야 한다.

**setTimeout(0)이 즉시 실행되지 않는 이유는?**

- setTimeout(0)을 사용하면 0ms 후 실행될 것처럼 보이지만, 즉시 실행되지 않고 이벤트 루프의 태스크 큐에 추가됨. 따라서 콜 스택이 비워진 후 실행됨.

**해결책**

- Node.js에서는 setTimeout보다 정확한 타이머를 제공하는 방법으로 process.hrtime()을 사용할 수 있습니다.

```js
const start = process.hrtime();

setTimeout(() => {
  const diff = process.hrtime(start);
  console.log(`Actual delay: ${diff[0]}s ${diff[1] / 1e6}ms`);
}, 1000);
```

---

# 1-3. 실행 컨텍스트(Execution Context)

1. 실행 컨텍스트의 생성 과정에서 변수는 어떻게 처리되나요?

**답변:**

1. 호이스팅(Hoisting) 발생: var 변수는 undefined로 초기화됨. let, const 변수는 TDZ(Temporal Dead Zone)에 놓임.
2. 변수 환경과 렉시컬 환경에 저장: var 변수는 변수 환경에 저장. let, const 변수는 렉시컬 환경에 저장.

2) 실행 컨텍스트가 여러 개 쌓일 때, 콜 스택(Call Stack)은 어떻게 관리되나요?
   **답변:**

- 자바스크립트는 단일 스레드(Single Thread) 기반이며, **콜 스택(Call Stack)**을 이용해 실행 컨텍스트를 관리합니다.
- **함수가 호출될 때마다 실행 컨텍스트가 콜 스택에 추가(Push)**되고, **함수 실행이 끝나면 해당 실행 컨텍스트가 제거(Pop)**됩니다.

3. 실행 컨텍스트에서 this 바인딩은 어떻게 결정되나요?
   **답변:**

- this는 실행 컨텍스트가 생성될 때 결정되며, 함수 호출 방식에 따라 값이 다릅니다.

| 호출 방식                   | this 값                              |
| --------------------------- | ------------------------------------ |
| 일반 함수 호출              | window (strict mode에서는 undefined) |
| 메서드 호출                 | 해당 메서드를 호출한 객체            |
| 생성자 함수 호출 (new 사용) | 새롭게 생성된 인스턴스 객체          |
| call, apply, bind 사용      | 명시적으로 지정한 객체               |
| 화살표 함수                 | 상위 스코프에서 this를 계승          |

4. var, let, const가 실행 컨텍스트에서 어떻게 다르게 동작하나요?
   **답변:**

- 변수 선언 방식에 따라 실행 컨텍스트에서의 변수 저장 방식과 스코프(유효 범위)가 달라집니다.

| 선언 방식 | 저장 위치                        | 호이스팅 여부               | TDZ(Temporal Dead Zone) |
| --------- | -------------------------------- | --------------------------- | ----------------------- |
| var       | 변수 환경(Variable Environment)  | 호이스팅 (초기값 undefined) | 없음                    |
| let       | 렉시컬 환경(Lexical Environment) | 호이스팅 (TDZ 있음)         | TDZ 존재                |
| const     | 렉시컬 환경(Lexical Environment) | 호이스팅 (TDZ 있음)         | ✅ TDZ 존재             |

- `let`은 `TDZ(Temporal Dead Zone, 일시적 사각지대)`에 위치하여 초기화 전에 접근 불가능.
- `var`는 변수 환경에 `undefined`로 초기화되어 접근 가능.
