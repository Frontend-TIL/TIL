# Tanstack Query 핵심 정리

## 개요

Tanstack Query는 서버로부터 가져온 데이터를 쉽고 효율적으로 관리하기 위한 라이브러리입니다.

### 주요 기능

- 데이터 가져오기 및 캐싱
- 동일 요청의 중복 제거
- 신선한 데이터 유지
- 무한 스크롤, 페이지네이션 등의 성능 최적화
- 네트워크 재연결, 요청 실패 등의 자동 갱신

### 데이터 캐싱

Tanstack Query를 활용해서 데이터를 가져올 때는 **queryKey**를 지정하게 됩니다. 이 **queryKey**를 통해 캐시된 데이터와 비교하여 새로운 데이터를 가져올지, 캐시된 데이터를 사용할지 결정하게 됩니다.

```typescript
const { data } = useQuery({
  queryKey: ['todos'],
  queryFn: async () => {
    const response = await fetch('/api/todos')
    return response.json()
  },
})
```

만약 쿼리 캐시에 todos의 key를 가진 데이터가 존재하지 않는다면, 서버에서 새로운 데이터를 가져오게 되고, 쿼리 캐시에 todos key가 존재한다면 서버로 요청을 보내지 않고 캐시된 데이터를 사용하게 됩니다. 이를 통해, 같은 데이터를 가져오는 요청을 여러 번 실행해도, 캐시된 데이터를 사용하게 되어 서버 부하를 줄일 수 있습니다. 하지만 실제 서버의 데이터는 변경되었을 수도 있기 때문에, 항상 캐시된 데이터를 그대로 사용하는 것은 좋지 않습니다. 따라서 Tanstack Query는 **데이터의 신선도**를 통해 캐시된 데이터를 사용할지, 서버에서 새로운 데이터를 가져올지 결정하게 됩니다.

### 데이터의 신선도

Tanstack Query는 캐시한 데이터를 신선(Fresh)하거나 상한(Stale) 상태로 구분해 관리합니다.
캐시된 데이터가 신선하다면 캐시된 데이터를 사용하고, 상한 상태라면 서버에 다시 요청을 보내 신선한 데이터를 가져옵니다.
데이터가 상하는 데까지 걸리는 시간은 **`staleTime`** 옵션으로 지정할 수 있습니다. 그리고 신선한지 상했는지 여부는 **`isStale`** 로 확인할 수 있습니다.

```typescript
const { data, isStale } = useQuery({
  queryKey: ['todos'],
  queryFn: async () => {
    const response = await fetch('/api/todos')
    return response.json()
  },
  staleTime: 1000 * 60 * 5, // 5분 후 상함
})
```

## 핵심 기능

### useQuery

**`useQuery`**는 서버에서 데이터를 가져올 때 사용하는 훅입니다. 

```typescript
const { data, isStale } = useQuery({
  queryKey: ['todos'],
  queryFn: async () => {
    const response = await fetch('/api/todos')
    return response.json()
  },
  staleTime: 1000 * 60 * 5, // 5분 후 상함
})
```

#### queryKey

queryKey는 쿼리를 식별하는 고유한 값으로, 배열 형태로 지정합니다.
다중 아이템 queryKey를 사용할 때는, 아이템의 순서가 중요합니다.

```typescript
// 단일 아이템 쿼리 키
useQuery({ queryKey: ['hello'] })

// 다중 아이템 쿼리 키
useQuery({ queryKey: ['hello', 'world', 123, { a: 1, b: 2 }] })

// 서로 같은 쿼리
useQuery({ queryKey: ['hello', 'world', 123, { a: 1, b: 2 }] })
useQuery({ queryKey: ['hello', 'world', 123, { b: 2, c: undefined, a: 1 }] })

// 서로 다른 쿼리
useQuery({ queryKey: ['hello', 'world', 123, { a: 1, b: 2 }] })
useQuery({ queryKey: ['hello', 'world', 123, { a: 1, b: 2, c: 3 }] })
useQuery({ queryKey: ['hello', 'world'] })
useQuery({ queryKey: [123, 'world', { a: 1, b: 2, c: 3 }, 'hello'] })
```

