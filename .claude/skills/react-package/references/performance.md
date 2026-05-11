# React 성능 최적화 규칙

Vercel Engineering이 정리한 65개 규칙, 8개 카테고리. 우선순위 순으로 적용한다.

## 카테고리별 우선순위

| 우선순위 | 카테고리               | 영향        |
| -------- | ---------------------- | ----------- |
| 1        | Waterfalls 제거        | CRITICAL    |
| 2        | Bundle 최적화          | CRITICAL    |
| 3        | Server-Side 성능       | HIGH        |
| 4        | 클라이언트 데이터 페칭 | MEDIUM-HIGH |
| 5        | Re-render 최적화       | MEDIUM      |
| 6        | Rendering 성능         | MEDIUM      |
| 7        | JavaScript 성능        | LOW-MEDIUM  |
| 8        | Advanced 패턴          | LOW         |

---

## 1. Waterfalls 제거 (CRITICAL)

### async-defer-await — 실제로 필요한 시점까지 await 미루기

```ts
// ❌ 순차 실행
const user = await getUser(id);
const posts = await getPosts(id);

// ✅ 병렬 실행
const [user, posts] = await Promise.all([getUser(id), getPosts(id)]);
```

### async-parallel — 독립적인 작업은 Promise.all()

의존 관계가 없는 비동기 작업은 항상 병렬 실행.

### async-dependencies — 부분 의존성은 better-all 패턴

A가 완료된 후 B, C를 병렬 시작:

```ts
const a = await fetchA();
const [b, c] = await Promise.all([fetchB(a.id), fetchC(a.type)]);
```

### async-api-routes — API route에서 Promise 조기 시작

```ts
// ✅ 함수 시작 시점에 Promise 시작, 나중에 await
const userPromise = getUser(id);
const settingsPromise = getSettings(id);
// ... 다른 동기 작업 ...
const [user, settings] = await Promise.all([userPromise, settingsPromise]);
```

### async-suspense-boundaries — Suspense로 콘텐츠 스트리밍

독립적인 데이터를 각각의 Suspense boundary로 감싸면 병렬 스트리밍.

---

## 2. Bundle 최적화 (CRITICAL)

### bundle-barrel-imports — barrel 파일 피하고 직접 import

```ts
// ❌ barrel import (전체 모듈 로드)
import { Button, Input, Modal } from "@common/components";

// ✅ 직접 import
import { Button } from "@common/components/Button";
```

### bundle-dynamic-imports — 무거운 컴포넌트는 dynamic import

```tsx
const Chart = React.lazy(() => import("./Chart"));
// 또는 Next.js: const Chart = dynamic(() => import('./Chart'))
```

### bundle-defer-third-party — 분석/로깅 라이브러리는 hydration 후 로드

```ts
useEffect(() => {
  import("analytics").then(({ init }) => init());
}, []);
```

### bundle-conditional — 기능 활성화 시점에만 모듈 로드

```ts
const handleFeature = async () => {
  const { heavyFeature } = await import("./heavyFeature");
  heavyFeature();
};
```

### bundle-preload — hover/focus 시 preload로 체감 속도 개선

```tsx
const handleMouseEnter = () => {
  import("./HeavyModal"); // preload only
};
```

---

## 3. Server-Side 성능 (HIGH)

> Next.js/RSC 환경에 해당하는 규칙들. 클라이언트 전용 앱은 참고용.

- `server-cache-react` — `React.cache()`로 요청 단위 중복 제거
- `server-cache-lru` — 요청 간 공유 데이터는 LRU 캐시
- `server-hoist-static-io` — 정적 자원(폰트, 로고)은 모듈 레벨에서 로드
- `server-parallel-fetching` — 컴포넌트 구조로 병렬 fetch 유도
- `server-serialization` — 클라이언트에 전달하는 데이터 최소화

---

## 4. 클라이언트 데이터 페칭 (MEDIUM-HIGH)

