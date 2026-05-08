---
name: lint-prettier-setup
description: 새 프로젝트 또는 새 패키지/앱을 추가할 때 ESLint와 Prettier 설정을 이 프로젝트의 표준에 맞게 세팅하는 스킬. 단일 프로젝트와 monorepo 모두 지원한다. "lint 설정해줘", "eslint 추가해줘", "prettier 설정", "새 패키지 설정", "프로젝트 초기 세팅", "새 앱 추가", "코드 품질 설정", "pre-commit 설정", "husky 설정" 등의 요청 시 반드시 사용한다. 신규 앱이나 패키지 생성 시에도 항상 적용한다.
---

# Lint & Prettier 설정 가이드

## 프로젝트 타입 확인

세팅을 시작하기 전에 프로젝트 구조를 먼저 파악한다.

- **단일 프로젝트**: `package.json`이 루트 하나뿐. ESLint/Prettier를 직접 설치하고 설정 파일을 루트에 둔다.
- **Monorepo (Turborepo/pnpm workspaces)**: 여러 앱/패키지가 있고 `packages/` 디렉토리가 있다. 공유 config 패키지(`packages/eslint-config`, `packages/prettier-config`)를 두고 각 앱/패키지가 참조한다.

---

## 공통: Prettier 설정값

프로젝트 타입과 무관하게 아래 값을 기준으로 한다.

```js
/** @type {import("prettier").Config} */
const config = {
  printWidth: 80,
  tabWidth: 2,
  useTabs: false,
  semi: false,
  singleQuote: true,
  trailingComma: "none",
  bracketSpacing: true,
  jsxSingleQuote: false,
  arrowParens: "always",
  endOfLine: "lf",
  quoteProps: "as-needed",
  htmlWhitespaceSensitivity: "css",
  proseWrap: "preserve",
};

export default config;
```

---

## 공통: ESLint 핵심 규칙

환경에 상관없이 아래 규칙을 반드시 적용한다.

| 규칙                                  | 설정                                                            | 이유                    |
| ------------------------------------- | --------------------------------------------------------------- | ----------------------- |
| `react/function-component-definition` | arrow-function만 허용                                           | 코드 스타일 일관성      |
| `@typescript-eslint/no-explicit-any`  | error                                                           | 타입 안전성             |
| `unused-imports/no-unused-imports`    | error                                                           | 불필요한 코드 제거      |
| `import/order`                        | builtin → external → internal → parent → sibling → index → type | 가독성                  |
| `import/newline-after-import`         | error                                                           | 가독성                  |
| `import/no-duplicates`                | error                                                           | 중복 방지               |
| `react/react-in-jsx-scope`            | off                                                             | React 17+ JSX transform |
| `prefer-arrow-callback`               | error                                                           | 스타일 일관성           |

**path alias (`@/**`, `@common/**` 등)가 있을 때만** `import/order`에 `pathGroups` 추가:

```js
pathGroups: [
  { pattern: '@/**', group: 'internal', position: 'after' },
  // 프로젝트에서 사용하는 alias만 추가
],
pathGroupsExcludedImportTypes: ['builtin'],
```

---

## A. 단일 프로젝트 설정

### 패키지 설치

```bash
pnpm add -D eslint prettier \
  @eslint/js typescript-eslint eslint-config-prettier \
  eslint-plugin-react eslint-plugin-react-hooks \
  eslint-plugin-unused-imports eslint-plugin-import \
  eslint-import-resolver-typescript \
  @tanstack/eslint-plugin-query
```

Next.js 앱이라면 추가:

```bash
pnpm add -D @next/eslint-plugin-next
```

### `eslint.config.js` (React/Next.js)

