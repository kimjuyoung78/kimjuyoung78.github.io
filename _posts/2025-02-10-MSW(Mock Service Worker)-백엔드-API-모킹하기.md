---
author: 주영
date: 2025-02-10 17:10:00 +0800
last_modified_at: 2025-02-10 20:00:00 +0900
categories: [Develop]
tags: [DDang(댕댕) Project, ReactNative, MSW, 생산성]
render_with_liquid: false
image: assets/img/msw.png
---


### 시작하며

DDang 프로젝트를 RN로 마이그레이션 하는 중, 백엔드 API 작업이 예상보다 길어질 경우를 대비하여 MSW를 적용해 보았다.
<br>
무엇보다 시간 부족으로 데브코스 최종프로젝트 기간동안 적용하지 못해 공부해보고 싶었다.
<br><br>
# MSW란?

Mock Service Worker 의 줄임말로, 서비스 워커(Service Worker) 기술을 활용하여 네트워크 레벨에서 API 요청을 가로채고 모킹(mocking)할 수 있는 라이브러리이다.

- 서비스 워커 : 최신 브라우저에서 지원하는 기술로, 웹 응용 프로그램, 브라우저, 네트워크 사이의 프록시 서버 역할을 수행한다.

<img src="https://github.com/user-attachments/assets/ab82bc51-3e4e-4548-904b-46b5698c6975" alt="image" width="500">


개발 일정 중 Pending 상태와 개발 단계의 마지막에 위치하여 기한에 쫒겨 충분한 테스트를 수행하지 못하는 경우를 대비하기 위해 사용한다.
<br><br>
## MSW의 동작 원리
<img src="https://github.com/user-attachments/assets/48668e45-b72b-421f-961c-e1a38746a7ea" alt="image" width="500">


다음과 같은 과정을 통해 MSW는 API 모킹을 처리한다.

1. 브라우저에 Service Worker가 구성되어 있고, 실제 요청(Request)이 발생하면 Service Worker가 이를 가로챔
2. Service Worker는 가로챈 요청(Request)을 복사하여 MSW로 전달
3. MSW는 가로챈 요청을 미리 정의된 모의 응답(Mock Response)과 매칭
4. MSW는 모의 응답을 Service Worker에 전달
5. Service Worker는 모의 응답을 브라우저에 전달

즉, Service Worker를 중간자 역할을 하여 클라이언트 요청에 대하여 예상되는 API 응답을 네트워크 레벨에서 Mocking 하여 제공
<br><br>
## MSW 특징

1. 프레임워크와 라이브러리에 종속되지 않는다. 
    1. React, Vue 등 다양한 프레임워크에서 잘 작동한다
2. 별도 Mocking 서버를 구축할 필요 없다.
    1. 프론트엔드 프로젝트 내에서 간단하게 API Mocking을 설정할 수 있다.
3. 다양한 환경을 지원한다.
    1. 따라서 브라우저 뿐만 아니라 React Native 의 Node.js 환경에서도 동작한다.
4. 기존 코드의 수정 없이 사용 가능하다.
    1. 따라서 실제 백엔드 서버와 MSW 간의 연결 스위칭이 편리하다.

우선, 서비스워커는 브라우저 환경이 아닌 곳에서는 동작하지 않는다. React Native 프로젝트이기 때문에 노드서버 방식을 사용했다.
<br><br><br>
## 적용해보기

아래 이미지는 적용하고자 하는 코드의 전제 데이터 흐름도이다.
가족 구성원 데이터를 배열 형태로 받아와 랜더링 시킬 것이다.
<br>

<img src="https://github.com/user-attachments/assets/63159a81-735b-48a0-8881-f989ee4e3e37" alt="image" width="500">



<br><br>
### **1. Mock 데이터 정의 (handlers.js)**

해당 데이터는 내가 사용할 예정인 api 모킹 코드를 임의 더미 데이터로 써보았다.

