## 디렉토리 구조

```
📂 core
    📂 api
        📂 functions: API 요청하는 axios 함수들
        📂 mutations: 공통화가 필요한 커스텀 useMutation 훅
        📂 queries: 공통화가 필요한 커스텀 useQuery & useQueries 훅
        📂 options: 기본적으로 매핑할 queryKey, queryFn, 그 외 옵션 정보
        📄 instance.ts: axios 인스턴스 객체
```

## API 로직 레이어

### 1. axios 인스턴스

모든 요청에 대한 공통 옵션 및 인터셉터 로직이 들어간 레이어입니다.

```ts
// 일반적으로 사용하는 axios 인스턴스
export const baseInstance: CustomAxiosInstance = axios.create({ baseURL });

// multipart/form-data 를 위한 axios 인스턴스
export const formDataInstance: CustomAxiosInstance = axios.create({
  baseURL,
  headers: { 'Content-Type': 'multipart/form-data' }
});
```

### 2. API 요청 함수 (functions)

axios 인스턴스를 가지고 실제로 API 요청을 하는 함수가 들어간 레이어입니다.  
적절한 요청과 응답 타입을 부여합니다.

#### 이름 컨벤션
`메소드 + 내용` 으로 함수 이름을 정의합니다.  
ex) `getDetailPost`

```ts
const examplePostAPI = {
  getAllPosts: async () => {
    return await baseInstance.get<Post[]>(`/example/posts`);
  },
  getDetailPost: async (postId: string) => {
    return await baseInstance.get<Post>(`/example/posts/${postId}`);
  },
  postPost: async (post: Post) => {
    return await baseInstance.post<{ message: string }>('/example/posts', post);
  },
  putPost: async (post: Post) => {
    return await baseInstance.put<{ message: string }>(
      `/example/posts/${post.id}`,
      post
    );
  },
  deletePost: async (postId: string) => {
    return await baseInstance.delete<{ message: string }>(
      `/example/posts/${postId}`
    );
  }
};

export default examplePostAPI;
```

### 3-1. queryOptions 매핑 (options)

`TanStack Query` 에서 사용하는 쿼리 키와 쿼리 함수의 매핑 정보를 표현하는 레이어입니다.  
(컴포넌트에서 사용하는 useQuery, useQueries 에서 활용할 수 있습니다.)

```ts
export const examplePostOptions = {
  all: () =>
    queryOptions({
      queryKey: ['groups', 'all'] as const,
      queryFn: postAPI.getAllPosts
    }),
  detail: (postId: string) =>
    queryOptions({
      queryKey: ['groups', 'detail', postId] as const,
      queryFn: () => postAPI.getDetailPost(postId)
    })
};

export default examplePostOptions;
```

### 3-2. 커스텀 훅 (queries, mutations)

`TanStack Query` 의 `useQuery` 나 `useMutation` 훅을 래핑해야 하는 경우에 사용하는 레이어입니다.  

#### 이름 컨벤션
- `queries`: 접미사에 `Query` 혹은 `Queries` 를 붙혀서 사용합니다.  
ex) `usePostQuery`
- `mutations`: `mutationFn` 의 이름 앞에 `use`를 붙혀서 사용합니다.  
ex) `usePutPost`

### 4. 사용 예시

```tsx
const App = () => {
  const { data: postList } = useQuery({ ...queryKeys.posts.all }); // 매핑된 queryKey 사용
  const { data: postDetail } = useQuery({ ...queryKeys.posts.detail('5') }); // 매핑된 queryKey 사용

  const { mutate: addPost } = useMutation({ mutationFn: postAPI.addPost }); // 정의된 API 함수 사용
  const { mutate: modifyPost } = useModifyPost(); // 커스텀 훅 사용

  const handleAddPost = () => {
    addPost({ id: 1, title: 'title', body: 'body', userId: 1 });
  };

  const handleModifyPost = () => {
    modifyPost({ id: 1, title: 'title', body: 'body', userId: 1 });
  };

  return (
    <div>
      <button onClick={handleAddPost}>게시글 추가 요청</button>
      <button onClick={handleModifyPost}>게시글 수정 요청</button>
    </div>
  );
}
```
