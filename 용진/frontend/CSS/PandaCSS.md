해당 글은 Panda CSS 1.10 버전을 기반으로 정리한 내용입니다.

## Panda CSS 시작하기 with React

### Panda CSS란?

Panda CSS는 **빌드 타임에 동작하는 Zero-runtime CSS-in-JS** 라이브러리이다.

기존 Styled-components나 Emotion이 브라우저 런타임에 스타일을 계산하여 성능 저하를 일으킬 수 있는 반면, Panda CSS는 빌드 단계에서 정적인 CSS 파일을 생성한다. 또한 완벽한 타입 추론을 제공하여 오타로 인한 스타일 에러를 사전에 방지할 수 있다.

### 시작

1. 패키지 설치 및 초기화

```tsx
npm install -D @pandacss/dev postcss
npm panda init -p
```

이 명령어를 실행하면 `panda.config.ts` 파일과 `postcss.config.cjs` 파일이 생성되며, 스타일 유틸리티가 포함된 `styled-system` 폴더가 만들어진다.

1. `panda.config.ts` 구성

Panda 엔진이 스캔할 파일 경로를 지정한다.

```tsx
import { defineConfig } from "@pandacss/dev"

export default defineConfig({
  // CSS Reset을 사용할지 여부
  preflight: true,
  
  // Panda가 스타일을 찾을 파일 확장자 및 경로
  include: ["./src/**/*.{js,jsx,ts,tsx}", "./app/**/*.{js,jsx,ts,tsx}"],

  // 제외할 경로
  exclude: [],

  // 커스텀 테마 (디자인 토큰)
  theme: {
    extend: {}
  },

  // 결과물이 생성될 폴더 이름
  outdir: "styled-system",
})
```

1. 글로벌 CSS에 연결

```tsx
@layer reset, base, tokens, recipes, utilities;
```

### 핵심 문법

Pands CSS는 `styled-system` 폴더에서 함수를 불러와서 사용한다.

1. `css` 함수

```tsx
import { css } from '../styled-system/css'

function Button() {
  return (
    <button className={css({ 
      bg: 'blue.500', 
      color: 'white', 
      px: '4', 
      py: '3', 
      borderRadius: 'md',
      _hover: { bg: 'blue.600' } 
    })}>
      클릭하세요
    </button>
  )
}
```

1. 패턴: 레이아웃을 쉽게 잡기 위한 함수들

```tsx
import { flex, grid } from '../styled-system/patterns'

function Layout() {
  return (
    <div className={flex({ direction: 'column', align: 'center', gap: '4' })}>
      <div className={grid({ columns: 2, gap: '2' })}>
        <div>아이템 1</div>
        <div>아이템 2</div>
      </div>
    </div>
  )
}
```

1. 디자인 토큰: `panda.config.ts` 에서 정의한 색상, 폰트 사이즈 등을 코드에서 참조

```tsx
theme: {
  extend: {
    tokens: {
      colors: {
        primary: { value: '#0F52BA' }
      }
    }
  }
}

// 컴포넌트에서의 사용
const className = css({ color: 'primary' })
```

### Recipe

#### cva

Recipe의 `cva` 함수를 통해 다양한 상태를 가진 컴포넌트를 정의할 수 있다.

cva는 단일 요소의 Recipe를 정의하는 데 좋다.

`defineRecipe` 을 통해 `panda.config.ts` 에 전역 Recipe을 등록하고 컴포넌트에서 이를 활용할 수 있다.

- `base`: 컴포넌트의 기본 스타일
- `variants` : 컴포넌트의 상태 별 스타일 정의
- `compoundVariants` : 복합 변형 스타일 정의
- `defaultVariants` : 기본 상태

```tsx
// button.recipe.ts
import { defineRecipe } from '@pandacss/dev'
 
export const buttonRecipe = defineRecipe({
  className: 'button',
  description: 'The styles for the Button component',
  base: {
    display: 'flex'
  },
  variants: {
    visual: {
      funky: { bg: 'red.200', color: 'white' },
      edgy: { border: '1px solid {colors.red.500}' }
    },
    size: {
      sm: { padding: '4', fontSize: '12px' },
      lg: { padding: '8', fontSize: '40px' }
    },
    shape: {
      square: { borderRadius: '0' },
      circle: { borderRadius: 'full' }
    }
  },
  defaultVariants: {
    visual: 'funky',
    size: 'sm',
    shape: 'circle'
  }
})
```

