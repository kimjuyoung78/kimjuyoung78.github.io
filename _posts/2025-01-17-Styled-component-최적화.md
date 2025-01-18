---
author: 주영
date: 2025-01-17 17:10:00 +0800
categories: [Develop]
tags: [DDang(댕댕), 효율화]
render_with_liquid: false
---
DDang 프로젝트 개발 중 소셜로그인 파트를 맡아 구현하고 있었다. 

<img src="https://github.com/user-attachments/assets/aba14019-2c59-4fb8-bd54-a4f55f94e1cd" alt="image" width="400">



LoginPage 컴포넌트 짜는 과정에서 불필요한 코드가 너무 많이 중복되었다. 특히 소셜로그인 3가지 디자인이 똑같기 때문에 더 그렇게 느꼈던 것 같다.

### 기존 style.ts 코드

해당 주제에 필요한 코드만 넣었다.

```jsx
import { FontWeight, styled } from 'styled-components'
import Kakao_Icon from '~assets/kakao_icon.svg?react'
import Naver_Icon from '~assets/naver_icon.svg?react'
import Google_Icon from '~assets/google_icon.svg?react'
import { FOOTER_HEIGHT } from '~constants/layout'

//소셜로그인
export const SocialLoginSection = styled.div`
  width: 100%;
  max-width: 350px;
  height: 200px;
  margin-top: 50px;
`
export const Kakao = styled.div<{ weight: FontWeight }>`
  width: 335px;
  height: 52px;
  /* flex-shrink: 0; */
  border-radius: 12px;
  background: #ffed16;

  color: ${({ theme }) => theme.colors.grayscale.font_1};
  font-size: ${({ theme }) => theme.typography._14};
  font-weight: ${({ weight }) => weight};

  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  margin: 10px 6px;
  cursor: pointer;
`
export const KakaoIcon = styled(Kakao_Icon)`
  width: 24px;
  height: 24px;
  position: absolute;
  left: 24px;
`
export const Naver = styled.div<{ weight: FontWeight }>`
  width: 335px;
  height: 52px;
  /* flex-shrink: 0; */
  border-radius: 12px;
  background: #03cf5d;

  color: ${({ theme }) => theme.colors.grayscale.gc_4};
  font-size: ${({ theme }) => theme.typography._14};
  font-weight: ${({ weight }) => weight};

  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  margin: 10px 6px;
  cursor: pointer;
`
export const NaverIcon = styled(Naver_Icon)`
  width: 24px;
  height: 24px;
  position: absolute;
  left: 24px;
`
export const Google = styled.div<{ weight: FontWeight }>`
  width: 335px;
  height: 52px;
  /* flex-shrink: 0; */
  border-radius: 12px;
  background: #f2f2f2;

  color: ${({ theme }) => theme.colors.grayscale.font_1};
  font-size: ${({ theme }) => theme.typography._14};
  font-weight: ${({ weight }) => weight};

  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  margin: 10px 6px;
  cursor: pointer;
`
export const GoogleIcon = styled(Google_Icon)`
  width: 24px;
  height: 24px;
  position: absolute;
  left: 24px;
`

```
딱 봐도 똑같은 코드가 반복되어 비효율적인 코드라는게 느껴졌다ㅠㅠ 
그래서 겹치는 공통 스타일을 **베이스 컴포넌트로 분리** 하였다!

### 개선된 코드 :

```jsx
// 추가: 공통 스타일을 베이스 컴포넌트로 분리
const SocialButtonBase = styled.div<{ weight: FontWeight }>`
  width: calc(100% - 12px);  // 변경: 동적 너비 계산 (부모 크기 - 좌우 마진)
  height: 52px;
  border-radius: 12px;
  font-size: ${({ theme }) => theme.typography._14};
  font-weight: ${({ weight }) => weight};
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  margin: 10px 6px;
  cursor: pointer;
`

// 기존: width: 335px로 고정
// 변경: SocialButtonBase 상속받아 사용
export const Kakao = styled(SocialButtonBase)`
  background: #ffed16;
  color: ${({ theme }) => theme.colors.grayscale.font_1};
`

// 기존: width: 335px로 고정
// 변경: SocialButtonBase 상속받아 사용
export const Naver = styled(SocialButtonBase)`
  background: #03cf5d;
  color: ${({ theme }) => theme.colors.grayscale.gc_4};
`

// 기존: width: 335px로 고정
// 변경: SocialButtonBase 상속받아 사용
export const Google = styled(SocialButtonBase)`
  background: #f2f2f2;
  color: ${({ theme }) => theme.colors.grayscale.font_1};
`

// Icon 컴포넌트들은 변경 없음
```

## 2차 Refactoring

2차 리팩토링에서는 코드의 **가독성과 효율성**을 더욱 높이기 위해 다음과 같은 개선 작업을 추가적으로 진행했다.

- 기존에는 각 소셜 로그인 버튼 컴포넌트가 개별적으로 정의되어 중복된 코드가 많았다. 이를 공통 스타일인 **`SocialButtonBase`**로 추출하여 중복을 제거하고, 각 버튼의 고유 속성만 분리해 가독성을 더 높혔다.
- 반복되는 버튼 생성 로직을 **`map`** 메서드로 처리하였다.
- 또, 버튼 색상과 레이아웃 관련 값을 상수로 분리하여 유지보수성을 강화했다. (예를 들어, 색상 변경이 필요한 경우 상수만 수정하면 된다!)

**index.tsx**

