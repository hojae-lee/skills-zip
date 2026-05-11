---
name: turborepo-workspace
description: Use this skill when working in a Turborepo + pnpm monorepo — setting up the workspace, adding packages, configuring shared configs (ui, eslint, prettier, tsconfig, css, utils), managing turbo.json pipelines, writing per-app commands, setting up GitHub Actions CI/CD, or writing Dockerfiles. Trigger on requests like "모노레포 설정해줘", "새 패키지 추가", "packages/ui에 컴포넌트 추가", "공통 설정 만들어줘", "CI 설정해줘", "github actions", "도커파일 만들어줘", "배포 설정", "@repo/ui", "pnpm workspace", or any work involving the monorepo root or packages/ directory. 앱 내부 코드는 nextjs-app-router 스킬을 함께 사용한다.
---

# Turborepo + pnpm Monorepo

## 기술 스택

- **Turborepo** — 태스크 오케스트레이션, 캐싱
- **pnpm workspaces** — 패키지 관리

## 전체 구조

```text
apps/
└── <app-name>/                   # Next.js 앱 (내부 구조는 nextjs-app-router 스킬 참고)

packages/
├── ui/                           # 아토믹 UI 컴포넌트 (@repo/ui)
├── utils/                        # 공통 유틸 함수 (@repo/utils)
├── styles/                       # 공통 CSS 변수·토큰 (@repo/styles)
├── tsconfig/                     # 공유 TypeScript 설정 (@repo/tsconfig)
├── eslint-config/                # 공유 ESLint 설정 (@repo/eslint-config)
└── prettier-config/              # 공유 Prettier 설정 (@repo/prettier-config)

turbo.json
pnpm-workspace.yaml
package.json
.github/
└── workflows/
    └── ci.yml
```

## packages/ 구성

### packages/ui — 아토믹 UI 컴포넌트

비즈니스 로직 없는 순수 UI 컴포넌트만 둔다. 앱별 비즈니스 로직이 필요한 경우 앱 내에서 래핑한다.

```text
packages/ui/
├── src/
│   ├── components/
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.test.tsx
│   │   │   └── index.ts
│   │   └── index.ts              # 전체 export
│   └── index.ts
├── package.json
└── tsconfig.json
```

```json
// packages/ui/package.json
{
  "name": "@repo/ui",
  "exports": {
    ".": "./src/index.ts"
  }
}
```

### packages/utils — 공통 유틸

앱 간 공유되는 순수 함수, 상수, 타입 유틸리티.

```text
packages/utils/
├── src/
│   ├── format/
│   ├── validation/
│   └── index.ts
└── package.json                  # name: "@repo/utils"
```

### packages/styles — 공통 CSS

디자인 토큰(CSS 변수), 전역 reset, 공통 폰트 등 앱 간 공유되는 스타일.

```text
packages/styles/
├── src/
│   ├── tokens.css                # CSS 변수 (색상, 간격, 타이포그래피)
│   ├── reset.css
│   └── index.css                 # 전체 import
└── package.json                  # name: "@repo/styles"
```

### packages/tsconfig — 공유 TypeScript 설정

```json
// packages/tsconfig/base.json
{
  "compilerOptions": {
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "moduleResolution": "bundler",
    "jsx": "preserve",
    "target": "ES2022"
  }
}
```

각 앱·패키지의 `tsconfig.json`에서 extends로 참조한다.

```json
// apps/<app-name>/tsconfig.json
{
  "extends": "@repo/tsconfig/base.json",
  "include": ["src", "next-env.d.ts"],
  "compilerOptions": {
    "paths": { "@/*": ["./src/*"] }
  }
}
```

### packages/eslint-config — 공유 ESLint 설정

```text
packages/eslint-config/
├── base.js      # Node.js/TypeScript 패키지용
├── react.js     # React 라이브러리용
├── next.js      # Next.js 앱용
└── package.json # name: "@repo/eslint-config"
```

각 앱·패키지에서 참조:

```js
// apps/<app-name>/eslint.config.js
import nextConfig from '@repo/eslint-config/next'
export default nextConfig
```