```jsx
import { http, HttpResponse } from 'msw';

export const handlers = [
    http.get(`${BASE_URL}/family`, () => {
    return HttpResponse.json({
      data: [
        {
          memberId: 1,
          memberName: '홍길동',
          email: 'test@naver.com',
          provider: 'KAKAO',
          memberGender: 'MALE',
          memberBirthDate: '2000-01-01',
          address: '서울시 강남구',
          familyRole: 'FATHER',
          memberProfileImg: 1,
          memberWalkCount: 4,
          isRepresent: false,
        },
      ],
    });
  }),
  *// 기타 API 엔드포인트들...*
];
```

API 엔드포인트별 모의 응답을 정의한다. 그리고 각 엔드포인트마다 반환할 데이터 구조와 내용을 설정하는 부분이다.
<br>
### **2. MSW 서버 설정 (server.js)**

```jsx
import { setupServer } from 'msw/native';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

MSW 서버 인스턴스를 생성한다.

handlers에 정의된 모든 모의 응답을 서버에 등록한다. 해당 코드는 앱 실행 시 모의 서버를 시작하는 기반이 된다.
<br>
### **3. MSW 초기화 Hook (useInitializeMsw.ts)**

```jsx
export const useInitializeMsw = () => {
  const [isMswEnabled, setIsMswEnabled] = useState(false);

  useEffect(() => {
    async function enableMocking() {
      if (!__DEV__) return;
      await import('../../msw.polyfills');
      const { server } = await import('../mocks/server.js');
      server.listen({ onUnhandledRequest: 'bypass' });
      setIsMswEnabled(true);
    }
    enableMocking();
  }, []);

  return { isMswEnabled };
};
```

- 개발 환경에서만 MSW를 활성화하고 활성화 상태를 관리한다.
- MSW 서버를 초기화하고 시작한다.
<br>

### **4. API 요청 함수 (fetchFamilyInfo.tsx)**

```jsx
export const fetchFamilyInfo = async () => {
  const response = await api.get('family')
    .json<APIResponse<FetchfamilyInfoResponseType[]>>();
  return response;
};
```

- 실제 API 호출 함수와 응답 데이터의 구조를 정의한다.
- 타입 안정성을 위한 인터페이스를 정의한다.
<br>

### **5. 커스텀 Query Hook (useFamilyInfo.tsx)**

```jsx
export const useFamilyInfo = () => {
  const { data: familyInfo } = useSuspenseQuery({
    queryKey: ['familyInfo'],
    queryFn: fetchFamilyInfo,
    select: ({ data }) => data,
  });
  return familyInfo;
};
```

- React Query를 사용하여 데이터 페칭을 관리한다.
- 캐싱과 재요청 로직을 처리한다.
- 컴포넌트에서 사용하기 쉽게 데이터를 가공한다.
<br>

### **6. 데이터 사용 컴포넌트 (FamilyList.tsx)**

```jsx
export const FamilyList = () => {
  const familyMembers = useFamilyInfo();
  return (
    <S.GapBox>
      {familyMembers.map((item) => (
        <S.ProfileWrapper key={item.memberId}>
          <Profile size={74} avatarNumber={item.memberProfileImg} />
          {*/* 프로필 정보 표시 */*}
        </S.ProfileWrapper>
      ))}
    </S.GapBox>
  );
};
```

- useFamilyInfo hook을 통해 가족 정보를 가져온다.
- 가족 구성원 목록을 화면에 렌더링한다.
<br>

이러한 구조로 개발 환경에서 MSW를 통해 API를 모킹하고, 실제 서비스와 동일한 방식으로 데이터를 사용해 보았다.
<br>
<br>

<img src="https://github.com/user-attachments/assets/a50f5d04-cf9b-4902-8b70-ff54d0cbfc8f" alt="image" width="300">


<br>

적용된 모습! mocking한 데이터들이 잘 들어온다.
<br><br><br>

**참고 출처** 

https://tech.ktcloud.com/263 <br>
https://velog.io/@hyo123/MSW-Mock-Service-Worker
