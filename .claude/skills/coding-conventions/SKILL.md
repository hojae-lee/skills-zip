---
name: coding-conventions
description: 기본 코딩 컨벤션 스킬. 코드 작성, 리뷰, 리팩토링 시 아래 규칙들을 반드시 적용한다. "코드 작성해줘", "리뷰해줘", "리팩토링해줘", "컨벤션 확인해줘", "코드 개선해줘", "이 코드 괜찮아?", "코드 짜줘" 등 코드 관련 요청이 오면 무조건 이 스킬을 사용한다.
---

# 기본 코딩 컨벤션

## 규칙 목록

### 1. Arrow function

`function` 키워드 대신 화살표 함수를 쓴다.

```ts
// ✅
const greet = (name: string) => `Hello, ${name}`;

// ❌
function greet(name: string) { return `Hello, ${name}`; }
```

---

### 2. Early return

조건이 맞지 않으면 일찍 반환해서 중첩을 줄인다.

```ts
// ✅
const getUser = (id: string) => {
  if (!id) return null;
  if (!isValid(id)) return null;
  return fetchUser(id);
};

// ❌
const getUser = (id: string) => {
  if (id) {
    if (isValid(id)) {
      return fetchUser(id);
    }
  }
  return null;
};
```

---

### 3. 하드코딩 금지

문자열/URL/경로 리터럴은 상수나 환경변수로 뺀다.

```ts
// ✅
const API_BASE_URL = process.env.NEXT_PUBLIC_API_URL;
const fetchData = () => fetch(`${API_BASE_URL}/users`);

// ❌
const fetchData = () => fetch("https://api.example.com/users");
```

---

### 4. const/let만, var 금지

재할당이 없으면 `const`, 있으면 `let`. `var`는 쓰지 않는다.

```ts
// ✅
const MAX_RETRIES = 3;
let count = 0;

// ❌
var MAX_RETRIES = 3;
```

---

### 5. 의미있는 이름

이름만 봐도 뭔지 알 수 있어야 한다. `a`, `temp`, `data2`, `foo` 금지.

```ts
// ✅
const userList = await getUsers();
const isLoggedIn = !!session?.user;

// ❌
const d = await getUsers();
const flag = !!session?.user;
```

---

### 6. DRY — 중복 제거

같은 로직이 두 번 이상 나오면 함수로 추출한다.

```ts
// ✅
const formatDate = (date: Date) => date.toLocaleDateString("ko-KR");
const createdAt = formatDate(user.createdAt);
const updatedAt = formatDate(user.updatedAt);

// ❌
const createdAt = user.createdAt.toLocaleDateString("ko-KR");
const updatedAt = user.updatedAt.toLocaleDateString("ko-KR");
```

---

### 7. 매직 넘버 금지

의미 없이 떠있는 숫자는 이름 있는 상수로 만든다.

```ts
// ✅
const SESSION_TIMEOUT_MS = 30 * 60 * 1000;
const MAX_FILE_SIZE_MB = 10;

// ❌
setTimeout(logout, 1800000);
if (file.size > 10485760) { ... }
```

---

### 8. async/await 우선

Promise 체인 대신 async/await을 쓴다. 가독성이 훨씬 낫다.

```ts
// ✅
const loadUser = async (id: string) => {
  const user = await fetchUser(id);
  const posts = await fetchPosts(user.id);
  return { user, posts };
};

// ❌
const loadUser = (id: string) =>
  fetchUser(id).then(user =>
    fetchPosts(user.id).then(posts => ({ user, posts }))
  );
```

---

### 9. 함수는 작게

함수 하나는 한 가지 일만. 스크롤 없이 읽힐 정도면 충분하다.

```ts
// ✅
const validateEmail = (email: string) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
const validatePassword = (pw: string) => pw.length >= 8;
const validateForm = (email: string, pw: string) =>
  validateEmail(email) && validatePassword(pw);

// ❌
const validateForm = (email: string, pw: string) => {
  const emailOk = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  const pwOk = pw.length >= 8;
  // ... 점점 길어지는 함수
  return emailOk && pwOk;
};
```