### packages/prettier-config — 공유 Prettier 설정

```js
// packages/prettier-config/index.js
export default {
  printWidth: 80,
  tabWidth: 2,
  useTabs: false,
  semi: false,
  singleQuote: true,
  trailingComma: 'none',
  bracketSpacing: true,
  arrowParens: 'always',
  endOfLine: 'lf'
}
```

```json
// 각 앱/패키지 package.json
{ "prettier": "@repo/prettier-config" }
```

## Import 규칙

```ts
import { Button } from '@repo/ui'
import { formatDate } from '@repo/utils'
import { ... } from '@/common/...'          // 앱 내 공통
import { ... } from './components/...'      // 도메인 내부 (상대 경로)
```

다른 앱의 코드를 직접 import 금지 — 공유가 필요하면 `packages/`로 올린다.

## turbo.json 파이프라인

```json
{
  "$schema": "https://turbo.build/schema.json",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "dist/**"]
    },
    "lint": {
      "dependsOn": ["^build"]
    },
    "test": {
      "dependsOn": ["^build"],
      "outputs": ["coverage/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

## 명령어

### 전체 실행

```bash
pnpm build          # 전체 빌드 (의존성 순서 자동 처리)
pnpm lint           # 전체 lint
pnpm test           # 전체 테스트
pnpm format         # 전체 포맷
```

### 특정 앱/패키지만

```bash
pnpm --filter <app-name> dev
pnpm --filter <app-name> build
pnpm --filter <app-name> lint
pnpm --filter <app-name> test
pnpm --filter <app-name> test:watch
```

### Turbo 직접 실행

```bash
turbo run build --filter=<app-name>
turbo run lint test --filter=<app-name>
turbo run build --filter=<app-name>...   # 해당 앱과 의존 패키지 포함
```

## GitHub Actions CI

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: 9

      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Lint
        run: pnpm lint

      - name: Test
        run: pnpm test

      - name: Build
        run: pnpm build
```

앱별로 분리가 필요한 경우 `--filter` 옵션으로 job을 분리한다.

```yaml
jobs:
  build-app:
    runs-on: ubuntu-latest
    steps:
      - ...
      - name: Build
        run: pnpm --filter <app-name> build
```

## Dockerfile

Turborepo의 `turbo prune`을 활용해 특정 앱에 필요한 파일만 추출 후 빌드한다.

```dockerfile
# Dockerfile (apps/<app-name>/ 또는 루트에 위치)
FROM node:22-alpine AS base

# ── prune ──────────────────────────────────────────
FROM base AS pruner
RUN corepack enable && corepack prepare pnpm@latest --activate
WORKDIR /app
COPY . .
RUN pnpm dlx turbo prune <app-name> --docker

# ── install ────────────────────────────────────────
FROM base AS installer
RUN corepack enable && corepack prepare pnpm@latest --activate
WORKDIR /app

COPY --from=pruner /app/out/json/ .
RUN pnpm install --frozen-lockfile

COPY --from=pruner /app/out/full/ .
RUN pnpm --filter <app-name> build

# ── runner ─────────────────────────────────────────
FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production

RUN addgroup --system --gid 1001 nodejs \
 && adduser  --system --uid 1001 nextjs

COPY --from=installer --chown=nextjs:nodejs /app/apps/<app-name>/.next/standalone ./
COPY --from=installer --chown=nextjs:nodejs /app/apps/<app-name>/.next/static ./apps/<app-name>/.next/static
COPY --from=installer --chown=nextjs:nodejs /app/apps/<app-name>/public ./apps/<app-name>/public

USER nextjs
EXPOSE 3000
CMD ["node", "apps/<app-name>/server.js"]
```

`standalone` 출력을 사용하려면 `next.config.ts`에 추가한다.

```ts
// apps/<app-name>/next.config.ts
const nextConfig = {
  output: 'standalone'
}
```

### Docker 빌드 & 실행

```bash
# 모노레포 루트에서 실행
docker build -t <app-name> -f apps/<app-name>/Dockerfile .
docker run -p 3000:3000 <app-name>
```