```js
import js from "@eslint/js";
import eslintConfigPrettier from "eslint-config-prettier";
import tseslint from "typescript-eslint";
import unusedImports from "eslint-plugin-unused-imports";
import react from "eslint-plugin-react";
import reactHooks from "eslint-plugin-react-hooks";
import tanstackQuery from "@tanstack/eslint-plugin-query";
import importPlugin from "eslint-plugin-import";
// Next.js라면:
// import pluginNext from '@next/eslint-plugin-next'

export default [
  {
    ignores: [
      "dist/**",
      "build/**",
      "node_modules/**",
      "**/*.stories.ts",
      "**/*.stories.tsx",
      "coverage/**",
      "*.config.js",
      "*.config.mjs",
      "**/*.js",
      // Next.js라면:
      // '.next/**'
    ],
  },

  js.configs.recommended,
  ...tseslint.configs.recommended,
  react.configs.flat.recommended,
  react.configs.flat["jsx-runtime"],
  eslintConfigPrettier,
  ...tanstackQuery.configs["flat/recommended"],

  // Next.js라면:
  // {
  //   plugins: { '@next/next': pluginNext },
  //   rules: {
  //     ...pluginNext.configs.recommended.rules,
  //     ...pluginNext.configs['core-web-vitals'].rules
  //   }
  // },

  {
    files: ["**/*.{ts,tsx}"],
    plugins: {
      "unused-imports": unusedImports,
      react,
      "react-hooks": reactHooks,
      import: importPlugin,
    },
    settings: {
      react: { version: "detect" },
      "import/resolver": {
        typescript: { alwaysTryTypes: true, project: "./tsconfig.json" },
        node: { extensions: [".js", ".jsx", ".ts", ".tsx"] },
      },
      "import/parsers": { "@typescript-eslint/parser": [".ts", ".tsx"] },
    },
    languageOptions: {
      ecmaVersion: "latest",
      sourceType: "module",
      parserOptions: {
        ecmaFeatures: { jsx: true },
        project: true,
        tsconfigRootDir: import.meta.dirname,
      },
    },
    rules: {
      ...reactHooks.configs.recommended.rules,
      "react/react-in-jsx-scope": "off",
      "react/jsx-uses-react": "off",
      "arrow-body-style": "off",
      "prefer-arrow-callback": "error",
      "react/function-component-definition": [
        "error",
        {
          namedComponents: "arrow-function",
          unnamedComponents: "arrow-function",
        },
      ],
      "unused-imports/no-unused-imports": "error",
      "@typescript-eslint/no-unused-vars": "off",
      "unused-imports/no-unused-vars": [
        "warn",
        {
          vars: "all",
          varsIgnorePattern: "^_",
          args: "after-used",
          argsIgnorePattern: "^_",
        },
      ],
      "@typescript-eslint/no-explicit-any": "error",
      "@typescript-eslint/no-unsafe-function-type": "off",
      "@typescript-eslint/no-empty-object-type": "off",
      "no-undef": "off",
      "react/prop-types": "off",
      "react/display-name": "off",
      "import/order": [
        "error",
        {
          groups: [
            "builtin",
            "external",
            "internal",
            "parent",
            "sibling",
            "index",
            "type",
          ],
          // path alias가 있으면 pathGroups 추가
          "newlines-between": "always",
          alphabetize: { order: "asc", caseInsensitive: true },
        },
      ],
      "import/first": "error",
      "import/newline-after-import": "error",
      "import/no-duplicates": "error",
    },
  },
];
```

### `prettier.config.js`

위 공통 설정값을 그대로 사용한다.

### `package.json` 스크립트

```json
{
  "scripts": {
    "lint": "eslint --max-warnings 0",
    "lint:fix": "eslint --fix",
    "format": "prettier --write \"src/**/*.{ts,tsx,md}\""
  }
}
```

---

## B. Monorepo 설정

공유 config 패키지를 `packages/`에 두고, 각 앱/패키지가 `workspace:*`로 참조한다.

### `packages/prettier-config/index.js`

위 공통 설정값 그대로 사용. `package.json`:

```json
{
  "name": "@repo/prettier-config",
  "version": "0.0.0",
  "type": "module",
  "private": true,
  "main": "index.js",
  "exports": { ".": "./index.js" }
}
```

### `packages/eslint-config/`

세 가지 config를 exports로 나눠 제공한다.

**`package.json`:**

```json
{
  "name": "@repo/eslint-config",
  "version": "0.0.0",
  "type": "module",
  "private": true,
  "exports": {
    "./base": "./base.js",
    "./react": "./react.js",
    "./next-js": "./next.js"
  },
  "devDependencies": {
    "@eslint/js": "^9.39.1",
    "@next/eslint-plugin-next": "^16.2.0",
    "@tanstack/eslint-plugin-query": "^5.66.1",
    "eslint": "^9.39.1",
    "eslint-config-prettier": "^10.1.1",
    "eslint-import-resolver-typescript": "^3.10.1",
    "eslint-plugin-import": "^2.31.0",
    "eslint-plugin-only-warn": "^1.1.0",
    "eslint-plugin-react": "^7.37.5",
    "eslint-plugin-react-hooks": "^5.2.0",
    "eslint-plugin-turbo": "^2.7.1",
    "eslint-plugin-unused-imports": "^4.1.4",
    "globals": "^16.5.0",
    "typescript": "^5.9.2",
    "typescript-eslint": "^8.50.0"
  }
}
```

