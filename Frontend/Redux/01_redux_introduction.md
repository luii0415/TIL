# Redux 공식문서 스터디 - TIL 1

> 작성일자 : 2026.02.19 ~ 2026.02.22

- 멋쟁이 사자처럼 14기 FE 운영진 스터디 2주차 내용인 'Redux 공식문서 스터디'에 관한 내용을 작성했다.
- TIL 작성 시점, [Redux](https://redux.js.org/introduction/getting-started) 공식문서의 ver은 **v5.0.1**이다.
- 작성한 것은 JS 기반의 Redux 내용이다. (TypeScript가 아님.)
- 공식문서의 Introduction 및 Fundamentals(Core Concepts) 내용을 공부했다.
- **Redux Toolkit(RTK) 관련 내용은 TIL 2에서 별도로 다룬다.**

## 참고 자료

- [Redux 공식문서 - Getting Started](https://redux.js.org/introduction/getting-started)
  - [Core Concepts](https://redux.js.org/introduction/core-concepts)
  - [Redux Fundamentals - Part 2: Concepts and Data Flow](https://redux.js.org/tutorials/fundamentals/part-2-concepts-data-flow)
  - [API Reference - Store](https://redux.js.org/api/store)

---

# 1. Redux란?

## 1.1 Redux를 왜 쓰는가?

React를 공부하다 보면 `useState`, `useContext`로 상태를 관리한다. 그런데 앱이 점점 커지면 이런 상황이 생긴다.

```
컴포넌트 A (state 보유)
  └── 컴포넌트 B
        └── 컴포넌트 C
              └── 컴포넌트 D  ← 여기서 A의 state가 필요함
```

이 경우 A의 state를 D까지 props로 계속 내려줘야 한다. 이걸 **Props Drilling**이라고 한다.
`useContext`로 어느 정도 해결할 수 있지만, 앱 규모가 커지고 **여러 컴포넌트가 동시에 같은 state를 읽고 변경해야 하는 상황**에서는 관리가 점점 어려워진다.

Redux는 이 문제를 해결하기 위해 **state를 컴포넌트 밖의 중앙 저장소(Store)에서 관리**한다.

```
         [ Redux Store ]  ← 전역 state 한 곳에 저장
         ↑            ↓
컴포넌트 A    컴포넌트 D    컴포넌트 Z
```

어떤 컴포넌트든 Store에서 직접 state를 읽고 업데이트할 수 있게 된다.

### Redux를 사용하면 좋은 상황

공식문서에서 다음 상황일 때 Redux를 고려하라고 안내한다.

- 시간이 지남에 따라 계속 변하는 데이터가 상당량 있을 때
- 앱 전체에서 단일 state 출처(Single Source of Truth)가 필요할 때
- 최상위 컴포넌트에서 모든 state를 관리하는 것이 더 이상 충분하지 않을 때

> **주의:** "남들이 쓰니까"라는 이유만으로 Redux를 쓰지 않는 것을 공식문서에서도 강조한다. 작은 앱에는 오히려 불필요한 복잡도를 추가한다.

---

# 2. Redux의 핵심 개념 3가지

Redux의 모든 동작은 **State(상태)**, **Action(행동)**, **Reducer(처리기)** 이 세 가지를 중심으로 돌아간다.

## 2.1 State (상태)

Redux에서 앱 전체의 상태는 **하나의 JavaScript 객체**로 표현된다.

```js
// Todo 앱의 전체 state 예시
{
  todos: [
    { text: "Redux 공부하기", completed: false },
    { text: "밥 먹기", completed: true }
  ],
  visibilityFilter: "SHOW_ALL"
}
```

- 이 객체는 **직접 수정할 수 없다.** (Immutability, 불변성)
- 변경하려면 반드시 Action을 dispatch해야 한다.

### 불변성(Immutability)이란?

JavaScript의 객체와 배열은 기본적으로 **변경 가능(mutable)**하다.

```js
const obj = { a: 1, b: 2 };
obj.b = 3; // 직접 수정 가능 → Redux에서는 이렇게 하면 안 됨!
```

Redux에서는 state를 직접 수정하지 않고, **기존 state를 복사한 뒤 새 값으로 변경한 새 객체를 반환**해야 한다.

```js
// 잘못된 예: state 직접 변경
state.b = 3;

// 올바른 예: 복사본을 만들어서 수정
const newState = { ...state, b: 3 };
```

Redux가 불변성을 요구하는 이유는 **이전 state와 새 state를 비교해서 변경을 감지**하기 때문이다. 직접 수정하면 참조가 같아서 변경을 인식하지 못한다.

## 2.2 Action (행동)

Action은 **"어떤 일이 일어났는지"를 설명하는 평범한 JavaScript 객체**다.

```js
// Action 예시들
{ type: "todos/todoAdded", payload: "Redux 공부하기" }
{ type: "todos/todoToggled", payload: 1 }
{ type: "filter/visibilityChanged", payload: "SHOW_COMPLETED" }
```

- `type` 필드는 **필수**다. 어떤 종류의 action인지 설명하는 문자열이다.
- `type` 문자열은 `"도메인/이벤트명"` 형태로 작성하는 것이 관례다.
- 추가 데이터는 `payload` 필드에 담는 것이 관례다.
- _Action 객체는 반드시 `type` 필드를 가져야 한다._

### Action Creator (액션 생성자)

Action 객체를 매번 직접 작성하면 번거롭고 오타가 생길 수 있다. 그래서 Action 객체를 반환하는 함수를 따로 만들어 쓰는 경우가 많다.

```js
// Action Creator
function addTodo(text) {
  return {
    type: "todos/todoAdded",
    payload: text,
  };
}

// 사용
store.dispatch(addTodo("Redux 공부하기"));
// 위는 아래와 동일하다
store.dispatch({ type: "todos/todoAdded", payload: "Redux 공부하기" });
```

## 2.3 Reducer (리듀서)

Reducer는 **현재 state와 action을 받아서, 새로운 state를 반환하는 순수 함수**다.

```
(currentState, action) => newState
```

이름이 "Reducer"인 이유는 JavaScript의 `Array.reduce()` 메서드의 콜백 함수와 개념이 같기 때문이다. 이전 결과(state)와 현재 항목(action)을 받아서 새로운 결과(newState)를 반환한다.

```js
const initialState = { value: 0 };

function counterReducer(state = initialState, action) {
  // action의 type을 보고 어떻게 state를 업데이트할지 결정한다
  if (action.type === "counter/incremented") {
    return {
      ...state, // 기존 state를 복사하고
      value: state.value + 1, // value만 변경한 새 객체를 반환
    };
  }
  // 해당 action에 관심 없으면 현재 state를 그대로 반환
  return state;
}
```

### Reducer 작성 규칙 (반드시 지켜야 함)

1. `state`와 `action` 인자만을 기반으로 새 state를 계산해야 한다.
2. 기존 `state`를 직접 수정하면 안 된다. **반드시 불변(immutable) 업데이트를 해야 한다.**
3. 비동기 로직, 랜덤 값 계산, 사이드 이펙트(외부 API 호출 등)를 수행하면 안 된다.

즉, **같은 입력에는 항상 같은 결과를 반환하는 순수 함수(Pure Function)**여야 한다.

```js
// 잘못된 예: state 직접 변경
function reducer(state = initialState, action) {
  if (action.type === "counter/incremented") {
    state.value += 1; // 직접 수정 금지!
    return state;
  }
  return state;
}

// 올바른 예: 새 객체 반환
function reducer(state = initialState, action) {
  if (action.type === "counter/incremented") {
    return { ...state, value: state.value + 1 }; // 복사 후 새 값 반환
  }
  return state;
}
```

---

# 3. Store (스토어)

Store는 Redux의 **중앙 저장소**다. 앱 전체의 state가 여기에 저장된다.

> **중요:** Redux 앱에는 **단 하나의 Store**만 존재해야 한다. (Single Source of Truth 원칙)

## 3.1 Store 생성

> ** 주의: `createStore`는 v4.2.0부터 deprecated(사용 중단 권고) 처리되었다.**
> 현재는 Redux Toolkit의 `configureStore`를 사용하는 것이 공식 권장 방식이다.
> 아래 내용은 Redux의 동작 원리를 이해하기 위한 학습 목적으로 다루는 레거시 방식이다.
> 실제 프로젝트에서는 Redux Toolkit의 `configureStore`를 사용할 것.

```js
import { legacy_createStore as createStore } from "redux";
// deprecated 표시를 피하려면 legacy_createStore로 import한다

const store = createStore(counterReducer);
```

## 3.2 Store의 메서드

Store는 세 가지 메서드를 제공한다.

### `getState()`

현재 state를 반환한다.

```js
console.log(store.getState()); // { value: 0 }
```

### `dispatch(action)`

action을 store에 전달해서 state를 변경하는 **유일한 방법**이다.
dispatch를 호출하면 store는 reducer를 실행하고 새 state를 저장한다.

```js
store.dispatch({ type: "counter/incremented" });
console.log(store.getState()); // { value: 1 }

store.dispatch({ type: "counter/incremented" });
console.log(store.getState()); // { value: 2 }
```

### `subscribe(listener)`

store의 state가 변경될 때마다 실행할 콜백 함수를 등록한다.
반환값은 **구독 해제 함수**다.

```js
const unsubscribe = store.subscribe(() => {
  console.log("state가 변경됨:", store.getState());
});

store.dispatch({ type: "counter/incremented" }); // "state가 변경됨: { value: 1 }" 출력

// 구독 해제
unsubscribe();

store.dispatch({ type: "counter/incremented" }); // 이제 콜백이 실행되지 않음
```

### Point

- `dispatch`는 reducer를 통해 새 state를 계산한 뒤, 등록된 모든 listener를 호출한다.
- `subscribe`의 listener는 state가 실제로 변경되었는지와 관계없이, dispatch가 호출될 때마다 실행된다.
- React와 함께 쓸 때는 `subscribe`를 직접 사용하는 경우는 거의 없다. 대신 React-Redux 라이브러리의 `useSelector`를 사용한다. (TIL 2에서 다룸)

---

# 4. Redux 데이터 흐름 (단방향)

Redux의 가장 중요한 특징 중 하나는 **데이터가 단방향으로만 흐른다**는 것이다.

```
[UI에서 이벤트 발생]
        ↓
[dispatch(action)]   ← "할 일 추가" 버튼 클릭 등
        ↓
[Store가 Reducer 실행]  ← (currentState, action) => newState
        ↓
[새 State 저장]
        ↓
[UI가 새 State로 리렌더링]
```

구체적인 순서를 정리하면 이렇다.

1. **UI에서 이벤트 발생** (버튼 클릭, 입력 등)
2. `store.dispatch(action)` 호출
3. Store가 현재 state와 action을 Reducer에 전달
4. Reducer가 새로운 state를 계산해서 반환
5. Store가 새 state를 저장하고, 등록된 listener들을 호출
6. UI가 새 state를 읽어 화면을 업데이트

### Point

이 흐름이 단방향이기 때문에 **state가 어떻게 변경되었는지 항상 추적 가능**하다. 어떤 action이 dispatch되었는지, 그 결과 state가 어떻게 바뀌었는지 Redux DevTools로 확인할 수 있다. 이것이 Redux의 핵심 장점인 **예측 가능성(Predictability)**이다.

---

# 5. Selector (셀렉터)

Selector는 **store의 state에서 필요한 데이터를 추출하는 함수**다.

```js
// Selector 예시
const selectCounterValue = (state) => state.counter.value;
const selectTodos = (state) => state.todos;
const selectCompletedTodos = (state) =>
  state.todos.filter((todo) => todo.completed);

// 사용
const currentValue = selectCounterValue(store.getState());
```

앱이 커지면 여러 컴포넌트에서 동일한 state 데이터를 참조하게 된다. Selector를 사용하면 **데이터를 추출하는 로직을 한 곳에서 관리**할 수 있어 중복을 줄일 수 있다.

---

# 6. Redux 실제 사용법

> **보일러플레이트란?** 매번 새 프로젝트를 시작할 때 반복적으로 작성해야 하는 기본 설정 코드의 묶음을 말한다.
> ... Redux는 보일러 플레이트 세팅이 상당히 복잡하다고 한..다...(실제로도 그런것 같다)

## 6.1 폴더 구조

공식문서(FAQ: Code Structure, Style Guide)는 **"Feature Folder"** 방식을 공식 권장한다.

기능(feature)별로 폴더를 나누고, 그 안에 해당 기능의 Redux 로직을 하나의 파일에 모아두는 방식이다.
이 파일 하나에 action, reducer, selector를 모두 담는 패턴을 **"Ducks 패턴"** 이라고 부른다.

```
src/
├── index.js              # 앱 진입점 (Entry Point)
├── App.js                # 루트 컴포넌트
│
├── app/                  # 앱 전체에서 공유되는 Redux 설정
│   ├── store.js          # Store 생성 및 설정
│   └── rootReducer.js    # 모든 reducer를 하나로 합치는 파일
│
├── common/               # 공통으로 쓰이는 컴포넌트, 훅, 유틸 등
│
└── features/             # 기능(feature)별 폴더
    ├── todos/            # "할 일" 기능 관련 파일 모음
    │   ├── todosSlice.js # todos의 action + reducer + selector를 한 파일에
    │   └── TodoList.js   # todos 관련 React 컴포넌트
    │
    └── filters/          # "필터" 기능 관련 파일 모음
        ├── filtersSlice.js
        └── Filters.js
```

### 왜 이렇게 나누는가?

과거에는 `actions/`, `reducers/`, `constants/` 처럼 **파일 타입별로 폴더를 나누는 방식**(Rails-style)을 많이 썼다.

```
// 예전 방식 (Rails-style) - 공식문서에서 비권장
src/
├── actions/
│   ├── todosActions.js
│   └── filtersActions.js
├── reducers/
│   ├── todosReducer.js
│   └── filtersReducer.js
└── constants/
    └── actionTypes.js
```

이 방식의 문제는 "할 일" 기능 하나를 수정하려면 `actions/`, `reducers/`, `constants/` 세 폴더를 왔다갔다 해야 한다는 것이다. 기능과 관련된 코드가 여기저기 흩어져 있어서 유지보수가 불편하다.

Feature Folder 방식은 관련된 코드를 한 폴더 안에 모아두므로, **하나의 기능을 수정할 때 한 폴더만 보면 된다.**

## 6.2 파일별 역할과 작성법

실제 Todo 앱을 예시로 각 파일이 어떻게 생겼는지 살펴본다.

---

### A. `features/todos/todosSlice.js` - Reducer + Action + Selector

기능 하나의 Redux 로직을 담당하는 핵심 파일이다. (Ducks 패턴)

```js
// src/features/todos/todosSlice.js

//  초기 State 정의
const initialState = [
  { id: 0, text: "Learn React", completed: true },
  { id: 1, text: "Learn Redux", completed: false },
];

//  ID 자동 증가 헬퍼 함수
function nextTodoId(todos) {
  const maxId = todos.reduce((maxId, todo) => Math.max(todo.id, maxId), -1);
  return maxId + 1;
}

//  Reducer
// state 기본값으로 initialState를 지정한다.
// 앱이 처음 실행될 때 state가 undefined이면 initialState가 사용된다.
export default function todosReducer(state = initialState, action) {
  switch (action.type) {
    case "todos/todoAdded": {
      // state를 직접 수정하지 않고, 새 배열을 반환한다 (불변성)
      return [
        ...state,
        {
          id: nextTodoId(state),
          text: action.payload,
          completed: false,
        },
      ];
    }
    case "todos/todoToggled": {
      return state.map((todo) => {
        if (todo.id !== action.payload) {
          return todo; // 해당 todo가 아니면 그대로 반환
        }
        // 해당 todo만 completed를 반전시킨 새 객체로 반환
        return { ...todo, completed: !todo.completed };
      });
    }
    default:
      return state;
  }
}

//  Action Creators
// action 객체를 직접 작성하는 대신, 함수로 만들어 재사용한다.
export function todoAdded(text) {
  return { type: "todos/todoAdded", payload: text };
}

export function todoToggled(id) {
  return { type: "todos/todoToggled", payload: id };
}

//  Selectors
// state에서 필요한 데이터를 추출하는 함수를 reducer와 같은 파일에 정의한다.
export const selectTodos = (state) => state.todos;
export const selectCompletedTodos = (state) =>
  state.todos.filter((todo) => todo.completed);
```

---

### B. `features/filters/filtersSlice.js` - 또 다른 기능의 Slice

```js
// src/features/filters/filtersSlice.js

const initialState = {
  status: "All", // 'All' | 'Active' | 'Completed'
};

export default function filtersReducer(state = initialState, action) {
  switch (action.type) {
    case "filters/statusFilterChanged": {
      return { ...state, status: action.payload };
    }
    default:
      return state;
  }
}

export function statusFilterChanged(status) {
  return { type: "filters/statusFilterChanged", payload: status };
}

export const selectFilterStatus = (state) => state.filters.status;
```

---

### C. `app/rootReducer.js` - 모든 Reducer를 하나로 합치기

앱에는 Store가 하나뿐이므로, 여러 개의 reducer를 **하나의 root reducer로 합쳐야** 한다.
Redux가 제공하는 `combineReducers`를 사용한다.

```js
// src/app/rootReducer.js
import { combineReducers } from "redux";
import todosReducer from "../features/todos/todosSlice";
import filtersReducer from "../features/filters/filtersSlice";

const rootReducer = combineReducers({
  // 키 이름이 곧 state 안의 필드 이름이 된다.
  // store.getState()를 호출하면 { todos: [...], filters: {...} } 형태가 된다.
  todos: todosReducer,
  filters: filtersReducer,
});

export default rootReducer;
```

`combineReducers`는 각 reducer가 **자신이 담당하는 state 조각(slice)만 받아서 처리**하도록 연결해준다.

```
전체 state = {
  todos:   todosReducer가 관리    → [{ id, text, completed }, ...]
  filters: filtersReducer가 관리  → { status: 'All' }
}
```

### Point: combineReducers를 쓰면 각 reducer는 자기 slice만 본다

```js
// combineReducers 이전: 전체 state를 직접 다뤄야 함
function appReducer(state = initialState, action) {
  return {
    todos: todosReducer(state.todos, action), // 매번 이렇게 꺼내야 함
    filters: filtersReducer(state.filters, action),
  };
}

// combineReducers 이후: 위 코드를 자동으로 처리해줌
const rootReducer = combineReducers({
  todos: todosReducer,
  filters: filtersReducer,
});
// todosReducer는 state.todos만 받고, filtersReducer는 state.filters만 받는다.
```

---

### Step 4. `app/store.js` - Store 생성

root reducer를 받아서 Store를 만든다.

```js
// src/app/store.js
import { legacy_createStore as createStore } from "redux";
import rootReducer from "./rootReducer";

const store = createStore(rootReducer);

export default store;
```

---

### Step 5. `index.js` - Store를 앱 전체에 연결하기

React와 Redux를 연결하려면 `react-redux` 패키지의 `<Provider>` 컴포넌트가 필요하다.
`<Provider>`로 앱 전체를 감싸면, 그 안의 모든 컴포넌트가 Store에 접근할 수 있게 된다.

```js
// src/index.js
import React from "react";
import { createRoot } from "react-dom/client";
import { Provider } from "react-redux"; // react-redux에서 Provider import
import App from "./App";
import store from "./app/store"; // 만들어둔 store import

const root = createRoot(document.getElementById("root"));

root.render(
  // Provider로 App 전체를 감싸고, store를 prop으로 전달한다.
  // 이제 App 안의 어떤 컴포넌트도 store에 접근할 수 있다.
  <Provider store={store}>
    <App />
  </Provider>,
);
```

## 6.3 실제 동작 확인 - dispatch 테스트

UI 없이도 `index.js`에서 직접 dispatch를 호출해서 동작을 검증할 수 있다.

```js
// src/index.js (테스트용)
import store from "./app/store";

// 초기 state 출력
console.log("초기 state:", store.getState());
// { todos: [{...}, {...}], filters: { status: 'All' } }

// state가 바뀔 때마다 출력
const unsubscribe = store.subscribe(() => {
  console.log("변경된 state:", store.getState());
});

// Action dispatch
store.dispatch({ type: "todos/todoAdded", payload: "Redux 공부하기" });
// todos 배열에 새 항목이 추가된 state가 출력됨

store.dispatch({ type: "todos/todoToggled", payload: 0 });
// id가 0인 todo의 completed가 반전된 state가 출력됨

store.dispatch({ type: "filters/statusFilterChanged", payload: "Active" });
// filters.status가 'Active'로 변경된 state가 출력됨

// 구독 해제
unsubscribe();
```

## 6.4 보일러플레이트 전체 흐름 요약

```
① 기능별 Slice 파일 작성       → features/todos/todosSlice.js
   (initialState + Reducer + Action Creator + Selector)

② 모든 Reducer를 하나로 합침   → app/rootReducer.js
   (combineReducers 사용)

③ Store 생성                   → app/store.js
   (createStore에 rootReducer 전달)

④ 앱 전체에 Store 연결         → index.js
   (<Provider store={store}>로 감싸기)
```

이 네 단계가 Redux 레거시 방식의 기본 세팅이다.
실제 프로젝트에서는 **Redux Toolkit의 `configureStore`와 `createSlice`** 를 사용하면 이 보일러플레이트를 훨씬 간결하게 작성할 수 있다.

## 6.5 요약

todosSlice.js → Reducer + Action Creator + Selector를 한 파일에
filtersSlice.js → 두 번째 기능 예시
rootReducer.js → combineReducers로 합치기 (이전/이후 비교 포함)
store.js → Store 생성
index.js → <Provider>로 앱에 연결

---

# 7. Redux의 3가지 원칙 (Three Principles)

공식문서가 강조하는 Redux 설계의 핵심 원칙이다.

## 7.1 Single Source of Truth (단일 진실 공급원)

앱의 전체 state는 **하나의 store 안에 하나의 객체 트리**로 저장된다.
같은 데이터가 여러 곳에 중복되어 존재하지 않는다.

## 7.2 State is Read-Only (state는 읽기 전용)

state를 변경하는 **유일한 방법은 action을 dispatch하는 것**이다.
직접 수정하는 것은 절대 불가능하다.

## 7.3 Changes are Made with Pure Functions (순수 함수로 변경)

state가 어떻게 변경될지는 **순수 함수인 Reducer**가 결정한다.
같은 state와 action이 주어지면 항상 같은 결과를 반환한다.

---

# 8. 핵심 용어 정리

| 용어               | 설명                                                            |
| ------------------ | --------------------------------------------------------------- |
| **Store**          | 앱 전체 state를 저장하는 중앙 저장소 (앱에 하나만 존재)         |
| **State**          | 현재 앱의 상태를 나타내는 객체                                  |
| **Action**         | "무슨 일이 일어났는지"를 설명하는 plain 객체 (`type` 필드 필수) |
| **Action Creator** | Action 객체를 반환하는 함수                                     |
| **Reducer**        | `(state, action) => newState` 형태의 순수 함수                  |
| **Dispatch**       | Action을 Store에 전달해 state 변경을 요청하는 행위              |
| **Selector**       | state에서 필요한 데이터를 추출하는 함수                         |
| **Immutability**   | state를 직접 수정하지 않고 복사본을 만들어 변경하는 원칙        |

---

> **다음 스터디 (Redux Toolkit)에서 다룰 내용 (예정...)**
>
> - Redux Toolkit (RTK) - 현재 공식 권장 방식
> - `configureStore`, `createSlice`, `createAsyncThunk`
> - React-Redux 연결 (`useSelector`, `useDispatch`, `<Provider>`)