### client-swr-dedup — SWR/TanStack Query로 자동 중복 제거

여러 컴포넌트가 같은 데이터를 요청할 때 라이브러리가 자동으로 dedup.

### client-event-listeners — 전역 이벤트 리스너 중복 방지

```ts
useEffect(() => {
  const handler = () => { ... }
  window.addEventListener('resize', handler)
  return () => window.removeEventListener('resize', handler)
}, [])
```

### client-passive-event-listeners — 스크롤 리스너는 passive

```ts
window.addEventListener("scroll", handler, { passive: true });
```

### client-localstorage-schema — localStorage 데이터에 버전 포함

```ts
const data = { version: 1, value: ... }
localStorage.setItem('key', JSON.stringify(data))
```

---

## 5. Re-render 최적화 (MEDIUM)

### rerender-no-inline-components — 컴포넌트를 컴포넌트 안에 정의하지 않기

```tsx
// ❌ 리렌더링마다 새 참조 생성
const Parent = () => {
  const Child = () => <div>...</div>; // 금지
  return <Child />;
};

// ✅ 분리 선언
const Child = () => <div>...</div>;
const Parent = () => <Child />;
```

### rerender-derived-state-no-effect — effect 없이 렌더링 중 derived state 계산

```tsx
// ❌
const [doubled, setDoubled] = useState(0);
useEffect(() => {
  setDoubled(count * 2);
}, [count]);

// ✅
const doubled = count * 2; // 렌더링 중 계산
```

### rerender-defer-reads — 콜백에서만 쓰는 state는 ref로

```tsx
// ❌ handleClick이 value를 읽어도 value 변경마다 리렌더
const [value, setValue] = useState(0);
const handleClick = useCallback(() => console.log(value), [value]);

// ✅
const valueRef = useRef(0);
const handleClick = useCallback(() => console.log(valueRef.current), []);
```

### rerender-memo — 비용 높은 작업은 메모이제이션

React Compiler 없는 환경에서: 렌더링 비용이 높은 순수 컴포넌트에 `memo` 적용.

### rerender-functional-setstate — setState에 이전 값 의존 시 함수형

```ts
// ❌
setCount(count + 1);

// ✅
setCount((prev) => prev + 1);
```

### rerender-lazy-state-init — 비용 높은 초기값은 함수로 전달

```ts
// ❌
const [data, setData] = useState(expensiveCompute());

// ✅
const [data, setData] = useState(() => expensiveCompute());
```

### rerender-transitions — 긴급하지 않은 업데이트는 startTransition

```ts
startTransition(() => {
  setFilteredItems(heavyFilter(items, query));
});
```

### rerender-use-deferred-value — 입력 반응성 유지하며 비싼 렌더 지연

```tsx
const deferredQuery = useDeferredValue(query);
const results = useMemo(() => heavySearch(deferredQuery), [deferredQuery]);
```

### rerender-use-ref-transient-values — 빠르게 변하는 값은 ref

애니메이션 frame 값, 마우스 좌표 등 렌더링에 불필요한 변화는 ref로 관리.

### rerender-split-combined-hooks — 독립적 의존성을 가진 훅은 분리

```ts
// ❌ 하나의 훅에서 관계없는 두 state를 계산
const { a, b } = useCombined(x, y);

// ✅ 각각 독립 훅
const a = useA(x);
const b = useB(y);
```

### rerender-move-effect-to-event — effect 대신 이벤트 핸들러에 로직

```ts
// ❌ 클릭 후 effect에서 처리
useEffect(() => {
  if (clicked) doSomething();
}, [clicked]);

// ✅ 클릭 핸들러에서 직접 처리
const handleClick = () => {
  doSomething();
};
```

---

## 6. Rendering 성능 (MEDIUM)

### rendering-hoist-jsx — 변하지 않는 JSX는 컴포넌트 외부로

