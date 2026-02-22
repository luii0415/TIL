# Redux 공식문서 스터디 - TIL 2

> 작성일자 : 2026.02.22

- 멋쟁이 사자처럼 14기 FE 운영진 스터디 3주차 내용인 'Redux Toolkit 공식문서 스터디'에 관한 내용을 작성했다.
- TIL 작성 시점, [Redux Toolkit](https://redux-toolkit.js.org/introduction/getting-started) 공식문서의 ver은 **v2.x (RTK 2.11 기준)** 이다.
- 작성한 것은 JS 기반의 Redux Toolkit 내용이다. (TypeScript가 아님.)
- Redux Core(레거시) 관련 내용은 TIL 1에서 별도로 다뤘다.

## 참고 자료

- [Redux Toolkit 공식문서 - Getting Started](https://redux-toolkit.js.org/introduction/getting-started)
  - [Quick Start](https://redux-toolkit.js.org/tutorials/quick-start)
  - [API: configureStore](https://redux-toolkit.js.org/api/configureStore)
  - [API: createSlice](https://redux-toolkit.js.org/api/createSlice)
  - [API: createAsyncThunk](https://redux-toolkit.js.org/api/createAsyncThunk)
- [React-Redux 공식문서 - Hooks](https://react-redux.js.org/api/hooks)

---

# 1. Redux Toolkit(RTK)이란?

TIL 1에서 Redux Core를 직접 써보면 느끼는 것이 있다.

- `createStore`는 deprecated(아직 작동은 하지만, 앞으로 없어질 예정) 처리됐다 ([링크](https://redux.js.org/api/createstore)).
- Action Creator를 매번 손으로 작성해야 한다.
- Reducer에서 불변성을 지키려고 `...state` 스프레드를 반복해야 한다.
- `combineReducers`, `applyMiddleware`, DevTools 설정... 파일마다 보일러플레이트가 쌓인다.

Redux 공식문서는 이 문제들을 직접 인정하면서 RTK를 만든 이유를 이렇게 설명한다.

- "Redux Store 설정이 너무 복잡하다"
- "유용하게 쓰려면 패키지를 너무 많이 설치해야 한다"
- "보일러플레이트 코드가 너무 많다"

그래서 Redux Toolkit(RTK)은 Redux를 올바르게 사용하기 위한 공식 표준 방식으로 만들어졌다.
RTK는 Redux Core를 내부적으로 사용하면서, 번거로운 설정을 자동화하고 코드를 간결하게 만들어준다.

> TIL 1의 레거시 방식과 RTK를 비교하면서 읽으면 이해가 더 빠르다.

---

# 2. RTK가 제공하는 핵심 API

RTK 패키지(`@reduxjs/toolkit`)에는 다음 API들이 포함되어 있다.

| API                   | 역할                                                 |
| --------------------- | ---------------------------------------------------- |
| `configureStore`      | Store 생성 (createStore의 현대적 대체)               |
| `createSlice`         | Reducer + Action Creator + Action Type을 한번에 생성 |
| `createAsyncThunk`    | 비동기 로직(API 호출 등) 처리                        |
| `createReducer`       | builder 패턴으로 reducer 작성                        |
| `createAction`        | Action Creator만 별도로 생성                         |
| `createEntityAdapter` | 정규화된 데이터 관리용 유틸                          |
| `createSelector`      | Reselect 라이브러리의 메모이제이션 셀렉터            |

이 중에서 실무에서 가장 자주 쓰이는 `configureStore`, `createSlice`, `createAsyncThunk`를 집중적으로 다룬다.

---

# 3. configureStore

## 3.1 개념

`configureStore`는 TIL 1에서 사용한 레거시 `createStore`를 대체하는 **공식 Store 생성 함수**다.

레거시 방식과 비교하면 달라지는 점이 분명하다.

```js
// 레거시 방식 (TIL 1)
import {
  legacy_createStore as createStore,
  combineReducers,
  applyMiddleware,
} from "redux";
import { composeWithDevTools } from "redux-devtools-extension";
import thunk from "redux-thunk";

const rootReducer = combineReducers({
  todos: todosReducer,
  filters: filtersReducer,
});
const store = createStore(
  rootReducer,
  composeWithDevTools(applyMiddleware(thunk)),
);

// RTK 방식
import { configureStore } from "@reduxjs/toolkit";

const store = configureStore({
  reducer: { todos: todosReducer, filters: filtersReducer },
});
// combineReducers, applyMiddleware, DevTools 연결이 자동으로 처리된다.
```

`configureStore`를 호출하면 다음이 자동으로 처리된다.

- `reducer` 객체를 받으면 내부적으로 `combineReducers`를 호출한다.
- `redux-thunk` 미들웨어가 기본으로 포함된다.
- 개발 환경에서 state 직접 변이(mutation) 감지 미들웨어가 자동으로 추가된다.
- Redux DevTools Extension 연결이 자동으로 활성화된다.

## 3.2 파라미터

`configureStore`는 하나의 설정 객체를 인자로 받는다.

```js
const store = configureStore({
  reducer, // 필수. reducer 함수 하나 또는 reducer 객체
  middleware, // 선택. 미들웨어 배열을 반환하는 콜백 함수
  devTools, // 선택. DevTools 활성화 여부 (기본값: true)
  preloadedState, // 선택. store의 초기 state 값
  enhancers, // 선택. store enhancer 배열을 반환하는 콜백 함수
});
```

### `reducer`

단일 reducer 함수 하나를 넘기거나, 여러 reducer를 담은 객체를 넘긴다.
객체를 넘기면 `configureStore`가 내부적으로 `combineReducers`를 호출한다.

```js
// reducer 하나만 넘기는 경우
const store = configureStore({ reducer: rootReducer });

// 여러 reducer를 객체로 넘기는 경우 (이 경우가 일반적)
const store = configureStore({
  reducer: {
    todos: todosReducer,
    filters: filtersReducer,
  },
});
```

### `middleware`

기본 미들웨어에 추가로 미들웨어를 붙이려면 `getDefaultMiddleware`를 활용한다.

```js
const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) => getDefaultMiddleware().concat(logger), // 기본 미들웨어에 logger 추가
});
```

### `devTools`

기본값은 `true`다. 프로덕션 환경에서는 끄는 것이 일반적이다.

```js
const store = configureStore({
  reducer: rootReducer,
  devTools: process.env.NODE_ENV !== "production",
});
```

---

# 4. createSlice

## 4.1 개념

`createSlice`는 RTK에서 가장 핵심적인 API다.
TIL 1에서 따로따로 작성하던 **initialState, Reducer, Action Creator, Action Type**을 **하나의 함수 호출로 한꺼번에 생성**해준다.

레거시 방식 vs RTK 방식을 비교하면 차이가 극명하다.

```js
// 레거시 방식 (TIL 1) - todos 기능 하나를 만들 때
const ADD_TODO = "todos/todoAdded";
const TOGGLE_TODO = "todos/todoToggled";

export function todoAdded(text) {
  return { type: ADD_TODO, payload: text };
}
export function todoToggled(id) {
  return { type: TOGGLE_TODO, payload: id };
}

const initialState = [];
export default function todosReducer(state = initialState, action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        { id: Date.now(), text: action.payload, completed: false },
      ];
    case TOGGLE_TODO:
      return state.map((todo) =>
        todo.id === action.payload
          ? { ...todo, completed: !todo.completed }
          : todo,
      );
    default:
      return state;
  }
}
```

```js
// RTK 방식 - 같은 기능을 createSlice로 작성
import { createSlice } from "@reduxjs/toolkit";

const todosSlice = createSlice({
  name: "todos",
  initialState: [],
  reducers: {
    todoAdded(state, action) {
      state.push({ id: Date.now(), text: action.payload, completed: false });
      // state를 직접 수정하는 것처럼 보이지만 실제로는 불변 업데이트가 적용된다. (Immer 덕분)
    },
    todoToggled(state, action) {
      const todo = state.find((todo) => todo.id === action.payload);
      if (todo) {
        todo.completed = !todo.completed; // 직접 수정처럼 보이지만 실제로는 불변 업데이트
      }
    },
  },
});

export const { todoAdded, todoToggled } = todosSlice.actions; // Action Creator 자동 생성
export default todosSlice.reducer; // Reducer 자동 생성
```

코드 양이 확연히 줄어들었다.

## 4.2 파라미터

`createSlice`는 설정 객체 하나를 인자로 받는다.

```js
const mySlice = createSlice({
  name, // 필수. slice의 이름. action type 문자열의 prefix가 된다.
  initialState, // 필수. 이 slice가 관리하는 state의 초기값
  reducers, // 필수. case reducer들을 담은 객체 (또는 콜백 함수)
  extraReducers, // 선택. 이 slice 외부에서 정의된 action을 처리할 때 사용
  selectors, // 선택. slice state에서 데이터를 추출하는 selector 함수 정의
});
```

### `name`

slice의 이름을 지정한다. 이 값이 자동 생성되는 action type의 prefix가 된다.

```js
// name: "todos", reducers 키: "todoAdded" 이면
// 자동 생성되는 action type은 "todos/todoAdded"가 된다.
```

### `reducers`

각 case reducer를 객체의 키-값으로 정의한다.
각 키 이름이 action type의 뒷부분이 되고, 해당 action에 대응하는 Action Creator가 자동으로 생성된다.

```js
reducers: {
  todoAdded(state, action) { ... },    // action type: "todos/todoAdded"
  todoToggled(state, action) { ... },  // action type: "todos/todoToggled"
}
```

### `extraReducers`

이 slice에서 정의하지 않은 다른 action(예: `createAsyncThunk`가 만들어내는 action)에 반응할 때 사용한다.
builder 패턴을 사용해서 작성한다.

```js
extraReducers: (builder) => {
  builder
    .addCase(fetchTodos.pending, (state) => {
      state.loading = true;
    })
    .addCase(fetchTodos.fulfilled, (state, action) => {
      state.loading = false;
      state.todos = action.payload;
    })
    .addCase(fetchTodos.rejected, (state, action) => {
      state.loading = false;
      state.error = action.error.message;
    });
},
```

## 4.3 반환값

`createSlice`는 다음 구조의 객체를 반환한다.

```js
{
  name: "todos",
  reducer: Function,    // store에 등록할 reducer 함수
  actions: {            // 자동 생성된 Action Creator들
    todoAdded: Function,
    todoToggled: Function,
  },
  caseReducers: { ... },
  getInitialState: Function,
}
```

실제 사용할 때는 `actions`와 `reducer`만 주로 export한다.

```js
export const { todoAdded, todoToggled } = todosSlice.actions;
export default todosSlice.reducer;
```

## 4.4 Immer와 불변성

RTK의 `createSlice`와 `createReducer`는 내부적으로 **Immer** 라이브러리를 사용한다.
Immer 덕분에 reducer 안에서 state를 직접 수정하는 것처럼 코드를 작성해도, 실제로는 불변 업데이트가 적용된다.

```js
reducers: {
  // 잘못된 것처럼 보이지만, Immer가 처리해줘서 실제로는 올바른 코드다.
  todoAdded(state, action) {
    state.push({ text: action.payload, completed: false }); // push로 직접 추가해도 OK
  },
  todoToggled(state, action) {
    const todo = state.find((t) => t.id === action.payload);
    todo.completed = !todo.completed; // 직접 수정해도 OK
  },
}
```

### Point: Immer가 없는 곳에서는 여전히 불변성을 지켜야 한다

Immer는 `createSlice`와 `createReducer` 안에서만 동작한다.
컴포넌트나 그 외의 코드에서 Redux state를 직접 수정하면 Immer가 개입하지 않는다.

```js
// 컴포넌트 안에서 state를 직접 수정하는 것은 여전히 금지다.
const todos = useSelector((state) => state.todos);
todos.push({ text: "새 할 일" }); // 이렇게 하면 절대 안 된다.
```

---

# 5. createAsyncThunk

## 5.1 개념

Redux에서 API 호출처럼 비동기 작업을 처리하려면 **Thunk 미들웨어**가 필요하다.
Thunk는 **함수를 반환하는 action creator**다. dispatch가 함수를 받으면 그 함수를 실행해줘서 비동기 로직을 처리할 수 있게 된다.

`createAsyncThunk`는 이 Thunk 패턴을 다루기 쉽게 만든 RTK API다.
action type prefix 문자열과 비동기 함수(payload creator)를 받아서, 자동으로 3가지 action을 생성해준다.

```
fetchTodos.pending   → 요청 시작 시 dispatch (type: "todos/fetchAll/pending")
fetchTodos.fulfilled → 요청 성공 시 dispatch (type: "todos/fetchAll/fulfilled")
fetchTodos.rejected  → 요청 실패 시 dispatch (type: "todos/fetchAll/rejected")
```

## 5.2 사용법

```js
import { createAsyncThunk } from "@reduxjs/toolkit";

// 첫 번째 인자: action type prefix 문자열
// 두 번째 인자: 비동기 작업을 수행하는 payload creator 함수 (반드시 Promise를 반환해야 함)
export const fetchTodos = createAsyncThunk("todos/fetchAll", async () => {
  const response = await fetch("https://jsonplaceholder.typicode.com/todos");
  return await response.json(); // 이 반환값이 fulfilled action의 payload가 된다.
});
```

그리고 이 thunk가 생성하는 action들을 slice의 `extraReducers`에서 처리한다.

```js
const todosSlice = createSlice({
  name: "todos",
  initialState: {
    items: [],
    loading: false,
    error: null,
  },
  reducers: {
    // 동기 action들
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchTodos.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchTodos.fulfilled, (state, action) => {
        state.loading = false;
        state.items = action.payload; // fetch 결과가 payload로 들어온다.
      })
      .addCase(fetchTodos.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message; // 에러 메시지가 들어온다.
      });
  },
});
```

### Point: extraReducers에 사용하는 이유

`fetchTodos`는 `createAsyncThunk`로 만들어진 외부 함수다.
`createSlice`의 `reducers` 필드는 이 slice 안에서 직접 정의한 action들만 처리할 수 있다.
외부에서 만들어진 action(여기서는 fetchTodos.pending/fulfilled/rejected)을 처리하려면 반드시 `extraReducers`를 써야 한다.

---

# 6. React-Redux Hooks

RTK는 Redux 로직을 작성하는 도구다.
React 컴포넌트가 Redux Store와 직접 연결되려면 `react-redux` 패키지의 Hooks가 필요하다.

## 6.1 useSelector

Store에서 state를 읽어오는 Hook이다.
selector 함수를 인자로 받아서 그 결과를 반환하고, state가 변경될 때 컴포넌트를 리렌더링한다.

```js
import { useSelector } from "react-redux";

function TodoList() {
  // state 전체에서 todos 배열만 꺼내온다.
  const todos = useSelector((state) => state.todos.items);

  return (
    <ul>
      {todos.map((todo) => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

### Point

`useSelector`는 반환하는 값이 이전 렌더링과 다를 때만 리렌더링을 유발한다.
기본 비교는 `===` (참조 비교)다. 객체나 배열을 새로 만들어 반환하면 매번 리렌더링된다.

```js
// 잘못된 예: 매 렌더링마다 새 배열 객체가 만들어져 불필요한 리렌더링 발생
const completedTodos = useSelector((state) =>
  state.todos.items.filter((todo) => todo.completed),
);

// 올바른 예: selector를 slice 파일 안에 미리 정의해서 재사용하거나
// createSelector(Reselect)로 메모이제이션 처리한다.
```

## 6.2 useDispatch

Store의 `dispatch` 함수를 반환하는 Hook이다.
반환된 `dispatch` 함수로 action을 보내거나 thunk를 실행한다.

```js
import { useDispatch } from "react-redux";
import { todoAdded, fetchTodos } from "./todosSlice";

function AddTodo() {
  const dispatch = useDispatch();

  const handleAdd = (text) => {
    dispatch(todoAdded(text)); // 동기 action dispatch
  };

  const handleFetch = () => {
    dispatch(fetchTodos()); // 비동기 thunk dispatch
  };

  return <button onClick={handleFetch}>불러오기</button>;
}
```

---

# 7. RTK 보일러플레이트 - 실제로 어떻게 쓰는가?

TIL 1의 레거시 방식과 비교하면서 파일별로 살펴본다.

## 7.1 폴더 구조

TIL 1과 동일하게 Feature Folder 방식을 사용한다. 파일 구성은 거의 같지만, `rootReducer.js` 파일이 사라지고 `store.js` 하나에서 처리 가능해진다.

```
src/
├── index.js
├── App.js
│
├── app/
│   └── store.js          # configureStore로 Store 생성
│
└── features/
    ├── todos/
    │   ├── todosSlice.js  # createSlice로 Reducer + Action 한번에
    │   └── TodoList.js    # React 컴포넌트
    │
    └── filters/
        ├── filtersSlice.js
        └── Filters.js
```

## 7.2 파일별 역할과 작성법

---

### Step 1. `features/todos/todosSlice.js`

```js
// src/features/todos/todosSlice.js
import { createSlice, createAsyncThunk } from "@reduxjs/toolkit";

// 비동기 thunk: API에서 todos 목록을 가져온다.
export const fetchTodos = createAsyncThunk("todos/fetchAll", async () => {
  const response = await fetch(
    "https://jsonplaceholder.typicode.com/todos?_limit=5",
  );
  return await response.json();
});

const todosSlice = createSlice({
  name: "todos",
  initialState: {
    items: [
      { id: 0, text: "React 공부하기", completed: true },
      { id: 1, text: "Redux 공부하기", completed: false },
    ],
    loading: false,
    error: null,
  },
  // 이 slice 안에서 정의하는 동기 action들
  reducers: {
    todoAdded(state, action) {
      // Immer 덕분에 직접 push해도 불변 업데이트가 적용된다.
      state.items.push({
        id: Date.now(),
        text: action.payload,
        completed: false,
      });
    },
    todoToggled(state, action) {
      const todo = state.items.find((todo) => todo.id === action.payload);
      if (todo) {
        todo.completed = !todo.completed;
      }
    },
    todoRemoved(state, action) {
      state.items = state.items.filter((todo) => todo.id !== action.payload);
    },
  },
  // 이 slice 외부에서 정의된 action(fetchTodos)에 반응
  extraReducers: (builder) => {
    builder
      .addCase(fetchTodos.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchTodos.fulfilled, (state, action) => {
        state.loading = false;
        state.items = action.payload;
      })
      .addCase(fetchTodos.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  },
});

// Action Creator export
export const { todoAdded, todoToggled, todoRemoved } = todosSlice.actions;

// Selector export: slice 파일 안에서 selector를 함께 정의하는 것이 권장 방식이다.
export const selectTodos = (state) => state.todos.items;
export const selectTodosLoading = (state) => state.todos.loading;
export const selectTodosError = (state) => state.todos.error;
export const selectCompletedTodos = (state) =>
  state.todos.items.filter((todo) => todo.completed);

// Reducer export (default)
export default todosSlice.reducer;
```

---

### Step 2. `features/filters/filtersSlice.js`

```js
// src/features/filters/filtersSlice.js
import { createSlice } from "@reduxjs/toolkit";

const filtersSlice = createSlice({
  name: "filters",
  initialState: {
    status: "All", // 'All' | 'Active' | 'Completed'
  },
  reducers: {
    statusFilterChanged(state, action) {
      state.status = action.payload;
    },
  },
});

export const { statusFilterChanged } = filtersSlice.actions;
export const selectFilterStatus = (state) => state.filters.status;
export default filtersSlice.reducer;
```

---

### Step 3. `app/store.js`

RTK 방식에서는 `rootReducer.js`를 별도로 만들 필요가 없다.
`configureStore`의 `reducer` 옵션에 객체를 넘기면 내부에서 `combineReducers`를 자동으로 처리해준다.

```js
// src/app/store.js
import { configureStore } from "@reduxjs/toolkit";
import todosReducer from "../features/todos/todosSlice";
import filtersReducer from "../features/filters/filtersSlice";

const store = configureStore({
  // 객체로 넘기면 combineReducers가 자동으로 호출된다.
  reducer: {
    todos: todosReducer,
    filters: filtersReducer,
  },
  // redux-thunk, DevTools 연결은 자동이다. 별도 설정 불필요.
});

export default store;
```

레거시 방식과 비교하면 다음 두 파일이 하나로 합쳐진 셈이다.

```
// 레거시: app/rootReducer.js + app/store.js 두 파일 필요
// RTK:   app/store.js 하나로 해결
```

---

### Step 4. `index.js`

TIL 1과 완전히 동일하다. `<Provider>`로 앱을 감싸서 store를 연결한다.

```js
// src/index.js
import React from "react";
import { createRoot } from "react-dom/client";
import { Provider } from "react-redux";
import App from "./App";
import store from "./app/store";

const root = createRoot(document.getElementById("root"));

root.render(
  <Provider store={store}>
    <App />
  </Provider>,
);
```

---

### Step 5. React 컴포넌트에서 실제로 사용하기

`useSelector`로 state를 읽고, `useDispatch`로 action을 dispatch한다.

```js
// src/features/todos/TodoList.js
import React, { useEffect } from "react";
import { useSelector, useDispatch } from "react-redux";
import {
  fetchTodos,
  todoAdded,
  todoToggled,
  todoRemoved,
  selectTodos,
  selectTodosLoading,
  selectTodosError,
} from "./todosSlice";

export function TodoList() {
  const dispatch = useDispatch();
  const todos = useSelector(selectTodos);
  const loading = useSelector(selectTodosLoading);
  const error = useSelector(selectTodosError);

  // 컴포넌트가 마운트될 때 API에서 todos를 불러온다.
  useEffect(() => {
    dispatch(fetchTodos());
  }, [dispatch]);

  if (loading) return <p>불러오는 중...</p>;
  if (error) return <p>오류: {error}</p>;

  return (
    <div>
      <ul>
        {todos.map((todo) => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => dispatch(todoToggled(todo.id))} // id를 payload로 dispatch
            />
            <span>{todo.title || todo.text}</span>
            <button onClick={() => dispatch(todoRemoved(todo.id))}>삭제</button>
          </li>
        ))}
      </ul>
      <button onClick={() => dispatch(todoAdded("새 할 일"))}>추가</button>
    </div>
  );
}
```

---

## 7.3 레거시 방식과 RTK 방식 비교 요약

| 항목                | 레거시 방식 (TIL 1)                                   | RTK 방식 (TIL 2)             |
| ------------------- | ----------------------------------------------------- | ---------------------------- |
| Store 생성          | `createStore` + `combineReducers` + `applyMiddleware` | `configureStore` 하나로 처리 |
| DevTools 연결       | 별도 패키지 설치 및 설정 필요                         | 자동 활성화                  |
| Thunk 미들웨어      | 별도 설치 및 설정                                     | 기본 내장                    |
| Action Type 정의    | 상수로 직접 작성                                      | `createSlice`가 자동 생성    |
| Action Creator 작성 | 직접 함수 작성                                        | `createSlice`가 자동 생성    |
| Reducer 불변성 처리 | `...state` 스프레드 수동 작성                         | Immer로 자동 처리            |
| 비동기 처리         | 직접 thunk 함수 작성                                  | `createAsyncThunk`           |
| 파일 수             | slice + rootReducer + store = 3개                     | slice + store = 2개          |

---

## 7.4 보일러플레이트 전체 흐름 요약

```
1. features/todos/todosSlice.js 작성
   createSlice로 initialState + reducer + action creator + selector 한번에 정의
   createAsyncThunk로 비동기 action 정의, extraReducers에서 처리

2. app/store.js 작성
   configureStore의 reducer 옵션에 slice reducer들을 객체로 전달
   (combineReducers, DevTools, thunk는 자동 처리)

3. index.js 수정
   <Provider store={store}>로 앱 전체를 감싸기

4. 컴포넌트에서 사용
   useSelector로 state 읽기
   useDispatch로 action 또는 thunk dispatch
```

---

# 8. 핵심 용어 정리

| 용어               | 설명                                                                              |
| ------------------ | --------------------------------------------------------------------------------- |
| `configureStore`   | RTK의 Store 생성 함수. DevTools, thunk, combineReducers 자동 처리                 |
| `createSlice`      | name + initialState + reducers를 받아 reducer와 action creator를 자동 생성        |
| `createAsyncThunk` | 비동기 작업을 위한 thunk 생성 함수. pending/fulfilled/rejected action 자동 생성   |
| `Immer`            | createSlice 내부에서 사용. state 직접 수정처럼 보이는 코드를 불변 업데이트로 변환 |
| `extraReducers`    | 다른 slice 또는 createAsyncThunk에서 만들어진 action을 처리할 때 사용             |
| `useSelector`      | React 컴포넌트에서 Redux state를 읽는 Hook                                        |
| `useDispatch`      | React 컴포넌트에서 dispatch 함수를 가져오는 Hook                                  |
| `Provider`         | react-redux의 컴포넌트. store를 앱 전체에 공급하는 역할                           |
| `Thunk`            | 함수를 반환하는 action creator. 비동기 로직 처리에 사용                           |
| `Slice`            | 특정 기능의 Redux 로직(reducer + actions + selectors)을 담은 파일 단위            |
