# React에서 Props와 State 이해하기

리액트에서 컴포넌트를 설계할 때 가장 중요한 개념은 props와 state이다.
두 개념 모두 UI를 구성하는 데이터지만, 역할과 책임이 명확하게 나뉘어 있다.

## Props란 무엇인가

props는 부모 컴포넌트가 자식 컴포넌트에 전달하는 데이터이다.
즉, 컴포넌트 외부에서 주어지는 입력값이다.

```js
function Parent() {
  return <Child name="Jack" />;
}

function Child(props) {
  return <div>{props.name}</div>;
}
```

위 코드에서 Child는 name이라는 값을 전달받아 화면에 출력만 한다.

### 핵심 특징

- 부모 → 자식으로 전달된다 (단방향)
- 읽기 전용이다 (immutable)
- 자식은 props를 직접 수정할 수 없다
- 컴포넌트를 재사용 가능하게 만든다


## 왜 Props는 수정할 수 없을까?

리액트는 **단방향 데이터 흐름(one-way data flow)** 을 따른다.

즉, 데이터는 항상 부모에서 자식으로만 흐른다.

이 구조를 유지하기 위해 props는 읽기 전용으로 설계되어 있다.

이렇게 설계된 이유

1. 데이터 흐름이 명확해진다
- 값이 어디서 바뀌는지 추적하기 쉽다
2. 디버깅이 쉬워진다
- 상태 변경의 출발점이 부모로 고정된다
3. 컴포넌트가 독립적으로 동작한다
- 입력만 받아서 렌더링하는 구조 → 재사용성 증가


잘못된 사용 예시

```js
function Child(props) {
  props.name = "New Name"; // ❌ 잘못된 코드
  return <div>{props.name}</div>;
}
```
props는 부모가 내려준 값이기 때문에
자식에서 직접 수정하면 리액트의 설계 원칙을 깨게 된다.


## State란 무엇인가

`state`는 컴포넌트 내부에서 관리하는 데이터이다.
시간에 따라 변할 수 있는 값들을 저장한다.

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>
        증가
      </button>
    </div>
  );
}
```

### 핵심 특징
- 컴포넌트 내부에서 관리된다
- 값이 변경될 수 있다
- state가 변경되면 컴포넌트가 다시 렌더링된다


### Props vs State 한 번에 정리
| 구분    | Props      | State       |
| ----- | ---------- | ----------- |
| 정의    | 부모가 전달하는 값 | 내부에서 관리하는 값 |
| 변경 여부 | ❌ 변경 불가    | ✅ 변경 가능     |
| 관리 위치 | 부모         | 해당 컴포넌트     |
| 역할    | 데이터 전달     | UI 상태 관리    |


## 자식이 값을 바꿔야 할 때는?

자식 컴포넌트는 props를 직접 수정할 수 없다.
대신 부모의 state를 변경하는 함수를 전달받아 사용한다.

-> 이걸 상태 끌어올리기 (Lifting State Up) 라고 한다.

```jsx
function Parent() {
  const [name, setName] = useState("Jack");

  return <Child name={name} changeName={setName} />;
}

function Child({ name, changeName }) {
  return (
    <div>
      <p>{name}</p>
      <button onClick={() => changeName("New Name")}>
        변경
      </button>
    </div>
  );
}
```

핵심은
> 데이터는 부모가 가지고 있고
> 자식은 변경 요청만 한다는 점이다.

한 줄 정리
props: 외부에서 주어지는 읽기 전용 데이터
state: 내부에서 관리하며 변경 가능한 데이터