**`base.js`** — Node.js/TypeScript 패키지 전용:

```js
import js from "@eslint/js";
import eslintConfigPrettier from "eslint-config-prettier";
import turboPlugin from "eslint-plugin-turbo";
import tseslint from "typescript-eslint";
import onlyWarn from "eslint-plugin-only-warn";

/** @type {import("eslint").Linter.Config[]} */
export const config = [
  js.configs.recommended,
  eslintConfigPrettier,
  ...tseslint.configs.recommended,
  {
    plugins: { turbo: turboPlugin },
    rules: { "turbo/no-undeclared-env-vars": "warn" },
  },
  { plugins: { onlyWarn } },
  { ignores: ["dist/**"] },
];
```

**`react.js`** — React 라이브러리 패키지용 (`packages/ui`, `packages/utils` 등):

단일 프로젝트의 `eslint.config.js`와 동일한 규칙을 `createReactConfig(dirname)` 팩토리 함수로 export한다. `tsconfigRootDir`에 `dirname`을 넘겨야 하기 때문에 팩토리 패턴을 사용한다.

```js
// (단일 프로젝트 eslint.config.js 내용과 동일하게 작성하되)
export const createReactConfig = (dirname) => [
  /* ... */
];
```

**`next.js`** — Next.js 앱용, `react.js` 확장:

```js
import pluginNext from "@next/eslint-plugin-next";
import { createReactConfig } from "./react.js";

export const createNextJsConfig = (dirname) => [
  ...createReactConfig(dirname),
  { ignores: [".next/**"] },
  {
    plugins: { "@next/next": pluginNext },
    rules: {
      ...pluginNext.configs.recommended.rules,
      ...pluginNext.configs["core-web-vitals"].rules,
    },
  },
];
```

### 각 앱/패키지에서 참조

```js
// apps/*/eslint.config.js (Next.js 앱)
import { createNextJsConfig } from "@repo/eslint-config/next-js";
export default createNextJsConfig(import.meta.dirname);
```

```js
// packages/ui/eslint.config.mjs (React 라이브러리)
import { createReactConfig } from "@repo/eslint-config/react";
export default createReactConfig(import.meta.dirname);
```

```js
// packages/api/eslint.config.js (일반 TS 패키지)
import { config } from "@repo/eslint-config/base";
export default config;
```

각 앱/패키지 `package.json`:

```json
{
  "scripts": {
    "lint": "eslint --max-warnings 0",
    "format": "prettier --write \"src/**/*.{ts,tsx}\""
  },
  "prettier": "@repo/prettier-config",
  "devDependencies": {
    "@repo/eslint-config": "workspace:*",
    "@repo/prettier-config": "workspace:*",
    "eslint": "^9.39.1",
    "prettier": "^3.7.4"
  }
}
```

루트 `package.json` 스크립트 (Turborepo 기준):

```json
{
  "scripts": {
    "lint": "turbo run lint",
    "lint:fix": "turbo run lint -- --fix",
    "format": "prettier --write \"**/*.{ts,tsx,md}\""
  }
}
```

---

## Husky pre-commit 훅 (선택)

커밋 전 format → lint fix → stage를 자동 실행한다.

```sh
# .husky/pre-commit
pnpm format
pnpm lint:fix
git add -A
```

mise 같은 버전 매니저를 쓴다면 PATH 추가:

```sh
export PATH="$HOME/.local/share/mise/shims:$PATH"
```

설치:

```bash
pnpm add -D husky -w   # monorepo라면 -w, 단일 프로젝트라면 생략
pnpm exec husky init
# .husky/pre-commit 파일을 위 내용으로 교체
```

루트 `package.json`에 추가:

```json
{ "scripts": { "prepare": "husky" } }
```

---

## 새 패키지/앱 추가 체크리스트

1. 프로젝트 타입 확인 (단일 / monorepo)
2. `eslint.config.js` 생성 — 패키지 성격에 맞는 config 선택
3. `package.json`에 `"prettier"` 참조 추가 (monorepo: `"@repo/prettier-config"`, 단일: `"./prettier.config.js"` 또는 패키지 이름)
4. `devDependencies`에 필요한 패키지 추가 후 설치
5. `scripts`에 `lint`, `format` 추가
