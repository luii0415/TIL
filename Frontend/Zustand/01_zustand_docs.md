# Zustand 공식문서 스터디 - TIL

> 작성일자 : 2026.02.28

- 멋쟁이 사자처럼 14기 FE 운영진 스터디 내용인 'Zustand 공식문서 스터디'에 관한 내용을 작성했다.
- TIL 작성 시점, [Zustand](https://zustand.docs.pmnd.rs/) 공식문서의 ver은 **v5.0.11**이다.
- 작성한 것은 JS 기반의 Zustand 내용이다. (TypeScript가 아님.)

## 참고 자료

- [Zustand 공식문서](https://zustand.docs.pmnd.rs/)
  - [GitHub README (JS 버전)](https://github.com/pmndrs/zustand)
  - [API: create](https://zustand.docs.pmnd.rs/reference/apis/create)
  - [API: createStore](https://zustand.docs.pmnd.rs/apis/create-store)
  - [Guides: Immutable State and Merging](https://zustand.docs.pmnd.rs/guides/immutable-state-and-merging)
  - [Middlewares: persist](https://zustand.docs.pmnd.rs/middlewares/persist)
  - [Middlewares: devtools](https://zustand.docs.pmnd.rs/middlewares/devtools)
  - [Migrations: Migrating to v5](https://zustand.docs.pmnd.rs/migrations/migrating-to-v5)

---

# 1. Zustand란?

## 1.1 Zustand를 왜 쓰는가?

Zustand는 React를 위한 전역 상태 관리 라이브러리다. 독일어로 상태(State)를 뜻한다.

공식문서는 Zustand를 "작고(small), 빠르고(fast), 확장 가능한(scalable)" 상태 관리 솔루션으로 소개한다.

- Provider 불필요: <Provider>로 앱을 감쌀 필요가 없다. Store를 만들고 컴포넌트에서 바로 import해서 쓰면 된다.
- 훅 기반: Store 자체가 훅이다. useBearStore()처럼 호출해서 상태와 액션을 꺼낸다.
- 보일러플레이트 최소화: 파일 하나에 상태와 업데이트 함수를 함께 정의한다.
- 초소형 번들 크기: 약 1KB 수준으로 매우 가볍다.
- 비동기 지원 내장: async/await를 그냥 쓰면 된다. 별도 미들웨어가 불필요하다.

---

# 2. Zustand의 핵심 개념

## 2.1 Store (스토어)

Zustand에서 Store는 **상태(state)와 그것을 바꾸는 함수(action)를 함께 담은 훅**이다.

`create()` 함수 하나로 만든다.

```js
import { create } from "zustand";

const useUserStore = create((set) => ({
  // 상태 (state)
  isLoggedIn: false,
  username: "",

  // 상태를 바꾸는 함수 (action)
  login: (name) => set({ isLoggedIn: true, username: name }),
  logout: () => set({ isLoggedIn: false, username: "" }),
}));
```

- create()의 인자로 **함수**를 전달한다. 이 함수가 초기 상태와 액션을 정의한다.
- set은 Zustand가 주입해주는 **상태 업데이트 함수**다.
- 반환되는 useUserStore는 hook이다. 컴포넌트 안에서 바로 호출해서 쓴다.

## 2.2 set 함수 - 상태를 어떻게 바꾸는가

set은 Zustand에서 상태를 업데이트하는 유일한 공식 방법이다.

```js
// 방법 1: 새 값을 직접 전달 (덮어쓰기 방식)
set({ isLoggedIn: true });

// 방법 2: 이전 state를 참조해서 업데이트 (함수 방식)
set((state) => ({ visitCount: state.visitCount + 1 }));
```

### Point: set은 얕은 병합(Shallow Merge)을 한다

{ ...state } 스프레드를 직접 쓸 필요가 없다. set은 기본적으로 기존 상태와 새 값을 자동으로 병합한다.

```js
// 스토어에 { theme: "dark", language: "ko" } 가 있을 때
set({ theme: "light" });
// → 결과: { theme: "light", language: "ko" }  (language는 그대로 유지됨)
```

단, 이 병합은 1단계(얕은) 수준에서만 이루어진다. 중첩 객체는 직접 스프레드해야 한다.
스프레드: ... 문법을 사용해 객체나 배열의 내용을 풀어서 복사하는 것.

```js
// 잘못된 예: 중첩 객체 통째로 덮어씀
const useStore = create((set) => ({
  user: { name: "Alice", role: "viewer" },
  updateRole: (role) => set({ user: { role } }), // name이 사라짐!
}));

// 올바른 예: 중첩 객체를 직접 스프레드
const useStore = create((set) => ({
  user: { name: "Alice", role: "viewer" },
  updateRole: (role) =>
    set((state) => ({
      user: { ...state.user, role }, // name은 유지, role만 변경
    })),
}));
```

## 2.3 get 함수 - 액션 안에서 현재 state 읽기

set 외에 get도 사용할 수 있다. 액션 함수 내부에서 현재 state를 읽어야 할 때 쓴다.

```js
const useSettingsStore = create((set, get) => ({
  theme: "light",
  applyTheme: () => {
    const theme = get().theme; // 현재 state 읽기
    document.body.dataset.theme = theme;
  },
}));
```

---

# 3. Store 사용하기 - 컴포넌트에서 읽고 쓰기

## 3.1 기본 사용법

Store를 만들었으면 컴포넌트에서 바로 import해서 쓰면 된다. Provider 없이.

```js
import { create } from "zustand";

// Store 정의
const useModalStore = create((set) => ({
  isOpen: false,
  open: () => set({ isOpen: true }),
  close: () => set({ isOpen: false }),
}));

// 컴포넌트에서 사용
function ModalTrigger() {
  const open = useModalStore((state) => state.open);
  return <button onClick={open}>모달 열기</button>;
}

function Modal() {
  const isOpen = useModalStore((state) => state.isOpen);
  const close = useModalStore((state) => state.close);
  if (!isOpen) return null;
  return (
    <div>
      <p>모달 내용</p>
      <button onClick={close}>닫기</button>
    </div>
  );
}
```

- useModalStore를 호출할 때 **Selector 함수**를 인자로 넘긴다.
- Selector는 전체 state 중 '이 컴포넌트가 필요한 부분만 골라내는 함수'다.
- Zustand는 selector가 반환한 값이 변경될 때만 해당 컴포넌트를 리렌더링한다.

## 3.2 Selector - 왜 꼭 써야 하는가

Selector 없이 Store 전체를 꺼내오는 방식은 사용하지 않는 것을 권장한다.

```js
// 권장하지 않는 방식: state 전체를 꺼냄
const { isOpen, close } = useModalStore();
// store의 어떤 값이 바뀌어도 이 컴포넌트가 리렌더링됨

// 권장하는 방식: 필요한 값만 selector로 꺼냄
const isOpen = useModalStore((state) => state.isOpen);
// isOpen이 바뀔 때만 리렌더링됨
```

Zustand는 selector가 반환한 값을 `===` (참조 비교)로 이전 값과 비교한다. 값이 같으면 리렌더링을 건너뛴다.

### Point: 여러 값을 한 번에 가져올 때 - useShallow

여러 값을 한 번에 가져오고 싶을 때 객체로 묶으면, 매 렌더링마다 새 객체가 생성되어 `===` 비교가 항상 실패한다.

```js
// 문제: 매번 새 객체가 생성되어 항상 리렌더링됨
const { username, role } = useUserStore((state) => ({
  username: state.username,
  role: state.role,
})); // 무한 리렌더링 위험
```

이 경우 v5에서 공식 도입된 useShallow를 사용해야 한다.

```js
import { useShallow } from "zustand/shallow";

const { username, role } = useUserStore(
  useShallow((state) => ({ username: state.username, role: state.role })),
); // 실제 값이 바뀔 때만 리렌더링
```

useShallow는 객체 안의 각 필드를 개별적으로 비교(shallow equality)하므로, 값이 같으면 리렌더링을 건너뛴다.

---

# 4. 비동기 처리 - async/await를 그냥 쓴다

Zustand는 비동기 처리를 위한 별도 도구가 필요 없다. **액션 함수를 async로 만들면 끝이다.**

```js
import { create } from "zustand";

const useProductStore = create((set) => ({
  products: [],
  fetchProducts: async (categoryId) => {
    const response = await fetch(`/api/products?category=${categoryId}`);
    const data = await response.json();
    set({ products: data });
  },
}));
```

> Zustand는 set이 언제 호출되는지 신경 쓰지 않는다. async 함수 안에서 await 이후에 set을 호출해도 정상적으로 동작한다.

```js
// 컴포넌트에서 사용
function ProductList() {
  const products = useProductStore((state) => state.products);
  const fetchProducts = useProductStore((state) => state.fetchProducts);

  useEffect(() => {
    fetchProducts("electronics");
  }, [fetchProducts]);

  return (
    <ul>
      {products.map((p) => (
        <li key={p.id}>{p.name}</li>
      ))}
    </ul>
  );
}
```

---

# 5. 핵심 미들웨어

Zustand는 Store를 생성할 때 미들웨어를 감싸(wrap)는 방식으로 기능을 확장한다.

## 5.1 persist - 새로고침해도 state를 유지하고 싶을 때

persist 미들웨어는 Store의 상태를 localStorage(또는 sessionStorage)에 자동으로 저장하고 복원한다.

```js
import { create } from "zustand";
import { persist, createJSONStorage } from "zustand/middleware";

const useSettingsStore = create(
  persist(
    (set) => ({
      theme: "light",
      language: "ko",
      setTheme: (theme) => set({ theme }),
      setLanguage: (language) => set({ language }),
    }),
    {
      name: "app-settings", // localStorage에 저장될 key 이름 (필수, 유일해야 함)
      storage: createJSONStorage(() => sessionStorage), // 선택사항. 기본값은 localStorage
    },
  ),
);
```

- persist로 기존 state creator 함수를 감싸면 된다.
- 페이지를 새로고침해도 name에 지정한 key로 저장된 값을 자동으로 불러온다.

### v5에서 바뀐 점

v4에서는 store 생성 시 초기 state가 자동으로 storage에 저장됐다. v5부터는 이 동작이 제거됐다. 초기 state를 storage에 반영하고 싶다면 store 생성 후 명시적으로 setState를 호출해야 한다.

```js
// v5 방식
const useSettingsStore = create(
  persist(() => ({ theme: "light" }), { name: "app-settings" }),
);

// 초기 state를 storage에 반영하고 싶다면 명시적으로 호출
useSettingsStore.setState({ theme: "dark" });
```

## 5.2 devtools - Redux DevTools로 디버깅하기

`devtools` 미들웨어를 사용하면 Redux DevTools Extension과 연결해서 상태 변화를 시각적으로 추적할 수 있다.

```js
import { create } from "zustand";
import { devtools } from "zustand/middleware";

const useUserStore = create(
  devtools((set) => ({
    isLoggedIn: false,
    username: "",
    login: (name) =>
      set(
        { isLoggedIn: true, username: name },
        false, // 두 번째 인자: replace 여부 (false = 병합)
        "login", // 세 번째 인자: DevTools에 표시될 액션 이름
      ),
    logout: () => set({ isLoggedIn: false, username: "" }, false, "logout"),
  })),
);
```

- 브라우저에 Redux DevTools Extension이 설치되어 있으면 상태 변화를 타임라인으로 볼 수 있다.
- set의 세 번째 인자로 액션 이름을 지정하면 DevTools에서 더 알아보기 쉽다.

## 5.3 미들웨어 조합하기

`persist`와 `devtools`를 함께 쓸 수 있다. 안쪽부터 바깥쪽으로 감싼다.

```js
import { create } from "zustand";
import { persist, devtools } from "zustand/middleware";

const useSettingsStore = create(
  devtools(
    // 바깥쪽: devtools가 전체를 감쌈
    persist(
      // 안쪽: persist가 state creator를 감쌈
      (set) => ({
        theme: "light",
        language: "ko",
        setTheme: (theme) => set({ theme }),
      }),
      { name: "app-settings" },
    ),
  ),
);
```

---

# 6. Zustand 보일러플레이트 - 실제로 어떻게 쓰는가?

> Zustand는 엄격한 폴더 구조를 강제하지 않는다. 하지만 프로젝트가 커질수록 기능별로 Store를 분리해야 한다.

## 6.1 폴더 구조 - 기능별 Store 분리

```
src/
├── index.js               # 앱 진입점 (Provider 불필요!)
├── App.js                 # 루트 컴포넌트
│
└── stores/                # Store 파일 모음
    ├── useUserStore.js    # 유저 관련 상태
    └── useSettingsStore.js # 설정 관련 상태
```

## 6.2 보일러플레이트 전체 흐름 요약

```
Step 1. Store 파일 작성           → stores/useUserStore.js
        (상태 + 액션 + selector를 한 파일에)

Step 2. 필요한 만큼 Store 추가    → stores/useSettingsStore.js
        (기능별로 분리하면 유지보수 편리)

Step 3. index.js 진입점 설정      → Provider 없이 그냥 렌더링

Step 4. 컴포넌트에서 바로 사용    → import 후 selector로 구독
        (useXxxStore((state) => state.xxx))
```

---

# 7. 핵심 용어 정리

| 용어              | 설명                                                                                 |
| ----------------- | ------------------------------------------------------------------------------------ |
| `create`          | Store를 만드는 함수. 반환값은 훅이다                                                 |
| `set`             | 상태를 업데이트하는 함수. 기존 상태와 얕은 병합을 수행한다                           |
| `get`             | 액션 내부에서 현재 상태를 읽는 함수                                                  |
| **Selector**      | `useXxxStore((state) => state.xxx)` 형태로 필요한 상태만 골라내는 함수               |
| `useShallow`      | 여러 값을 객체로 선택할 때 얕은 비교를 적용해 불필요한 리렌더링을 막는 훅            |
| `persist`         | 상태를 localStorage/sessionStorage에 자동 저장/복원하는 미들웨어                     |
| `devtools`        | Redux DevTools Extension에 연결하는 미들웨어                                         |
| `immer`           | Immer를 활용해 직접 수정하는 문법으로 불변 업데이트를 할 수 있게 하는 미들웨어       |
| **Shallow Merge** | `set` 호출 시 최상위 필드를 자동으로 병합하는 동작. 중첩 객체는 직접 스프레드해야 함 |

---
