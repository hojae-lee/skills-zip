# TanStack Query 패턴

## 기본 원칙

- 컴포넌트에서 `useQuery`/`useMutation` 직접 호출 금지 → 도메인별 커스텀 훅으로 래핑
- 컴포넌트에서 API 함수 직접 호출 금지 → TanStack Query를 통해 처리
- Mutation 후 관련 쿼리 캐시를 `invalidateQueries`로 무효화

---

## Query Factory 패턴

쿼리 키와 옵션을 한 곳에서 관리한다. `queryOptions()`로 타입 추론을 활용한다.

```ts
// queries/postQueries.ts
import { queryOptions } from "@tanstack/react-query";
import { getPost, listPosts } from "../api/postApi";
import type { ListPostsParams } from "../types";

export const postQueries = {
  allKey: () => ["post"] as const,

  getPost: (postId: string) =>
    queryOptions({
      queryKey: [...postQueries.allKey(), "get-post", postId],
      queryFn: () => getPost(postId),
      enabled: !!postId,
      staleTime: 5 * 60 * 1000,
    }),

  listPosts: (params?: ListPostsParams) =>
    queryOptions({
      queryKey: [...postQueries.allKey(), "list-posts", params],
      queryFn: () => listPosts(params),
      staleTime: 5 * 60 * 1000,
    }),
} as const;
```

---

## 커스텀 훅 패턴

### Query 훅

```ts
// hooks/useGetPost.ts
import { useQuery } from "@tanstack/react-query";
import { postQueries } from "../queries/postQueries";

export const useGetPost = (postId: string) =>
  useQuery(postQueries.getPost(postId));

// hooks/useListPosts.ts
export const useListPosts = (params?: ListPostsParams) =>
  useQuery(postQueries.listPosts(params));
```

### Mutation 훅

```ts
// hooks/useCreatePost.ts
import { useMutation, useQueryClient } from "@tanstack/react-query";
import { createPost } from "../api/postApi";
import { postQueries } from "../queries/postQueries";
import type { CreatePostRequest } from "../types";

export const useCreatePost = () => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data: CreatePostRequest) => createPost(data),
    onSuccess: () =>
      queryClient.invalidateQueries({ queryKey: postQueries.allKey() }),
  });
};

// hooks/useUpdatePost.ts
export const useUpdatePost = (postId: string) => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data: UpdatePostRequest) => updatePost(postId, data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: postQueries.allKey() });
    },
  });
};
```

---

## Query Key 네이밍 규칙

```ts
// 기본 도메인 키
allKey: () => ["domain"];

// 단일 조회
queryKey: [...allKey(), "get-entity", id];

// 목록 조회
queryKey: [...allKey(), "list-entities", params];

// 검색
queryKey: [...allKey(), "search-entities", searchParams];
```

key는 계층 구조를 이룬다. `allKey()`를 무효화하면 해당 도메인의 모든 쿼리가 무효화된다.

---

## 캐시 무효화 패턴

```ts
// 생성 후: 목록 전체 무효화
queryClient.invalidateQueries({ queryKey: postQueries.allKey() });

// 수정 후: 특정 항목 + 목록 개별 무효화
queryClient.invalidateQueries({ queryKey: postQueries.getPost(id).queryKey });
queryClient.invalidateQueries({ queryKey: postQueries.listPosts().queryKey });

// 삭제 후: 특정 항목 제거 + 목록 무효화
queryClient.removeQueries({ queryKey: postQueries.getPost(id).queryKey });
queryClient.invalidateQueries({
  queryKey: [...postQueries.allKey(), "list-posts"],
});
```

---

## Enabled 조건부 쿼리

```ts
export const useGetPost = (postId: string | null) =>
  useQuery({
    ...postQueries.getPost(postId ?? ""),
    enabled: !!postId, // postId가 있을 때만 실행
  });
```

---

## Optimistic Update

```ts
export const useToggleLike = (postId: string) => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: () => toggleLike(postId),
    onMutate: async () => {
      await queryClient.cancelQueries({
        queryKey: postQueries.getPost(postId).queryKey,
      });
      const previous = queryClient.getQueryData(
        postQueries.getPost(postId).queryKey,
      );
      queryClient.setQueryData(
        postQueries.getPost(postId).queryKey,
        (old: Post) => ({
          ...old,
          liked: !old.liked,
        }),
      );
      return { previous };
    },
    onError: (_, __, context) => {
      queryClient.setQueryData(
        postQueries.getPost(postId).queryKey,
        context?.previous,
      );
    },
    onSettled: () => {
      queryClient.invalidateQueries({
        queryKey: postQueries.getPost(postId).queryKey,
      });
    },
  });
};
```

---

## 컴포넌트에서 사용

```tsx
const PostDetail = ({ postId }: { postId: string }) => {
  const { data, isPending, isError } = useGetPost(postId);
  const { mutate: createComment } = useCreateComment(postId);

  if (isPending) return <Spinner />;
  if (isError) return <ErrorMessage />;

  return (
    <div>
      <h1>{data.title}</h1>
      <button onClick={() => createComment({ text: "comment" })}>
        댓글 추가
      </button>
    </div>
  );
};
```