기본적으로 쿼리 함수(queryFn)에서 사용하는 변수는 쿼리 키에 포함돼야 합니다.
그러면 변수가 변경될 때마다 자동으로 다시 가져올 수 있습니다.
그런데 만약 변수와는 상관없이 항상 하나의 쿼리로 처리하고 싶다면, ESLint exhaustive-deps 규칙을 비활성화할 수 있습니다.

```typescript
export default function DelayedData({ wait = 1000 }: { wait: number }) {
  const { data } = useQuery({
    // eslint-disable-next-line @tanstack/query/exhaustive-deps
    queryKey: ['todos'], // ESLint Error - The following dependencies are missing in your queryKey: wait
    queryFn: async () => {
    const response = await fetch('/api/todos')
    return response.json()
  },
    staleTime: 1000 * 60 * 5
  })
  return <div>{data?.time}</div>
}
```

#### queryFn

`queryFn`은 데이터를 가져오는 비동기 함수로, 데이터를 반환하거나 오류를 던져야 합니다.
던져진 오류는 반환되는 `error` 객체로 확인할 수 있습니다.
`error`는 기본적으로 `null`입니다.

```typescript
const { data, error } = useQuery({
  queryKey: ['todos'],
  queryFn: async () => {
    const response = await fetch('/api/todos')
    const data response.json()
    if (!data.ok) {
        throw new Error('에러 발생');
    }
    return data;
  },
  staleTime: 1000 * 60 * 5
})
```

#### select
  
`select`를 사용하면 가져온 데이터를 변형할 수 있습니다.
`queryFn`이 반환하는 데이터를 인수로 받아 선택 함수에서 처리하고 반환하면 최종 데이터가 됩니다.
최종 데이터 타입은 `useQuery`의 3번째 제네릭 타입으로 선언할 수 있습니다.
2번째는 오류 타입입니다.

```typescript
const { data, error } = useQuery<Todo[], Error, string[]>({
  queryKey: ['todos'],
  queryFn: async () => {
    const response = await fetch('/api/todos')
    const data response.json()
    if (!data.ok) {
        throw new Error('에러 발생');
    }
    return data;
  },
  staleTime: 1000 * 60 * 5,
  select: data => data.map(todo => todo.title)
})
```

`queryFn`을 따로 선언해 제공하면, 선택 함수를 통한 최종 데이터의 타입을 추론할 수 있습니다.

```typescript
async function queryFn(): Promise<Todo[]> {
  const response = await fetch('/api/todos')
  return response.json()
}

const { data } = useQuery({
  queryKey: ['todos'],
  queryFn
  select: data => data.map(todo => todo.title)
})
```

#### placeholderData

새로운 데이터를 가져오는 과정에서는 쿼리가 무효화되어 일시적으로 데이터가 없는 상태가 되면서 깜빡거리는 현상이 발생할 수 있습니다.
이런 현상을 방지하기 위해 `placeholderData` 옵션을 사용하면, 쿼리 함수가 호출되는 대기 상태에서 임시로 표시할 데이터를 미리 표시할 수 있습니다.
`placeholderData`는 이전 데이터를 받아서 임시 데이터로 사용할 수 있습니다.

```typescript
const { data } = useQuery({
  queryKey: ['todos'],
  queryFn: async () => {
    const response = await fetch('/api/todos')
    return response.json()
  },
  placeholderData: prev => prev
})
```

#### structuralSharing

`structuralSharing` 옵션은 새로운 데이터를 가져올 때 이전 데이터와 비교해 변경되지 않은 부분은 이전 데이터를 재사용하도록 지정할 수 있습니다. `true`면 변경된 부분만 새롭게 업데이트하고 변경되지 않은 부분은 이전 데이터의 참조를 재사용합니다.

만약 매우 큰 중첩 객체를 다루는 경우에는 구조적인 비교가 오히려 부담될 수 있고, 데이터가 항상 새로운 참조여야 하거나 데이터가 단순해 깊은 비교가 필요하지 않은 경우에도 `false`로 지정하는 것이 유리할 수 있습니다.

