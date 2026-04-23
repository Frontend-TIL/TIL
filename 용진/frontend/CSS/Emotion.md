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

## Styled Components (`@emotion/styled`)

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

## 글로벌 스타일

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

## 애니메이션 (keyframes)

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