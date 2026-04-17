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