#### meta

`meta`는 쿼리에 대한 추가 정보를 제공할 수 있습니다.
쿼리 클라이언트 생성의 `queryCache` 옵션에서 호출 쿼리의 추가 정보를 얻을 수 있습니다.

```typescript
import {
  QueryClient,
  QueryCache
} from '@tanstack/react-query'

const queryClient = new QueryClient({
  queryCache: new QueryCache({
    onError: (_error, query) => {
      alert(query.meta?.errorMessage)
    }
  })
})
```

```typescript
const { data } = useQuery({
  queryKey: ['todos'],
  queryFn: async () => {
    const response = await fetch('/api/todos')
    return response.json()
  },
  meta: {
    errorMessage: '투두 리스트를 가져오는 데 실패했습니다.'
  }
})
```

#### 반환 데이터

- `data`: 성공적으로 가져온 데이터
- `dataUpdatedAt`: 최근에 데이터를 성공적으로 가져온 시간(유닉스 타임스탬프)
- `error`: 오류 객체. 오류가 발생하지 않았다면 null
- `errorUpdatedAt`: 최근에 오류가 발생한 시간(유닉스 타임스탬프)
- `isError`: 쿼리 함수에서의 오류 발생 여부
- `isPending`: 쿼리가 처음으로 데이터를 가져오는 중인지 여부
- `isSuccess`: 성공적으로 데이터를 가져왔는지 여부
- `isLoading`: 쿼리 함수의 첫 번째 가져오기가 진행 중. isFetching && isPending와 같음.
- `isFetched`: 쿼리의 첫 데이터 가져오기가 완료되었는지 여부
- `isFetching`: 쿼리 함수가 실행 중.(첫 대기 및 백그라운드 다시 가져오기 포함)
- `isPaused`: 쿼리 가져오기가 일시 중단됨.
- `isStale`: 데이터가 신선한지 여부
- `isRefetching`: 데이터를 다시 가져오는 중인지 여부
- `isPlaceholderData`: 표시된 데이터가 대체 데이터인지 여부
- `refetch`: 데이터를 다시 가져오는 함수

#### refetch

`refetch` 함수를 사용하면, 데이터를 항상 새롭게 다시 가져올 수 있습니다.

```typescript
export default function DelayedData() {
  const { data, isStale, refetch } = useQuery({
    queryKey: ['todos'],
    queryFn: async () => {
      const response = await fetch('/api/todos')
      return response.json()
    },
    placeholderData: prev => prev
  })
  return (
    <>
      <div>{data?.time}</div>
      <div>데이터가 상했나요?: {JSON.stringify(isStale)}</div>
      <button onClick={() => refetch()}>데이터 가져오기</button>
    </>
  )
}
```

만약 신선도(`staleTime`) 기반으로 데이터를 가져오려면, `queryClient.fetchQuery()` 메소드를 사용할 수 있습니다.
주의할 부분은, `queryKey`와 `staleTime`를 기존 쿼리와 동일하게 제공해야 합니다.(`queryFn` 생략 가능)

```typescript
import { useQuery, useQueryClient, queryOptions } from '@tanstack/react-query'

const todoOptions = queryOptions({
  queryKey: ['todos'],
  queryFn: async () => {
    const response = await fetch('/api/todos')
    return response.json()
  },
  staleTime: 1000 * 60 * 5
})

export default function DelayedData() {
  const queryClient = useQueryClient()
  const { data, isStale } = useQuery(todoOptions)
    queryFn: async () => {
      const response = await fetch('/api/todos')
      return response.json()
    },
    placeholderData: prev => prev
  })

  function fetchData() {
    queryClient.fetchQuery(todoOptions)
  }

  return (
    <>
      <div>{data?.time}</div>
      <div>데이터가 상했나요?: {JSON.stringify(isStale)}</div>
      <button onClick={fetchData}>
        데이터 가져오기
      </button>
    </>
  )
}
```

### useInfiniteQuery

