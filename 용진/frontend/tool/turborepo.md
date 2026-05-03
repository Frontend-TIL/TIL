Turborepo의 공식문서를 기반으로 정리한 글입니다.

## Turborepo란?

`Tuborepo` 는 JavaScript 및 TypeScript 코드베이스용 **고성능 빌드 시스템**이다. 모노레포를 확장하도록 설계되었으며 단일 패키지 워크스페이스의 워크플로우도 더 빠르게 만들어준다.

저장소에서 실행하는 데 필요한 작업을 최적화하는 경량 접근 방식을 통해 수년간의 엔지니어링 시간과 수백만 달러의 컴퓨팅 비용을 절약하고 있다.

### 모노레포 문제

모노레포는 **코드 공유**와 **의존성 관리**에 있어 많은 장점을 제공하지만, **규모가 커질수록 극심한 속도 저하**를 겪게 된다.

- 모노레포 내부의 각 패키지는 자체 테스트, 린트, 빌드 프로세스를 가지고 있다.
- 거대한 모노레포에서는 한 번의 PR이나 배포를 위해 수천 개의 작업이 실행되어야 할 수도 있다.
- 이렇게 느려진 피드백 루프는 개발자 경험(DX)을 해치고, 고품질 코드를 제공하는 것을 방해한다.

### 해결책

`Turborepo` 는 모노레포의 확장성 문제를 다음과 같이 해결한다.

- **원격 캐싱**: 모든 작업의 결과를 캐시에 저장한다. 누군가 이미 한 번 실행했던 작업이라면, **절대 똑같은 작업을 두 번 반복하지 않는다**.
- **작업 스케줄링**: 모노레포에서는 A를 먼저 빌드하고, B를 테스트하고, C를 린트하는 것처럼 복잡한 실행 순서가 존재한다. Turborepo는 이런 의존성을 파악한 뒤, **사용 가능한 모든 CPU 코어에서 작업을 최대한 병렬로 실행**한다. 이를 통해 속도를 향상 시킬 수 있다.
- **점진적 도입**: Turborepo는 점진적으로 도입할 수 있으며, 몇 분 만에 모든 저장소에 추가할 수 있다. 이미 작성해 둔 `package.json` 의 스크립트, 종속성을 그대로 사용하며, 오직 `turbo.json` 파일 하나만 추가하면 된다. npm 생태계의 표준을 따르기 때문에, `npm` , `yarn` , `pnpm` 등 어떤 패키지 매니저를 사용하든 완벽하게 동작한다.

## Turborepo 시작하기 with Next.js

### 빠른 시작 (스타터 레포지토리)

```bash
# pnpm
pnpm dlx create-turbo@latest

# yarn
yarn dlx create-turbo@latest

# npm
npx create-turbo@latest

# bun
bunx create-turbo@latest
```

이 명령어를 통해 생성된 스타터 레포지토리는 다음을 포함한다.

- 2개의 배포가능한 애플리케이션
- 모노레포 내의 다른 곳에서 가져다 쓸 수 있는 공유 라이브러리 3개

### Turbo 설치하기 (전역 및 로컬 설치 권장)

1. **전역 설치**

```bash
# pnpm
pnpm add turbo --global

# yarn
yarn global add turbo

# npm
npm install turbo --global

# bun
bun install turbo --global
```

전역 설치 후에는 저장소 내 어디서든 `turbo`  명령어를 사용하여 스크립트를 빠르게 실행할 수 있다.(ex: `turbo build` , `turbo generate` , 의존성 그래프를 무시하고 특정 패키지만 빌드 구조를 확인하는 `turbo build --filter=docs --dry`  등)

1. **저장소 설치**

```bash
# pnpm
pnpm add turbo --save-dev --ignore-workspace-root-check

# yarn
yarn add turbo --dev --ignore-workspace-root-check

# npm
npm install turbo --save-dev
```