```tsx
// ❌
const MyComponent = () => {
  const icon = <svg>...</svg>; // 매 렌더마다 새 객체
  return <div>{icon}</div>;
};

// ✅
const ICON = <svg>...</svg>;
const MyComponent = () => <div>{ICON}</div>;
```

### rendering-conditional-render — && 대신 삼항 연산자

```tsx
// ❌ count가 0일 때 "0" 렌더
{
  count && <Component />;
}

// ✅
{
  count > 0 ? <Component /> : null;
}
```

### rendering-content-visibility — 긴 목록에 content-visibility CSS

```css
.list-item {
  content-visibility: auto;
  contain-intrinsic-size: 0 80px; /* 예상 높이 */
}
```

### rendering-usetransition-loading — 수동 loading state 대신 useTransition

```tsx
const [isPending, startTransition] = useTransition();

const handleClick = () => {
  startTransition(() => {
    navigate("/heavy-page");
  });
};
```

### rendering-animate-svg-wrapper — SVG 대신 wrapper div 애니메이션

SVG 요소 직접 애니메이션은 성능 비용이 크다. wrapper div를 대신 사용.

---

## 7. JavaScript 성능 (LOW-MEDIUM)

### js-index-maps — 반복 조회는 Map으로 O(1)

```ts
// ❌ O(n) 반복
const find = (id: string) => items.find((item) => item.id === id);

// ✅ O(1) Map
const itemMap = new Map(items.map((item) => [item.id, item]));
const find = (id: string) => itemMap.get(id);
```

### js-set-map-lookups — includes/indexOf 대신 Set

```ts
const allowedRoles = new Set(['admin', 'editor'])
if (allowedRoles.has(user.role)) { ... }
```

### js-early-exit — 조건 불충족 시 일찍 반환

```ts
const process = (items: Item[]) => {
  if (!items.length) return [];
  // ... 실제 처리
};
```

### js-combine-iterations — filter + map은 reduce 또는 flatMap으로

```ts
// ❌ 두 번 순회
const result = items.filter(isValid).map(transform);

// ✅ 한 번 순회
const result = items.flatMap((item) =>
  isValid(item) ? [transform(item)] : [],
);
```

### js-hoist-regexp — RegExp는 루프 밖에서 생성

```ts
// ❌ 루프마다 RegExp 생성
items.filter((item) => /pattern/.test(item.name));

// ✅ 한 번만 생성
const PATTERN = /pattern/;
items.filter((item) => PATTERN.test(item.name));
```

### js-cache-function-results — 반복 호출되는 순수 함수는 캐시

```ts
const cache = new Map<string, Result>();
const expensiveOp = (key: string): Result => {
  if (cache.has(key)) return cache.get(key)!;
  const result = compute(key);
  cache.set(key, result);
  return result;
};
```

### js-request-idle-callback — 비중요 작업은 idle 시간에

```ts
requestIdleCallback(() => {
  logAnalytics(event);
});
```

### js-tosorted-immutable — 불변 정렬은 toSorted()

```ts
// ❌ 원본 배열 변경
const sorted = items.sort((a, b) => a.name.localeCompare(b.name));

// ✅ 원본 유지
const sorted = items.toSorted((a, b) => a.name.localeCompare(b.name));
```

---

## 8. Advanced 패턴 (LOW)

### advanced-init-once — 앱 시작 시 한 번만 초기화

```ts
let initialized = false;
const initApp = () => {
  if (initialized) return;
  initialized = true;
  // 초기화 로직
};
```

### advanced-event-handler-refs — 이벤트 핸들러는 ref에 저장

```ts
const handlerRef = useRef(handler);
handlerRef.current = handler;

useEffect(() => {
  const fn = (e: Event) => handlerRef.current(e);
  window.addEventListener("event", fn);
  return () => window.removeEventListener("event", fn);
}, []); // 의존성 없음
```

### advanced-use-latest — 최신 콜백 안정적으로 참조

```ts
const useLatest = <T>(value: T) => {
  const ref = useRef(value);
  ref.current = value;
  return ref;
};
```