`useInfiniteQuery`는 '더 보기' 버튼을 통해 추가 데이터를 더 가져오거나 무한 스크롤 기능을 구현하기 위한 훅입니다.

```typescript
const 반환 = useInfiniteQuery<타입>(옵션)
```

#### 옵션

- `getNextPageParam`: 새로운 다음 페이지를 가져오면, 다음 페이지의 정보로 호출되는 함수. 다음 페이지 번호를 반환하고, 다음 페이지가 없으면 `undefined` 또는 `null`을 반환해야 함.
- `getPreviousePageParam`: 새로운 이전 페이지를 가져오면, 이전 페이지의 정보로 호출되는 함수. 이전 페이지 번호를 반환하고, 이전 페이지가 없으면 `undefined` 또는 `null`을 반환해야 함.
- `initialPageParam`: 첫 번째 페이지의 번호
- `maxPage`: 저장 및 출력할 최대 페이지의 수

#### 반환

- `fetchNextPage`: 다음 페이지를 가져오는 함수
- `fetchPreviousePage`: 이전 페이지를 가져오는 함수
- `hasNextPage`: 다음 페이지가 있는지 여부
- `hasPreviousePage`: 이전 페이지가 있는지 여부
- `isFetchingNextPage`: 다음 페이지를 가져오는 중인지 여부
- `isFetchingPreviousePage`: 이전 페이지를 가져오는 중인지 여부

#### 예제

```typescript
import { useInfiniteQuery } from '@tanstack/react-query'

function Todos() {
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage } =
    useInfiniteQuery({
      queryKey: ['todos'],
      queryFn: async ({ pageParam = 0 }) => {
        const response = await fetch(`/api/todos?page=${pageParam}`)
        return response.json()
      },
      initialPageParam: 0,
      getNextPageParam: (lastPage, allPages) => {
        const maxPage = Math.max(...allPages.map(page => page.page))

        if (lastPage.Response == 'True' & allPages.length < maxPage) {
          return maxPage + 1
        }

        return null
      },
    })

  return (
    <div>
      {data?.pages.map((page, i) => (
        <div key={i}>
          {page.data.map(todo => (
            <div key={todo.id}>{todo.title}</div>
          ))}
        </div>
      ))}
      <button
        onClick={() => fetchNextPage()}
        disabled={!hasNextPage || isFetchingNextPage}
      >
        {isFetchingNextPage
          ? '로딩 중...'
          : hasNextPage
            ? '더 보기'
            : '마지막 페이지'}
      </button>
    </div>
  )
}
```

무한 스크롤 버전

```typescript
import { useInfiniteQuery } from '@tanstack/react-query'
import { useEffect, useRef } from 'react'

function Todos() {
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage } =
    useInfiniteQuery({
      queryKey: ['todos'],
      queryFn: async ({ pageParam = 0 }) => {
        const response = await fetch(`/api/todos?page=${pageParam}`)
        return response.json()
      },
      initialPageParam: 0,
      getNextPageParam: (lastPage, allPages) => {
        const maxPage = Math.max(...allPages.map(page => page.page))

        if (lastPage.Response == 'True' && allPages.length < maxPage) {
          return maxPage + 1
        }

        return null
      },
    })

  const observerRef = useRef<HTMLDivElement | null>(null)

  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting && hasNextPage) {
          fetchNextPage()
        }
      },
      { threshold: 1.0 }
    )

    if (observerRef.current) {
      observer.observe(observerRef.current)
    }

    return () => observer.disconnect()
  }, [fetchNextPage, hasNextPage])

  return (
    <div>
      {data?.pages.map((page, i) => (
        <div key={i}>
          {page.data.map((todo: any) => (
            <div key={todo.id}>{todo.title}</div>
          ))}
        </div>
      ))}
      {/* 관찰 대상 요소 */}
      <div ref={observerRef} style={{ padding: '20px', textAlign: 'center' }}>
        {isFetchingNextPage
          ? '로딩 중...'
          : hasNextPage
            ? '스크롤을 내려 더 보기'
            : '마지막 페이지'}
      </div>
    </div>
  )
}
```