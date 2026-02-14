# React 공식문서 스터디

> 작성일자 : 2026.02.15

- 멋쟁이 사자처럼 14기 FE 운영진 스터디 1주차 내용인 'React 공식문서 스터디'에 관한 내용을 작성했다.
- TIL 작성 시점, [React](https://react.dev/reference/react/hooks) 공식문서의 ver은 v19.2이다.
- 작성한것은 JS 기반의 React 내용이다. (TypeScript가 아님.)
- 공식문서의 reference 탭의 내용중, hooks의 내용만을 공부했다.

## 참고 자료

- [React reference](https://react.dev/reference/react/hooks)
  -> [한글 버전](https://ko.react.dev/reference/react/hooks)

# 1. Hooks

- Hook은 컴포넌트의 최상위 레벨이나, 직접만든 Hook에서만 호출 가능하다. 반복문이나, 조건문 안에서는 호출할 수 없다.

## 1.1 useState

컴포넌트의 최상위 레벨에서 useState를 호출하여 state 변수를 선언한다.

```js
import { useState } from "react";

function MyComponent() {
  const [state, setState] = useState(initialState);
  // [변수명, set변수명]의 형태로 선언.
}
```

- `initialState` : state의 초기 값이다. 어떤 유형의 값이던 지정가능하다.
- StrictMode에서, React는 초기화 함수를 두번 호출한다. 이는 개발 환경 전용 동작이며, 호출 중 하나의 결과는 무시된다.(일반적으로)

- useState가 반환하는 set 함수를 사용하면 state를 다른 값으로 업데이트하고 *리렌더링*을 촉발한다.
- React는 업데이터 함수(set...)를 대기열에 넣고 컴포넌트를 리렌더링 한다. -_ set...함수는 반환값이 없다._

### Point

1. set 함수는 다음 렌더링에 대한 state 변수만 업데이트한다.
   이 상황에서, set 함수를 호출해도 실행 중인 코드의 state는 변경되지 않는다.

```js
function handleClick() {
  console.log(count); // 0

  setCount(count + 1); // 1로 리렌더링 요청합니다.
  console.log(count); // 0

  setTimeout(() => {
    console.log(count); // 0
  }, 5000);
}
```

- state를 업데이트하면 새로운 state 값으로 다른 렌더링을 요청하지만 이미 실행 중인 이벤트 핸들러의 count 변수에는 영향을 미치지 않는다.

```js
const nextCount = count + 1;
setCount(nextCount);

console.log(count); // 0
console.log(nextCount); // 1
```

- 업데이트 된 값을 사용해야 할 때에는, set 함수에 전달하기 전에 변수에 저장해서 사용해야 한다.

2. 이전 state를 기반으로 state 업데이트 하기

- 계산이 끝난 state에 이어서 또다른 계산을 하기 위해서는, 업데이터 함수를 이용해서 계산해야 한다.
- 이와 관련해서는 [스냅샷으로서의 State](https://ko.react.dev/learn/state-as-a-snapshot)를 참고

```js
// 틀린 예시
function handleClick() {
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
}
// 올바른 예시
function handleClick() {
  setAge((a) => a + 1); // setAge(42 => 43)
  setAge((a) => a + 1); // setAge(43 => 44)
  setAge((a) => a + 1); // setAge(44 => 45)
}
```

## 1.2 useEffect

컴포넌트의 최상위 레벨에서 useEffect를 호출하여 Effect를 선언한다.

```js
import { useEffect } from "react";

function MyComponent() {
  useEffect(() => {
    // setup 로직
    return () => {
      // cleanup 로직
    };
  }, [dependencies]);
}
```

- setup : 컴포넌트가 화면에 나타난 후 실행할 작업들을 정의하는 함수다. (API 호출, 구독 시작, 타이머 설정 등)
- cleanup : setup에서 시작한 작업들을 종료하거나 해제하는 함수다. (구독 해제, 타이머 제거, 연결 끊기 등)
- dependencies : setup 함수 안에서 사용하는 props, state, 변수 등을 담는 배열이다. 이 배열에 있는 값이 변경되면 useEffect가 다시 실행된다.
- _useEffect는 undefined를 반환한다._

_dependencies 사용 방식_

- [dep1, dep2] - dep1이나 dep2가 변경될 때만 실행
- [] - 컴포넌트가 처음 화면에 나타날 때만 실행
- 생략 - 컴포넌트가 리렌더링될 때마다 실행

### Point

1. Effect가 컴포넌트 마운트 시 2번 실행된다.

개발 환경에서 Strict Mode가 활성화되면 React는 Effect를 2번 실행한다.

> 실행 순서

- 개발 환경: setup → cleanup → setup
- 배포 환경: setup

React는 실제 설정 이전에 설정과 정리를 한 번 더 실행하여 Effect 로직이 올바르게 구현되었는지 테스트한다.

만약 2번 실행될 때 눈에 띄는 문제가 발생한다면, cleanup 함수에 로직이 누락되었을 가능성이 있다.

2. Effect 내에서 동적 객체 및 함수 이동에 관해서...

- 원문을 읽는것을 추천 : https://react.dev/reference/react/useEffect#my-cleanup-logic-runs-even-though-my-component-didnt-unmount
- 영어가 힘들다면 한국어 버전도 있다 : https://ko.react.dev/learn/removing-effect-dependencies#move-dynamic-objects-and-functions-inside-your-effect

객체나 함수를 Effect 외부에 선언하면 매 렌더링마다 새로 생성되어 불필요한 Effect 재실행이 발생한다.

**문제 상황:**

```js
//  잘못된 예: 객체를 Effect 외부에 선언
function ChatRoom({ roomId }) {
  const options = {
    serverUrl: "https://localhost:1234",
    roomId: roomId,
  };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // options는 매 렌더링마다 새 객체이므로 Effect가 계속 재실행됨!
}
```

- JavaScript에서 객체는 참조로 비교된다.
- 내용이 같아도 매 렌더링마다 새로운 객체가 생성되므로 React는 "변경되었다"고 판단한다.

```js
const obj1 = { name: "John" };
const obj2 = { name: "John" };
console.log(obj1 === obj2); // false (다른 객체!)
```

**해결 방법:**

```js
//  올바른 예: 객체를 Effect 내부로 이동
function ChatRoom({ roomId }) {
  useEffect(() => {
    const options = {
      // Effect 안에서 생성
      serverUrl: "https://localhost:1234",
      roomId: roomId,
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // roomId만 의존성에 추가
}
```

- `options`는 Effect 내부에서만 존재하므로 의존성에 넣을 필요가 없다.
- `roomId`는 문자열이므로 값으로 비교된다 (내용이 같으면 같다고 판단).
- `roomId`가 실제로 변경될 때만 Effect가 재실행된다.

```js
const roomId1 = "music";
const roomId2 = "music";
console.log(roomId1 === roomId2); // true (같은 값!)
```

**함수도 마찬가지:**

```js
// 함수도 Effect 내부에 선언
function ChatRoom({ roomId }) {
  useEffect(() => {
    function createOptions() {
      // Effect 안에서 정의
      return {
        serverUrl: "https://localhost:1234",
        roomId: roomId,
      };
    }
    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // 함수는 의존성에 넣을 필요 없음
}
```

Effect 내부에서 선언한 객체나 함수는 반응형 값이 아니므로 의존성 배열에 포함할 필요가 없다.

## 1.3 useContext

컴포넌트의 최상위 레벨에서 useContext를 호출하여 context를 읽고 구독한다.

```js
import { useContext } from "react";

function MyComponent() {
  const value = useContext(SomeContext);
}
```

- `SomeContext`: createContext로 생성한 context 객체다. 컴포넌트 간에 공유할 데이터의 종류를 정의한다.
- useContext는 해당 context의 현재 값을 반환한다.
- _useContext는 context 값을 반환한다._

useContext는 컴포넌트 트리에서 자신보다 상위에 있는 가장 가까운 Provider의 value를 가져온다. Provider가 없으면 createContext를 만들 때 설정한 기본값을 반환한다.

### 기본 사용 예시

```js
import { createContext, useContext } from "react";

const ThemeContext = createContext("light");

export default function MyApp() {
  return (



  );
}

function Form() {
  return (

      Sign up

  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = "panel-" + theme;
  return (

      {title}
      {children}

  );
}

function Button({ children }) {
  const theme = useContext(ThemeContext);
  const className = "button-" + theme;
  return {children};
}
```

### Point

1. Context 값 업데이트하기

context를 시간이 지남에 따라 변경하려면 state와 결합해야 한다. 부모 컴포넌트에서 state 변수를 선언하고 현재 state를 context 값으로 provider에 전달한다.

```js
function MyPage() {
  const [theme, setTheme] = useState("dark");
  return (


      <Button
        onClick={() => {
          setTheme("light");
        }}
      >
        Switch to light theme


  );
}
```

2. Context provider 간결한 문법 (React 19)

React 19부터는 `<SomeContext.Provider>` 대신 `<SomeContext>`를 provider로 직접 사용할 수 있다.

```js
// React 18 이하

{
  children;
}

// React 19

{
  children;
}
```

3. useContext(SomeContext)는 호출한 컴포넌트에서 반환된 provider의 영향을 받지 않는다.

- 해당 `<Context.Provider>`는 useContext() 호출을 수행하는 컴포넌트보다 위에 있어야 한다.

```js
// 잘못된 예: Provider가 useContext를 호출하는 컴포넌트보다 아래에 있음
function MyComponent() {
  const theme = useContext(ThemeContext); // undefined 또는 기본값
  return (
    <ThemeContext value="dark">
      <Button /> {/_ 이 컴포넌트만 context를 받음 _/}
    </ThemeContext>
  );
}

// 올바른 예: Provider가 useContext를 호출하는 컴포넌트보다 위에 있음
function App() {
  return (
    <ThemeContext value="dark">
      <MyComponent /> {/_ 이 컴포넌트가 context를 받을 수 있음 _/}
    </ThemeContext>
  );
}

function MyComponent() {
  const theme = useContext(ThemeContext); // "dark"
  return <Button />;
}
```

## 1.4 useRef

컴포넌트의 최상위 레벨에서 useRef를 호출하여 ref를 선언한다.

```js
import { useRef } from "react";

function MyComponent() {
  const ref = useRef(initialValue);
}
```

- `initialValue`: ref의 초기값이다. 어떤 유형의 값이든 가능하다. 첫 렌더링 이후에는 이 값은 무시된다.
- useRef는 `{ current: initialValue }` 형태의 객체를 반환한다.
- _useRef는 ref 객체를 반환한다._

다음 렌더링에서도 useRef는 동일한 객체를 반환한다. `ref.current`에 값을 저장하고 나중에 읽을 수 있다.

- 프로퍼티란 객체가 가진 속성을 의미한다. 예를 들어 `{ name: "John", age: 25 }`에서 `name`과 `age`가 프로퍼티다. useRef가 반환하는 객체는 `current`라는 프로퍼티 하나만 가지고 있다.

### 기본 사용 예시

```js
import { useRef } from "react";

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return <>Focus the input</>;
}
```

### Point

1. useRef와 useState의 차이

- useRef: 변경 시 리렌더링을 트리거하지 않는다. 렌더링과 무관한 정보를 저장하는 데 적합하다.
- useState: 변경 시 리렌더링을 트리거한다. 화면에 표시되는 정보를 저장하는 데 적합하다.

2. ref의 특징

- 리렌더링 사이에 정보를 저장할 수 있다 (일반 변수는 렌더링마다 초기화됨).
- 변경해도 리렌더링을 트리거하지 않는다 (state 변수는 리렌더링을 트리거함).
- 각 컴포넌트에 로컬이다 (외부 변수는 공유됨).

3. React 19에서의 변경사항

React 19부터 ref는 일반 prop으로 전달할 수 있으며, forwardRef가 필요 없다.

```js
// React 19
function TextInput({ ref }) {
  return;
}

function App() {
  const inputRef = useRef(null);
  return;
}
```

4. 흔히 하는 실수

- 렌더링 중에 ref.current를 읽거나 쓰지 않는다. 렌더링 중 정보가 필요하면 state를 사용한다.
- ref는 일반 JavaScript 객체이므로 React는 변경 사항을 인지하지 못한다.

## 1.5 useMemo

컴포넌트의 최상위 레벨에서 useMemo를 호출하여 리렌더링 사이에 계산 결과를 캐싱한다. 캐싱은 계산 결과를 메모리에 저장해두었다가 재사용하는 것을 의미한다. 동일한 입력값이면 다시 계산하지 않고 저장된 결과를 반환한다.

```js
import { useMemo } from "react";

function MyComponent() {
  const cachedValue = useMemo(() => calculateValue(a, b), [a, b]);
}
```

- `calculateValue`: 캐싱하려는 값을 계산하는 함수다. 순수 함수여야 하며 인자를 받지 않고 모든 타입의 값을 반환해야 한다.
- `dependencies`: calculateValue 코드 내에서 참조되는 모든 반응형 값의 목록이다.
- useMemo는 초기 렌더링에서 calculateValue를 호출한 결과를 반환한다.
- _useMemo는 캐싱된 값을 반환한다._

다음 렌더링에서 의존성이 변경되지 않았다면 같은 값을 반환한다. 그렇지 않으면 calculateValue를 다시 호출하고 결과를 반환한 뒤 저장한다.

### 기본 사용 예시

```js
import { useMemo } from "react";

function TodoList({ todos, filter }) {
  const visibleTodos = useMemo(
    () => filterTodos(todos, filter),
    [todos, filter],
  );
  // ...
}
```

### Point

1. useMemo 사용 시기

- 기본적으로 React는 컴포넌트가 리렌더링될 때마다 전체 본문을 다시 실행한다.
- 대부분의 계산은 매우 빠르므로 문제가 되지 않는다.
- 하지만 대량의 배열을 필터링/변환하거나 비용이 많이 드는 계산을 하는 경우 데이터가 변경되지 않았다면 다시 계산하는 것을 건너뛸 수 있다.

2. React 19 컴파일러와의 관계

- React 19의 컴파일러는 자동으로 메모이제이션을 처리하여 수동 useMemo 호출의 필요성을 줄인다.
- 하지만 특정 비용이 많이 드는 계산이나 복잡한 데이터 구조에는 여전히 명시적 useMemo가 유용할 수 있다.

3. 흔히 하는 실수

- useMemo는 성능 최적화를 위한 것이지 의미론적 보장이 아니다. 코드가 useMemo 없이도 동작해야 한다.
- 모든 곳에 useMemo를 사용하지 않는다. 작은 계산에는 오히려 오버헤드가 될 수 있다.

4. 의존성 배열 관련 이슈

useMemo 함수 내부에서 사용하는 모든 반응형 값(props, state, 변수)은 의존성 배열에 포함해야 한다. 누락하면 값이 변경되어도 재계산되지 않아 버그가 발생한다.

```js
//  잘못된 예: 의존성 누락
const cachedValue = useMemo(() => {
  return expensiveCalculation(a, b);
}, [a]); // b가 변경되어도 재계산하지 않음

//  올바른 예: 모든 의존성 포함
const cachedValue = useMemo(() => {
  return expensiveCalculation(a, b);
}, [a, b]);
```

## 1.6 useCallback

컴포넌트의 최상위 레벨에서 useCallback을 호출하여 리렌더링 사이에 함수 정의를 캐싱한다.

```js
import { useCallback } from "react";

function MyComponent() {
  const cachedFn = useCallback(fn, [dependencies]);
}
```

- `fn`: 캐싱하려는 함수 값이다. 어떤 인자든 받을 수 있고 어떤 값이든 반환할 수 있다.
- `dependencies`: fn 코드 내에서 참조되는 모든 반응형 값의 목록이다.
- useCallback은 초기 렌더링에서 전달한 함수를 반환한다.
- _useCallback은 캐싱된 함수를 반환한다._

다음 렌더링에서 의존성이 변경되지 않았다면 같은 함수를 반환한다. 그렇지 않으면 현재 렌더링 중에 전달한 함수를 반환한다.

### 기본 사용 예시

```js
import { useCallback } from "react";

function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback(
    (orderDetails) => {
      post("/product/" + productId + "/buy", {
        referrer,
        orderDetails,
      });
    },
    [productId, referrer],
  );
  // ...
}
```

### Point

1. useCallback vs useMemo

- useCallback: 함수 자체를 캐싱한다.
- useMemo: 함수를 호출한 결과를 캐싱한다.

```js
// useCallback(fn, deps)는 다음과 동일하다:
useMemo(() => fn, deps);
```

2. useCallback 사용 시기

자식 컴포넌트에 함수를 prop으로 전달할 때, 그 자식이 React.memo로 감싸져 있다면 useCallback이 유용하다.

```js
function ParentComponent() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    console.log("Clicked!");
  }, []); // 의존성이 없으므로 함수가 절대 재생성되지 않음

  return ;
}

const MemoizedChild = React.memo(({ onClick }) => {
  return Click me;
});
```

3. React 19 컴파일러와의 관계

- React 19의 컴파일러는 자동으로 함수를 메모이제이션하여 수동 useCallback 호출의 필요성을 줄인다.
- 하지만 기존 코드에서 useCallback을 제거할 때는 주의가 필요하다. 코드가 useCallback에 의존하는 경우 제거하면 문제가 발생할 수 있다.

4. 흔히 하는 실수

```js
// 잘못된 예: 불필요한 useCallback
function Component() {
  const handleClick = useCallback(() => {
    console.log("clicked");
  }, []); // 이 함수를 자식에게 전달하지 않는다면 불필요

  return Click;
}

// 올바른 예: 필요한 경우에만 사용
function Component() {
  const handleClick = () => {
    console.log("clicked");
  };

  return Click;
}
```

5. 의존성 배열 관련 이슈

useCallback 함수 내부에서 사용하는 모든 반응형 값은 의존성 배열에 포함해야 한다. 누락하면 함수가 오래된 값(stale value)을 참조하게 된다.

```js
// 잘못된 예: 의존성 누락으로 인한 stale closure
const [count, setCount] = useState(0);
const handleClick = useCallback(() => {
  console.log(count); // 항상 0을 출력
}, []); // count가 의존성에 없음

// 올바른 예: 모든 의존성 포함
const handleClick = useCallback(() => {
  console.log(count);
}, [count]);
```

## 1.7 useReducer

컴포넌트의 최상위 레벨에서 useReducer를 호출하여 reducer로 state를 관리한다.

```js
import { useReducer } from "react";

function MyComponent() {
  const [state, dispatch] = useReducer(reducer, initialState);
}
```

- `reducer`: state가 어떻게 업데이트되는지 지정하는 함수다. 순수 함수여야 하며, state와 action을 인자로 받고 다음 state를 반환해야 한다.
- `initialState`: 초기 state 값이다. 어떤 유형의 값이든 가능하다.
- `init` (선택사항): 초기 state를 반환하는 초기화 함수다. 지정하지 않으면 초기 state가 initialState로 설정되고, 지정하면 init(initialState)의 반환값으로 설정된다.
- useReducer는 현재 state와 dispatch 함수를 가진 배열을 반환한다.
- _useReducer는 배열 [state, dispatch]를 반환한다._

### 기본 사용 예시

```js
import { useReducer } from "react";

function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };
    case "decrement":
      return { count: state.count - 1 };
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: "decrement" })}>-
      <button onClick={() => dispatch({ type: "increment" })}>+
    </>
  );
}
```

### Point

1. useReducer vs useState

**useState를 사용할 때:**

- state 업데이트가 간단할 때
- state가 원시 값이거나 간단한 객체일 때
- state 로직이 여러 이벤트 핸들러에 흩어져 있어도 괜찮을 때

**useReducer를 사용할 때:**

- state 업데이트 로직이 복잡할 때
- 다음 state가 이전 state에 의존할 때
- state가 여러 하위 값을 포함하는 복잡한 객체일 때
- state 로직을 한 곳에 모으고 싶을 때

2. reducer 함수 작성 규칙

- reducer는 순수 함수여야 한다.
- 각 action은 단일 사용자 상호작용을 설명해야 한다.
- state를 직접 변경하지 않고 새로운 객체를 반환한다.

- 순수 함수: 같은 입력에 대해 항상 같은 결과를 반환하고, 외부 변수나 상태를 변경하지 않는 함수

```js
// 잘못된 예: state 직접 변경
function reducer(state, action) {
  state.count++; // 잘못됨!
  return state;
}

// 올바른 예: 새 객체 반환
function reducer(state, action) {
  return { ...state, count: state.count + 1 };
}
```

3. dispatch 함수

- dispatch 함수는 안정적인 식별자를 가지므로 Effect의 의존성에서 생략해도 안전하다.
- StrictMode에서는 reducer와 초기화 함수가 두 번 호출된다 (개발 환경에서만).

4. 흔히 하는 실수

- action type은 reducer의 switch case와 정확히 일치해야 한다. 대소문자가 다르거나 오타가 있으면 action이 실행되지 않는다. 상수로 정의해서 사용하면 오타를 방지할 수 있다.

```js
// 잘못된 예: action type 불일치
dispatch({ type: "INCREMENT" }); // 'increment'가 아닌 'INCREMENT'

// 올바른 예: 일관된 action type 사용
const ACTIONS = {
  INCREMENT: "increment",
  DECREMENT: "decrement",
};

dispatch({ type: ACTIONS.INCREMENT });
```

# 2. StrictMode

StrictMode를 사용하여 컴포넌트 트리에 대한 추가 개발 동작 및 경고를 활성화한다.

```js
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";

const root = createRoot(document.getElementById("root"));
root.render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

### StrictMode가 활성화하는 동작

StrictMode는 다음과 같은 개발 전용 동작을 활성화한다:

1. 렌더링으로 인한 버그를 찾기 위해 컴포넌트를 한 번 더 리렌더링한다.
2. 누락된 Effect cleanup으로 인한 버그를 찾기 위해 Effect를 한 번 더 실행한다.
3. 누락된 ref cleanup으로 인한 버그를 찾기 위해 ref 콜백을 한 번 더 실행한다.
4. 사용 중단된 API 사용을 확인한다.

- _StrictMode는 Props를 받지 않는다_

### Point

1. StrictMode의 이중 호출 동작

**개발 환경에서 호출되는 함수들 (2번 호출됨):**

- 컴포넌트 함수 본문
- useState, useMemo, useReducer에 전달하는 함수
- 클래스 컴포넌트의 constructor, render, shouldComponentUpdate 메서드

```js
function MyComponent() {
  console.log("렌더링됨"); // StrictMode에서 2번 출력됨
  return Hello;
}
```

2. Effect 이중 실행

개발 환경에서 StrictMode는 다음 순서로 실행된다:

- setup → cleanup → setup

이는 cleanup 함수가 제대로 작성되었는지 확인하기 위함이다.

```js
useEffect(() => {
  const connection = createConnection();
  connection.connect();
  return () => {
    connection.disconnect(); // cleanup이 제대로 구현되었는지 테스트
  };
}, []);
```

3. StrictMode는 프로덕션에 영향을 미치지 않는다

- 모든 StrictMode 검사는 개발 모드에서만 실행된다.
- 프로덕션 빌드에는 영향을 주지 않는다.

4. 부분적 적용

StrictMode는 전체 앱이 아닌 일부에만 적용할 수 있다.

5. 사용 중단된 API 경고

StrictMode는 다음과 같은 레거시 API 사용 시 경고한다:

- 안전하지 않은 생명주기 메서드 (componentWillMount 등)
- 레거시 문자열 ref API
- 레거시 context API
- findDOMNode 사용
