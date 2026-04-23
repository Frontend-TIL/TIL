해당 글은 **Emotion 공식문서 11버전**을 기반으로 정리한 내용입니다.

## 1. Emotion 시작하기 with React

React 환경에서는 주로 `@emotion/react` 와 `@emotion/styled` 를 함께 사용한다.

```bash
npm install @emotion/react @emotion/styled
```

## 2. css prop (`@emotion/react`)

`css prop` 은 Emotion의 가장 기본적인 스타일링 도구이다. JSX 안에 직접 스타일을 적을 수 있어서 인라인 스타일처럼 간편하지만, 실제로는 CSS 클래스로 변환되기 때문에 **가상 클래스, 미디어 쿼리, 스타일 조합, 재사용, 우선순위 관리** 등 CSS의 강력한 기능을 사용할 수 있다.

### 객체 스타일

JavaScript 객체 형태로 스타일을 작성한다.

```tsx
import { css } from '@emotion/react'

const containerStyle = css({
  display: 'flex',
  justifyContent: 'center',
  alignItems: 'center',
  backgroundColor: '#f4f4f9',
  padding: '20px',
  borderRadius: '8px',
  '&:hover': {
    backgroundColor: '#e2e2ea'
  }
})

export default function ProfileCard() {
  return <div css={containerStyle}>프로필 영역</div>
}
```

### 문자열 스타일

기존 순수 CSS 문법에 익숙하다면 백틱을 이용해 문자열로 작성할 수도 있다.

```tsx
import { css } from '@emotion/react'

const titleStyle = css`
  color: #333;
  font-size: 24px;
  font-weight: bold;
  
  @media (max-width: 768px) {
    font-size: 18px;
  }
`

export default function Title() {
  return <h1 css={titleStyle}>제목</h1>
}
```

## 3. Styled Components (`@emotion/styled`)

스타일이 적용된 컴포넌트 자체를 생성하는 방식이다. 관심사를 분리하고, 마크업을 읽기 쉽게 만드는데 도움을 준다. 컴포넌트의 `props` 를 받아 동저긍로 스타일을 변경할 때 유용하다.

```tsx
import styled from '@emotion/styled'

// Props 타입 정의
interface ButtonProps {
  primary?: boolean;
}

// Styled Component 생성
const Button = styled.button<ButtonProps>`
  padding: 10px 20px;
  border-radius: 4px;
  border: none;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.2s ease-in-out;
  
  /* Props에 따른 동적 스타일링 */
  background-color: ${(props) => (props.primary ? '#0070f3' : '#eaeaea')};
  color: ${(props) => (props.primary ? 'white' : '#333')};

  &:hover {
    background-color: ${(props) => (props.primary ? '#005bb5' : '#cccccc')};
  }
`

export default function ActionArea() {
  return (
    <div>
      <Button>일반 버튼</Button>
      <Button primary>주요 버튼</Button>
    </div>
  )
}
```

다른 Emotion 컴포넌트를 타겟으로 지정하여 스타일을 지정할 수 있다,

```tsx
// 객체 스타일
import styled from '@emotion/styled'

const Child = styled.div({
  color: 'red'
})

const Parent = styled.div({
  [Child]: {
    color: 'green'
  }
})

render(
  <div>
    <Parent>
      <Child>Green because I am inside a Parent</Child>
    </Parent>
    <Child>Red because I am not inside a Parent</Child>
  </div>
)
```

```tsx
// 문자열 스타일
import styled from '@emotion/styled'

const Child = styled.div`
  color: red;
`

const Parent = styled.div`
  ${Child} {
    color: green;
  }
`

render(
  <div>
    <Parent>
      <Child>Green because I am inside a Parent</Child>
    </Parent>
    <Child>Red because I am not inside a Parent</Child>
  </div>
)
```

## 4. 글로벌 스타일

애플리케이션 전체에 적용되는 `reset CSS` 나 폰트 설정 등은 `<Global>`  컴포넌트를 사용하여 주입해야 한다.

```tsx
import { Global, css } from '@emotion/react'

const globalStyles = css`
  body {
    margin: 0;
    padding: 0;
    font-family: 'Pretendard', sans-serif;
    box-sizing: border-box;
  }
  
  * {
    box-sizing: inherit;
  }
`

export default function App() {
  return (
    <>
      <Global styles={globalStyles} />
      <main>앱 콘텐츠</main>
    </>
  )
}
```

## 5. 애니메이션 (keyframes)

`keyframes` 헬퍼를 통해 애니메이션을 쉽게 구현할 수 있다.

```tsx
/** @jsxImportSource @emotion/react */
import { css, keyframes } from '@emotion/react'

const bounce = keyframes`
  from, 20%, 53%, 80%, to {
    transform: translate3d(0,0,0);
  }
  40%, 43% {
    transform: translate3d(0, -30px, 0);
  }
  70% {
    transform: translate3d(0, -15px, 0);
  }
  90% {
    transform: translate3d(0, -4px, 0);
  }
`

const bounceStyle = css`
  animation: ${bounce} 1s ease infinite;
`

export default function LoadingIcon() {
  return <div css={bounceStyle}>⏳</div>
}
```

## 6. Best Practice

Emotion을 사용할 때 성능을 최적화하고 유지보수성을 높이기 위한 권장 사항은 다음과 같다.

- **정적 스타일 추출**: 컴포넌트 렌더링 시마다 스타일이 다시 계산되는 것을 방지하기 위해, `props`나 상태(state)에 의존하지 않는 정적인 스타일은 컴포넌트 외부로 분리하는 것이 좋다.
```tsx
// Bad: 렌더링될 때마다 새로운 객체가 생성됨
function Component() {
  return <div css={{ color: 'red' }} />
}

// Good: 외부에서 한 번만 생성
const redStyle = css({ color: 'red' });
function Component() {
  return <div css={redStyle} />
}
```

- **Object Styles 활용**: 문자열 방식보다 객체 방식을 사용하면 **TypeScript의 자동 완성 지원**을 받을 수 있고, **린팅 및 타입 검사**에서 유리하다.
- **과도한 동적 스타일링 주의**: 스타일 내부에 너무 많은 삼항 연산자나 조건문을 넣기보다는, 상황에 따라 스타일 객체 자체를 분리하거나 컴포넌트를 나누는 것이 가독성과 성능에 좋다.

## 7. 테마 설정(Theming)

애플리케이션 전체에 일관된 디자인 시스템을 적용할 수 있도록 `<ThemeProvider>`를 제공한다.

### 테마 정의(`ThemeProvider`)
```tsx
import { ThemeProvider } from '@emotion/react'

// 1. 테마 객체 정의
const theme = {
  colors: {
    primary: '#0070f3',
    background: '#f4f4f9',
  },
}

// 2. 최상단 앱에 ThemeProvider 감싸기
export default function App({ children }) {
  return (
    <ThemeProvider theme={theme}>
      {children}
    </ThemeProvider>
  )
}
```

### 테마 사용

`styled` 컴포넌트나 `css` prop 내부에서 콜백 함수를 통해 테마에 접근할 수 있다.

```tsx
import styled from '@emotion/styled'
import { css, useTheme } from '@emotion/react'

// styled 컴포넌트에서 사용
const Button = styled.button`
  background-color: ${props => props.theme.colors.primary};
  color: white;
`

// css prop에서 사용
function Header() {
  return (
    <header css={theme => ({ backgroundColor: theme.colors.background })}>
      헤더 영역
    </header>
  )
}

// useTheme 훅 사용
function MyButton() {
  const theme = useTheme()
  return <button css={{ backgroundColor: theme.colors.primary }}>버튼</button>
}
```