```tsx
// panda.config.ts
import { defineConfig } from '@pandacss/dev'
import { buttonRecipe } from './button.recipe'
 
export default defineConfig({
  //...
  jsxFramework: 'react',
  theme: {
    extend: {
      recipes: {
        button: buttonRecipe
      }
    }
  }
})
```

```tsx
// recipe 사용
import { button } from '../styled-system/recipes'
 
function App() {
  return (
    <div>
      <button className={button()}>Click me</button>
      <button className={button({ shape: 'circle' })}>Click me</button>
    </div>
  )
}
```

#### sva

`cva` 가 단일 요소를 위한 기능이었다면, `sva` 는 여러 요소로 구성된 복합 컴포넌트를 위한 기능이다. **여러 개의 하위 요소들의 스타일과 상태를 한 곳에서 관리**할 수 있게 해준다.

만약 `Card` 컴포넌트를 만든다고 가정할 때, `Card` 컴포넌트는 보통`컨테이너` , `헤더` , `본문` , `푸터` 로 구성되어 있다.

- cva 방식: 카드 사이즈가 `lg` 로 변할 때마다 컨테이너 패딩, 헤더 폰트 크기, 본문 여백 등을 각각의 하위 요소 Recipe에서 따로 제어해야 한다. 즉, 상태 동기화가 매우 까다롭다.
- sva 방식: 최상단에서 `size: 'lg'` 라는 상태 하나만 주입하면, 정의해둔 `root` , `header` , `body` 의 스타일이 일제히 사이즈에 맞게 변형된다.

```tsx
// checkbox.recipe.ts
import { sva } from '../styled-system/css'
 
const cardRecipe = sva({
  // 1. 컴포넌트를 구성하는 슬롯(하위 요소)들을 정의합니다.
  slots: ['root', 'header', 'body', 'footer'],

  // 2. 각 슬롯의 기본 스타일을 정의합니다.
  base: {
    root: { display: 'flex', flexDirection: 'column', borderWidth: '1px', borderRadius: 'md' },
    header: { padding: '4', borderBottomWidth: '1px', fontWeight: 'bold' },
    body: { padding: '4', flex: '1' },
    footer: { padding: '4', bg: 'gray.50' }
  },

  // 3. 상태(Variant)에 따라 슬롯들의 스타일이 어떻게 변할지 정의합니다.
  variants: {
    size: {
      sm: {
        root: { fontSize: 'sm' },
        header: { padding: '2' },
        body: { padding: '2' },
        footer: { padding: '2' }
      },
      lg: {
        root: { fontSize: 'lg' },
        header: { padding: '6' },
        body: { padding: '6' },
        footer: { padding: '6' }
      }
    },
    // 카드 형태 변형
    visual: {
      elevated: { root: { boxShadow: 'md', borderWidth: '0' } },
      outline: { root: { borderWidth: '1px' } }
    }
  },
  
  defaultVariants: {
    size: 'sm',
    visual: 'outline'
  }
})
```

```tsx
// Card.tsx
import { cardRecipe } from '../styled-system/recipes'
import type { ComponentProps } from 'react'

// 레시피의 Variant 타입을 추출합니다. (예: { size?: 'sm' | 'lg', visual?: 'elevated' | 'outline' })
type CardVariants = Parameters<typeof cardRecipe>[0]

interface CardProps extends CardVariants {
  title: string
  content: string
  footerText?: string
}

export const Card = ({ title, content, footerText, size, visual }: CardProps) => {
  // 1. 선택된 variant를 넣어 sva 함수를 실행합니다.
  const classes = cardRecipe({ size, visual })

  return (
    // 2. 반환된 classes 객체에서 해당 슬롯의 이름을 꺼내 className에 주입합니다.
    <div className={classes.root}>
      <div className={classes.header}>
        {title}
      </div>
      <div className={classes.body}>
        {content}
      </div>
      {footerText && (
        <div className={classes.footer}>
          {footerText}
        </div>
      )}
    </div>
  )
}
```

```tsx
// Card 사용: 크기는 크고, 그림자가 있는 카드 생성 (내부의 header, body 패딩이 모두 자동으로 lg 사이즈에 맞춰짐)
<Card 
  size="lg" 
  visual="elevated" 
  title="공지사항" 
  content="Panda CSS Slot Recipe 업데이트 안내" 
/>
```