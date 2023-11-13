## 디렉토리 구조

```
📂 core
    📂 mocks
        📂 datas: 모킹 데이터를 보관하는 곳
        📂 handlers: MSW 핸들러가 정의된 곳
          📄 *.ts: MSW 핸들러
          📄 index.ts: 정의한 모든 핸들러를 하나의 배열로 반환하는 파일
```

## 핸들러 정의 컨벤션

### 핸들러

모킹 핸들러를 작성할 때 `delay()` 함수로 딜레이를 테스트하고, `status` 변수에 따른 분기 처리를 테스트합니다.

```ts
http.post(baseURL('/rooms'), async () => {
  await delay(1000); // 로딩 UI를 테스트할 때 원하는 수치로 수정해서 테스트합니다.

  const status: number = 201; // 성공 여부에 따른 분기를 테스트할 때 값을 바꿔서 테스트합니다.
  let response = {};

  switch (status) {
    case 201:
      response = { message: '모킹 룸 생성 완료', roomId: 1 };
      break;
    case 400:
      response = {
        message: '유효하지 않은 요청입니다.',
        validation: {
          title: 'Title Error',
          password: 'Password Error',
          type: 'Type Error',
          routine: 'Routine Error',
          certifyTime: 'CertifyTime Error',
          maxUserCount: 'MaxUserCount Error'
        }
      };
      break;
    case 401:
      response = { message: '로그인이 필요합니다.' };
      break;
  }

  return HttpResponse.json(response, { status });
})
```

## 모킹 데이터

필요에 따라서 모킹 데이터를 `@/core/mocks/datas` 에 보관합니다.

```ts
export const ALL_POST = [
  {
    userId: 1,
    id: 1,
    title: '모킹 포스트 1',
    body: '모킹된 포스트에요. MSW를 이용했어요.'
  },
  {
    userId: 1,
    id: 2,
    title: '모킹 포스트 2',
    body: '모킹된 포스트에요. MSW를 이용했어요.'
  },
  {
    userId: 1,
    id: 3,
    title: '모킹 포스트 3',
    body: '모킹된 포스트에요. MSW를 이용했어요.'
  },
  {
    userId: 1,
    id: 4,
    title: '모킹 포스트 4',
    body: '모킹된 포스트에요. MSW를 이용했어요.'
  },
  {
    userId: 1,
    id: 5,
    title: '모킹 포스트 5',
    body: '모킹된 포스트에요. MSW를 이용했어요.'
  }
];
```
