# TanStack Query 공식문서 스터디 - TIL

> 작성일자 : 2026.03.08

- 멋쟁이 사자처럼 14기 FE 운영진 스터디 내용인 'TanStack Query 공식문서 스터디'에 관한 내용을 작성했다.
- TIL 작성 시점, [@tanstack/react-query](https://tanstack.com/query/latest) 공식문서의 ver은 **v5.90.21**이다.
- 작성한 것은 TS(TypeScript) 기반의 TanStack Query 내용이다.

## 참고 자료

- [TanStack Query 공식문서](https://tanstack.com/query/latest)
  - [Overview](https://tanstack.com/query/v5/docs/framework/react/overview)
  - [Installation](https://tanstack.com/query/v5/docs/framework/react/installation)
  - [Important Defaults](https://tanstack.com/query/v5/docs/framework/react/guides/important-defaults)
  - [Guides: Queries](https://tanstack.com/query/v5/docs/framework/react/guides/queries)
  - [Guides: Mutations](https://tanstack.com/query/v5/docs/framework/react/guides/mutations)
  - [Guides: Query Keys](https://tanstack.com/query/v5/docs/framework/react/guides/query-keys)
  - [Guides: Caching](https://tanstack.com/query/v5/docs/framework/react/guides/caching)
  - [Guides: Query Invalidation](https://tanstack.com/query/v5/docs/framework/react/guides/query-invalidation)
  - [Guides: Infinite Queries](https://tanstack.com/query/v5/docs/framework/react/guides/infinite-queries)
  - [API: useQuery](https://tanstack.com/query/v5/docs/framework/react/reference/useQuery)
  - [API: useQueries](https://tanstack.com/query/v5/docs/framework/react/reference/useQueries)
  - [API: useInfiniteQuery](https://tanstack.com/query/v5/docs/framework/react/reference/useInfiniteQuery)
  - [API: useMutation](https://tanstack.com/query/v5/docs/framework/react/reference/useMutation)
  - [API: QueryClient](https://tanstack.com/query/v5/docs/framework/react/reference/QueryClient)
  - [Migrations: Migrating to v5](https://tanstack.com/query/v5/docs/framework/react/guides/migrating-to-v5)

---

# 1. TanStack Query란?

## 1.1 TanStack Query를 왜 쓰는가?

TanStack Query(구 React Query)는 서버 상태(Server State)를 관리하는 라이브러리다.

공식문서는 이를 "데이터 페칭, 캐싱, 동기화, 서버 상태 업데이트를 쉽게 만드는 라이브러리"로 소개한다.

React를 비롯한 대부분의 프레임워크는 데이터를 어떻게 가져오고 갱신할지에 대한 기본적인 방법을 제공하지 않는다. 그래서 개발자들은 주로 `useEffect`와 `useState`를 조합해 직접 비동기 로직을 구현하는데, 이 방식은 로딩 상태 관리, 에러 처리, 캐시 무효화 등을 모두 직접 처리해야 한다는 단점이 있다.

TanStack Query는 이 문제를 해결하기 위해 만들어졌다. 핵심 특징은 다음과 같다.

- 자동 캐싱: 동일한 데이터를 여러 컴포넌트에서 요청해도 네트워크 요청은 한 번만 발생한다.
- 자동 백그라운드 리페치: 창이 포커스되거나 네트워크가 복구되면 데이터를 자동으로 다시 가져온다.
- 로딩/에러 상태 내장: `isPending`, `isError`, `isSuccess` 등 상태 플래그를 기본으로 제공한다.
- 캐시 무효화: Mutation(데이터 변경) 후 관련 Query를 무효화해 자동으로 최신 데이터를 가져올 수 있다.
- 서버 상태와 클라이언트 상태 분리: 서버에서 온 데이터는 TanStack Query가, 클라이언트 UI 상태는 Zustand 등이 담당하는 방식으로 역할을 나눌 수 있다.

## 1.2 설치 및 기본 세팅

```bash
npm install @tanstack/react-query
```

TanStack Query를 사용하려면 앱 최상단에 `QueryClientProvider`로 감싸야 한다. 이때 `QueryClient` 인스턴스(객체)를 만들어서 넘긴다.

```tsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const queryClient = new QueryClient();

export default function App() {
  return {
    /* 앱 전체 컴포넌트 */
  };
}
```

`QueryClient`는 모든 쿼리의 캐시를 관리하는 중심 인스턴스다. `QueryClientProvider`를 통해 하위 컴포넌트 어디서든 이 캐시에 접근할 수 있다.

### DevTools (선택사항)

```bash
npm install @tanstack/react-query-devtools
```

```tsx
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";

export default function App() {
  return {
    /* 앱 전체 컴포넌트 */
  };
}
```

개발 환경에서만 렌더링되며, 캐시 상태·쿼리 키·fetch 상태 등을 시각적으로 확인할 수 있다.

---

# 2. Query Key - 캐시의 핵심 식별자

## 2.1 Query Key란?

**Query Key는 TanStack Query가 각 쿼리를 구분하고 캐시를 관리하는 유일한 식별자다.**

공식문서는 "제공된 유니크한 키는 리페치, 캐싱, 쿼리 공유에 내부적으로 사용된다"고 설명한다.

Query Key는 반드시 **배열** 형태로 작성한다.

```tsx
// 단순한 키 - 문자열 하나
useQuery({ queryKey: ["리소스명"], queryFn: fetchAll });

// 세부 ID를 포함한 키
useQuery({ queryKey: ["리소스명", id], queryFn: () => fetchOne(id) });

// 필터/옵션을 포함한 키
useQuery({
  queryKey: ["리소스명", { status: "active", page: 1 }],
  queryFn: fetchAll,
});
```

## 참고

### 동일한 쿼리 키를 써야 하는 이유(때)

TanStack Query의 캐시는 **쿼리 키를 기준으로 공유**된다. 키가 같으면 어느 컴포넌트에서 쓰든 동일한 캐시를 바라본다.

**시나리오: 게시글 상세 페이지에서 좋아요 → 메인화면에 즉시 반영**

```tsx
// 메인화면과 상세페이지가 동일한 키를 구독하고 있을 때
// 상세페이지에서 좋아요 mutation 성공 후 invalidateQueries를 날리면
// 같은 키를 가진 모든 쿼리가 동시에 무효화 → 메인화면 자동 리페치

useQuery({ queryKey: ["posts", postId] }); // 메인화면
useQuery({ queryKey: ["posts", postId] }); // 상세페이지
```

키가 다르면 한쪽을 무효화해도 다른 쪽은 갱신되지 않는다.

**해결 방법 1. 키 구조를 통일해서 prefix 매칭 활용**

```tsx
useQuery({ queryKey: ["posts"] }); // 목록
useQuery({ queryKey: ["posts", postId] }); // 단건

// 좋아요 성공 후 - posts prefix를 가진 모든 쿼리 무효화
queryClient.invalidateQueries({ queryKey: ["posts"] });
```

**해결 방법 2. setQueryData로 캐시 직접 수정 (낙관적 업데이트)**

```tsx
onSuccess: () => {
  queryClient.setQueryData(["posts", postId], (oldData) => ({
    ...oldData,
    likes: oldData.likes + 1,
  }));
};
```

### Point: Query Key와 캐시의 관계

Query Key가 다르면 별개의 캐시 항목으로 저장된다.

```tsx
// 아래 두 쿼리는 서로 다른 캐시를 가진다
useQuery({ queryKey: ["리소스명", 1], queryFn: () => fetchOne(1) });
useQuery({ queryKey: ["리소스명", 2], queryFn: () => fetchOne(2) });
```

Query Key 배열의 내용을 해시(hash)해서 캐시 키로 사용하기 때문에, 배열 안의 값이 같으면 동일한 캐시를 공유한다. 객체의 경우 프로퍼티 순서는 상관없다. `{ a: 1, b: 2 }`와 `{ b: 2, a: 1 }`은 같은 키로 취급한다.

## 2.2 캐시 생명주기 - staleTime과 gcTime

TanStack Query의 캐시는 두 단계의 시간 개념으로 관리된다.

### staleTime - 데이터가 "신선한" 시간

- 공식문서 기본값: **0** (즉시 오래된 것으로 간주)
- staleTime 동안은 캐시된 데이터를 그대로 사용하고, 네트워크 요청을 하지 않는다.
- staleTime이 지나면 데이터는 "stale(오래된)" 상태가 되어 조건에 따라 백그라운드 리페치가 발생한다.

```tsx
useQuery({
  queryKey: ["리소스명"],
  queryFn: fetchAll,
  staleTime: 5 * 60 * 1000, // 5분 동안은 캐시 데이터를 그대로 사용
});
```

### gcTime - 캐시가 메모리에 남아있는 시간

- 공식문서 기본값: **5분** (1000 _ 60 _ 5)
- 쿼리를 사용하는 컴포넌트가 모두 언마운트되면 해당 쿼리는 "비활성(inactive)" 상태가 된다.
- gcTime이 지나면 비활성 쿼리의 캐시 데이터는 메모리에서 삭제(가비지 컬렉션)된다.
- v4에서는 `cacheTime`이라는 이름이었으나 v5에서 `gcTime`으로 변경됐다.

```ts
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60 * 1000, // 모든 쿼리에 기본 staleTime 1분 적용
      gcTime: 10 * 60 * 1000, // 모든 쿼리에 기본 gcTime 10분 적용
    },
  },
});
```

### Point: staleTime과 gcTime의 차이

|         | staleTime                   | gcTime                      |
| ------- | --------------------------- | --------------------------- |
| 역할    | 리페치 여부 결정            | 메모리 삭제 여부 결정       |
| 기본값  | 0 (즉시 오래된 것으로 간주) | 5분                         |
| 만료 시 | 백그라운드 리페치 발생      | 캐시 데이터 메모리에서 삭제 |

공식문서 캐시 생명주기 예시:

```
1. useQuery({ queryKey: ['리소스명'], queryFn: fetchAll }) 마운트
   → 캐시 없음 → 네트워크 요청 발생 → ['리소스명'] 키로 데이터 캐시
   → staleTime 0이므로 데이터는 즉시 stale 상태가 됨

2. 다른 곳에서 동일한 useQuery({ queryKey: ['리소스명'] }) 마운트
   → 캐시에 ['리소스명'] 데이터 있음 → 캐시 데이터 즉시 반환
   → 데이터가 stale이므로 백그라운드에서 리페치 발생

3. 두 컴포넌트 모두 언마운트
   → ['리소스명'] 쿼리 비활성 상태가 됨
   → gcTime(5분) 타이머 시작

4. 5분 이내에 다시 마운트 시
   → 캐시 데이터를 반환하면서 백그라운드 리페치

5. 5분 경과 후
   → ['리소스명'] 캐시 데이터 메모리에서 삭제
```

## 2.3 쿼리 무효화 - invalidateQueries

`invalidateQueries`는 캐시를 "만료(stale)" 상태로 강제로 표시하고, 활성 쿼리는 즉시 리페치하는 메서드다. Mutation(데이터 변경) 후 관련 데이터를 최신으로 갱신할 때 주로 사용한다.

```tsx
// '리소스명' 키로 시작하는 모든 쿼리 무효화 (prefix 매칭)
queryClient.invalidateQueries({ queryKey: ["리소스명"] });

// 아래 쿼리들 모두 무효화됨
// useQuery({ queryKey: ['리소스명'] })
// useQuery({ queryKey: ['리소스명', id] })
// useQuery({ queryKey: ['리소스명', { status: 'active' }] })
```

`exact: true`를 사용하면 정확히 일치하는 키만 무효화한다.

```tsx
// ['리소스명'] 키와 정확히 일치하는 쿼리만 무효화
queryClient.invalidateQueries({ queryKey: ["리소스명"], exact: true });
// useQuery({ queryKey: ['리소스명'] })          → 무효화됨
// useQuery({ queryKey: ['리소스명', id] })      → 무효화되지 않음
```

---

# 3. 핵심 훅 6가지

TanStack Query의 훅은 용도에 따라 크게 두 가지로 나뉜다.

- **Query 훅** (`useQuery`, `useQueries`, `useInfiniteQuery`): 데이터를 읽는(GET) 용도
- **Mutation 훅** (`useMutation`): 데이터를 변경하는(POST/PUT/PATCH/DELETE) 용도

v5부터 모든 훅은 **단일 객체** 하나를 인자로 받는다. 여러 인자를 받던 v4 방식은 제거됐다.

```ts
// v4 방식 (제거됨)
useQuery(queryKey, queryFn, options);

// v5 방식 (단일 객체)
useQuery({ queryKey, queryFn, ...options });
```

## 3.1 useQuery - 데이터 조회 (GET)

서버에서 데이터를 가져올 때 사용하는 가장 기본적인 훅이다.

```tsx
import { useQuery } from '@tanstack/react-query'

const { data, isPending, isError, error } = useQuery({
  queryKey: ['리소스명', id],   // 캐시 식별자
  queryFn: () => fetchFn(id),   // 데이터를 가져오는 비동기 함수
})

if (isPending) return 로딩 중...
if (isError) return 에러: {error.message}
// isSuccess 이후에는 data가 정의된 상태임이 보장됨
```

### 반환값 주요 상태

| 상태         | 설명                                          |
| ------------ | --------------------------------------------- |
| `data`       | 성공 시 반환된 데이터                         |
| `isPending`  | 아직 데이터가 없는 상태 (최초 로딩)           |
| `isFetching` | 백그라운드 리페치 포함, 현재 네트워크 요청 중 |
| `isError`    | 요청 실패 상태                                |
| `error`      | 에러 객체                                     |
| `isSuccess`  | 요청 성공 상태                                |
| `refetch`    | 수동으로 리페치를 트리거하는 함수             |

### Point: isPending vs isFetching

`isPending`과 `isFetching`은 다르다.

```
isPending: 캐시에 데이터가 전혀 없는 상태 (최초 로딩 스피너 표시에 사용)
isFetching: 현재 네트워크 요청이 진행 중인 상태 (백그라운드 리페치 포함)

최초 데이터 로딩:    isPending = true,  isFetching = true
백그라운드 리페치:   isPending = false, isFetching = true
데이터 있고 유휴:    isPending = false, isFetching = false
```

### 주요 옵션

```tsx
useQuery({
  queryKey: ["리소스명"],
  queryFn: fetchFn,

  // 데이터를 신선하게 유지할 시간 (기본값: 0)
  staleTime: 60 * 1000,

  // false이면 자동 실행 안 함 - 조건부 실행에 사용
  enabled: !!id,

  // 반환된 data를 변환하거나 일부만 선택
  select: (data) => data.items,

  // 로딩 중 보여줄 임시 데이터
  placeholderData: [],
});
```

`enabled` 옵션은 의존성이 있는 쿼리(Dependent Query)에 유용하다. 앞 쿼리의 결과가 있어야 다음 쿼리를 실행할 수 있는 경우에 사용한다.

```tsx
const { data: firstData } = useQuery({
  queryKey: ["첫번째리소스"],
  queryFn: fetchFirst,
});

const { data: secondData } = useQuery({
  queryKey: ["두번째리소스", firstData?.id],
  queryFn: () => fetchSecond(firstData!.id),
  enabled: !!firstData?.id, // firstData가 있을 때만 실행
});
```

---

## 3.2 useMutation - 데이터 생성 (POST)

서버에 새 데이터를 생성할 때 사용한다. `useMutation`은 자동으로 실행되지 않고, `mutate` 또는 `mutateAsync`를 명시적으로 호출해야 실행된다.

```tsx
import { useMutation, useQueryClient } from "@tanstack/react-query";

const queryClient = useQueryClient();

const { mutate, isPending, isError, error } = useMutation<
  ResponseType, // mutationFn의 반환 타입
  Error, // 에러 타입
  VariablesType // mutate()에 전달할 인자 타입
>({
  mutationFn: (variables) => postFn(variables),

  // 성공 시: 관련 쿼리 무효화 → 최신 데이터 자동 갱신
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ["리소스명"] });
  },
});

// 사용: mutate()를 명시적으로 호출해야 실행됨
mutate(variables);
```

### 반환값 주요 상태

| 상태          | 설명                                                       |
| ------------- | ---------------------------------------------------------- |
| `mutate`      | Mutation을 실행하는 함수                                   |
| `mutateAsync` | Promise를 반환하는 버전. `await`와 `try/catch`로 사용 가능 |
| `isPending`   | Mutation이 실행 중인 상태                                  |
| `isSuccess`   | Mutation 성공 상태                                         |
| `isError`     | Mutation 실패 상태                                         |
| `data`        | 성공 시 반환된 데이터                                      |
| `variables`   | `mutate`에 전달한 인자                                     |
| `reset`       | Mutation 상태를 초기화하는 함수                            |

### 주요 콜백 옵션

```tsx
useMutation({
  mutationFn: postFn,

  // 실행 직전: 낙관적 업데이트(Optimistic Update)에 주로 사용
  onMutate: (variables) => {
    // variables: mutate()에 전달한 인자
  },

  // 성공 시
  onSuccess: (data, variables, context) => {
    queryClient.invalidateQueries({ queryKey: ["리소스명"] });
  },

  // 실패 시
  onError: (error, variables, context) => {
    console.error(error);
  },

  // 성공/실패 관계없이 항상 실행
  onSettled: (data, error, variables, context) => {
    queryClient.invalidateQueries({ queryKey: ["리소스명"] });
  },
});
```

### Point: v5에서 onError, onSuccess, onSettled가 useQuery에서 제거됨

v4에서는 `useQuery`에도 `onSuccess`, `onError` 콜백을 쓸 수 있었다. v5에서는 이 콜백들이 `useQuery`에서 완전히 제거됐다. 사이드이펙트가 필요하면 `useEffect`로 처리해야 한다.

```tsx
// v4 방식 (제거됨)
useQuery({
  queryKey: ["리소스명"],
  queryFn: fetchFn,
  onSuccess: (data) => toast.success("완료"), // 사용 불가
});

// v5 방식
const { data, isSuccess } = useQuery({
  queryKey: ["리소스명"],
  queryFn: fetchFn,
});

useEffect(() => {
  if (isSuccess) toast.success("완료");
}, [isSuccess]);
```

`onSuccess`, `onError`, `onSettled` 콜백은 `useMutation`에서는 여전히 사용 가능하다.

---

## 3.3 useMutation - 데이터 수정 (PUT / PATCH)

수정도 `useMutation`을 사용한다. `mutationFn`에서 HTTP 메서드만 다를 뿐 패턴은 동일하다.

```tsx
const { mutate, isPending } = useMutation({
  mutationFn: (variables) => patchFn(variables),

  onSuccess: (data, variables) => {
    // 수정된 항목의 단건 쿼리와 목록 쿼리 모두 무효화
    queryClient.invalidateQueries({ queryKey: ["리소스명", variables.id] });
    queryClient.invalidateQueries({ queryKey: ["리소스명"] });
  },
});

mutate({ id, ...updatedFields });
```

---

## 3.4 useMutation - 데이터 삭제 (DELETE)

삭제도 `useMutation`을 사용한다.

```tsx
const { mutate, isPending } = useMutation({
  mutationFn: (id) => deleteFn(id),

  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ["리소스명"] });
  },

  onError: (error) => {
    console.error(error);
  },
});

mutate(id);
```

---

## 3.5 useQueries - 병렬 다중 조회

여러 쿼리를 동시에 실행해야 할 때 사용한다. 실행할 쿼리 수가 동적으로 결정될 때 특히 유용하다.

```tsx
import { useQueries } from "@tanstack/react-query";

const results = useQueries({
  queries: ids.map((id) => ({
    queryKey: ["리소스명", id] as const,
    queryFn: () => fetchOne(id),
  })),
});

// results는 각 쿼리 결과의 배열. 순서는 입력 순서와 동일
const isLoading = results.some((result) => result.isPending);
const dataList = results.map((result) => result.data);
```

### combine 옵션 - 결과를 하나로 합치기

v5에서 추가된 `combine` 옵션을 사용하면 여러 쿼리 결과를 하나의 값으로 합칠 수 있다.

```tsx
const { data, isPending } = useQueries({
  queries: ids.map((id) => ({
    queryKey: ["리소스명", id] as const,
    queryFn: () => fetchOne(id),
  })),
  combine: (results) => ({
    data: results.map((result) => result.data),
    isPending: results.some((result) => result.isPending),
  }),
});
```

`combine`을 사용하면 반환값이 배열 대신 단일 객체가 된다. 단, `combine` 함수에서 반환하지 않은 개별 쿼리 속성들은 접근할 수 없게 된다.

---

## 3.6 useInfiniteQuery - 무한 스크롤 조회

페이지네이션이나 무한 스크롤처럼 데이터를 "더 불러오는" 패턴에 사용한다.

```tsx
import { useInfiniteQuery } from "@tanstack/react-query";

const { data, fetchNextPage, hasNextPage, isFetchingNextPage, isPending } =
  useInfiniteQuery({
    queryKey: ["리소스명"],
    queryFn: ({ pageParam }) => fetchPage(pageParam as number),
    initialPageParam: 1, // v5 필수 옵션 - 첫 페이지 파라미터 명시
    getNextPageParam: (lastPage) => lastPage.nextPage ?? null,
    // lastPage: 가장 최근에 불러온 페이지 데이터
    // null 또는 undefined를 반환하면 hasNextPage = false
  });

// data.pages: 불러온 모든 페이지 데이터의 배열
// fetchNextPage(): 다음 페이지를 가져오는 함수
// hasNextPage: 다음 페이지가 있는지 여부
// isFetchingNextPage: 다음 페이지를 가져오는 중인지 여부
```

### 반환값 주요 상태

| 상태                 | 설명                                                                           |
| -------------------- | ------------------------------------------------------------------------------ |
| `data.pages`         | 불러온 모든 페이지 데이터의 배열                                               |
| `data.pageParams`    | 각 페이지를 불러올 때 사용한 파라미터 배열                                     |
| `fetchNextPage`      | 다음 페이지를 가져오는 함수                                                    |
| `hasNextPage`        | 다음 페이지가 있는지 여부 (`getNextPageParam`이 null/undefined가 아닐 때 true) |
| `isFetchingNextPage` | 다음 페이지를 가져오는 중인지 여부                                             |

### Point: v5에서 initialPageParam이 필수가 됨

v4에서는 `queryFn`의 `pageParam`에 기본값을 직접 할당했다. v5부터는 `initialPageParam` 옵션이 별도로 필수 추가됐다.

```tsx
// v4 방식 (제거됨)
useInfiniteQuery({
  queryKey: ["리소스명"],
  queryFn: ({ pageParam = 1 }) => fetchPage(pageParam), // 기본값을 여기서 설정
});

// v5 방식
useInfiniteQuery({
  queryKey: ["리소스명"],
  queryFn: ({ pageParam }) => fetchPage(pageParam as number),
  initialPageParam: 1, // 별도 옵션으로 명시 (필수)
  getNextPageParam: (lastPage) => lastPage.nextPage ?? null,
});
```

---

# 4. QueryClient 직접 사용하기

컴포넌트 내에서 `useQueryClient` 훅으로 `QueryClient` 인스턴스에 접근할 수 있다. 캐시를 직접 조작하거나 무효화할 때 사용한다.

```tsx
import { useQueryClient } from "@tanstack/react-query";

const queryClient = useQueryClient();

// 캐시에서 데이터 읽기 (네트워크 요청 없음)
const cachedData = queryClient.getQueryData(["리소스명"]);

// 캐시 데이터를 수동으로 업데이트
queryClient.setQueryData(["리소스명", id], (oldData) =>
  oldData ? { ...oldData, field: newValue } : oldData,
);

// 쿼리 무효화 (stale 표시 + 활성 쿼리 리페치)
queryClient.invalidateQueries({ queryKey: ["리소스명"] });
```

---

# 5. 보일러플레이트 - 실제로 어떻게 구성하는가?

> TanStack Query는 특정 폴더 구조를 강제하지 않는다.

## 5.1 폴더 구조

```
src/
├── main.tsx                  # QueryClientProvider 세팅
├── App.tsx
│
└── queries/                  # 쿼리 정의 파일 모음
    ├── useAQueries.ts        # A 관련 쿼리/뮤테이션
    └── useBQueries.ts        # B 관련 쿼리/뮤테이션
```