```jsx
import * as S from './styles'
import { Helmet } from 'react-helmet-async'

const SOCIAL_LOGIN_BUTTONS = [
  { Component: S.Kakao, Icon: S.KakaoIcon, text: '카카오계정 로그인' },
  { Component: S.Naver, Icon: S.NaverIcon, text: '네이버로 로그인' },
  { Component: S.Google, Icon: S.GoogleIcon, text: '구글로 로그인' },
] as const

const TitleSection = () => (
  <S.TitleSection>
    건강한 반려 생활{'\n'}
    <S.BrandText>댕</S.BrandText>과 함께해요!
  </S.TitleSection>
)

const SocialLoginButtons = (): JSX.Element => {
  return (
    <S.SocialLoginSection>
      {SOCIAL_LOGIN_BUTTONS.map(({ Component, Icon, text }) => (
        <Component key={text} weight='700'>
          <Icon />
          {text}
        </Component>
      ))}
    </S.SocialLoginSection>
  )
}

export default function LoginPage() {
  return (
    <S.LoginPageContainer>
      <Helmet>
        <title>DDang | 로그인</title>
        <meta name='description' content='DDang 서비스 로그인' />
      </Helmet>
      <TitleSection />
      <S.Logo>로고</S.Logo>
      <SocialLoginButtons />
    </S.LoginPageContainer>
  )
}

```

**style.ts**

```jsx
import { FontWeight, styled } from 'styled-components'
import Kakao_Icon from '~assets/kakao_icon.svg?react'
import Naver_Icon from '~assets/naver_icon.svg?react'
import Google_Icon from '~assets/google_icon.svg?react'
import { FOOTER_HEIGHT } from '~constants/layout'

const SOCIAL_COLORS = {
  KAKAO: '#ffed16',
  NAVER: '#03cf5d',
  GOOGLE: '#f2f2f2',
} as const

const LAYOUT = {
  ICON_SIZE: '24px',
  BUTTON_HEIGHT: '52px',
  ICON_LEFT_PADDING: '24px',
  CONTAINER_PADDING: '50px 20px',
} as const

export const LoginPageContainer = styled.div`
  background-color: ${({ theme }) => theme.colors.grayscale.gc_4};
  height: 100%;
  min-height: calc(100dvh - ${FOOTER_HEIGHT}px);
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-direction: column;

  padding: 50px 20px;
  position: relative;
`
export const TitleSection = styled.div`
  color: ${({ theme }) => theme.colors.grayscale.font_1};
  font-size: 28px;
  font-weight: 700;
  white-space: pre-line; //줄바꿈!
  text-align: center;
  margin-top: 20%;
  margin-bottom: 30px;
`
export const BrandText = styled.span`
  color: ${({ theme }) => theme.colors.brand.default};
`
export const Logo = styled.div`
  width: 180px;
  height: 180px;
  flex-shrink: 0;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;

  background-color: ${({ theme }) => theme.colors.brand.lighten_2};
  font-size: ${({ theme }) => theme.typography._24};
  color: ${({ theme }) => theme.colors.grayscale.font_1};
`
//소셜로그인
export const SocialLoginSection = styled.div`
  width: 100%;
  max-width: 350px;
  height: 200px;
  margin-top: 50px;
`
const SocialButtonBase = styled.div<{ weight: FontWeight }>`
  width: calc(100% - 12px); // 변경: 동적 너비 계산 (부모 크기 - 좌우 마진)
  height: 52px;
  border-radius: 12px;
  font-size: ${({ theme }) => theme.typography._14};
  font-weight: ${({ weight }) => weight};
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  margin: 10px 6px;
  cursor: pointer;
`
export const Kakao = styled(SocialButtonBase)`
  background: ${SOCIAL_COLORS.KAKAO};
  color: ${({ theme }) => theme.colors.grayscale.font_1};
`
export const Naver = styled(SocialButtonBase)`
  background: ${SOCIAL_COLORS.NAVER};
  color: ${({ theme }) => theme.colors.grayscale.gc_4};
`
export const Google = styled(SocialButtonBase)`
  background: ${SOCIAL_COLORS.GOOGLE};
  color: ${({ theme }) => theme.colors.grayscale.font_1};
`
const IconBase = styled.svg`
  width: ${LAYOUT.ICON_SIZE};
  height: ${LAYOUT.ICON_SIZE};
  position: absolute;
  left: ${LAYOUT.ICON_LEFT_PADDING};
`
export const KakaoIcon = styled(Kakao_Icon)`
  ${IconBase}
`
export const NaverIcon = styled(Naver_Icon)`
  ${IconBase}
`
export const GoogleIcon = styled(Google_Icon)`
  ${IconBase}
`

```

### 주요 개선사항

1. **상수 분리**
- 색상, 레이아웃 관련 값을 상수로 분리해 유지보수성을 향상시켰다.
- 테마와 독립적인 값들을 별도로 관리 하였다.
2. **스타일 최적화** 
- 아이콘 스타일을 베이스 스타일로 통합하였다.
- 공통 스타일 속성을 재사용 가능하게 구성하였다
3. **타입 안정성**
- 상수 객체에 as const 사용해서 타입 안정성을 강화했다
- 스타일 컴포넌트의 props 타입을 명확히 정의하였다
- 컴포넌트의 반환 타입을 ‘JSX.Element’ 로 명시적으로 지정하였다
- 중괄호와 return 문 사용하여 명시적으로 반환하였다.

후반엔 프로젝트 볼륨에 비해 소셜로그인이 많은 것 같다는 의견과, 디벨로퍼 계정 승인 관련하여 Naver가 절차가 까다로운 이슈로 카카오와 구글 로그인만 구현하게 되었다.

하지만 이번 기회로 **컴포넌트 설계와 스타일 최적화**에 대해 공부할 수 있었다!
