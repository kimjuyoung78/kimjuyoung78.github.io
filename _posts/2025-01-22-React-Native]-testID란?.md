---
author: 주영
date: 2025-01-22 17:10:00 +0800
last_modified_at: 2025-01-17 20:00:00 +0900
categories: [Develop]
tags: [DDang(댕댕) Project, ReactNative]
render_with_liquid: false
image: assets/img/react.png
---

<br><br>
프로젝트에서 공통 컴포넌트를 구현하는 가운데, Storybook을 도입해서 적용하였다. 

이때 **testID라는 개념을 새롭게 알게 되었다.**

<br><br>

## **testID란?**

**React Native에서 UI 테스트를 위해 특정 컴포넌트를 고유하게 식별할 수 있도록 돕는 속성이다.**

1. **자동화된 테스트 지원**: Jest, React Native Testing Library, Detox와 같은 도구에서 특정 UI 요소를 식별하고 조작할 수 있도록 한다.
2. **테스트 안정성 강화**: UI 구조 변경에도 영향을 덜 받으므로 테스트 코드의 유지보수가 용이하다.
3. **효율적인 컴포넌트 선택**: **getByTestId**와 같은 메서드를 통해 빠르고 정확하게 요소를 탐색할 수 있다.

<br><br>
물론 React에서도 유사한 개념이 있다. React에는 **'data-testid'** 속성을 사용하여 비슷한 목적을 달성한다.
<br>
<br>
<br>
내 프로젝트 코드의 예시로 알아보자. Profile 공통 컴포넌트를 제작하였다.
<br>

<img width="126" alt="image" src="https://github.com/user-attachments/assets/44610850-ef10-4716-b752-70bccdc5e26a" />
 <br>컴포넌트는 요렇게 생겼다.


<br><br>

## **Profile 컴포넌트에 testID 적용하기**

이 컴포넌트는 사용자 프로필 이미지를 표시하며, 클릭 가능 여부는 **`userId`** prop의 유무로 결정된다.


### [Profile/index.tsx]

```jsx
import React from 'react';
import { TouchableOpacity, Image } from 'react-native';
import { SvgProps } from 'react-native-svg';

type ProfileProps = {
  size: number;
  src: string | React.FC<SvgProps>;
  userId?: number;
  testID?: string; *// testID를 옵셔널 prop으로 추가*
};

export default function Profile({ size, src, userId, testID }: ProfileProps) {
  const renderImage = () => {
    if (typeof src === 'string') {
      return <Image source={{ uri: src }} style={{ width: size, height: size }} />;
    }
    const SvgComponent = src;
    return <SvgComponent width={size} height={size} />;
  };

  return (
    <TouchableOpacity activeOpacity={userId ? 0.8 : 1} disabled={!userId} testID={testID}>
      {renderImage()}
    </TouchableOpacity>
  );
}
```

```jsx
    <TouchableOpacity activeOpacity={userId ? 0.8 : 1} disabled={!userId} testID={testID}>
      {renderImage()}
    </TouchableOpacity>
```

`TouchableOpacity` 컴포넌트에 `testID`를 전달하여 테스트 환경에서 이 요소를 식별할 수 있도록 했다.

`testID`는 선택적(`optional`) prop으로 설정되어 필요할 때만 유연하게 사용할 수 있다.

- **사용 예시**:
    - 클릭 가능한 프로필: `userId`가 존재하면 **터치 피드백이 활성화**된다.
    - 클릭 불가능한 프로필: `userId`가 없으면 **터치 피드백이 비활성화**된다.

<br><br>

## **Storybook에서의 활용**

Storybook을 사용해 `Profile` 컴포넌트를 시각적으로 테스트할 때도 **testID**를 활용할 수 있다. 

### **[Profile.stories.tsx]**

```jsx
import type { Meta, StoryObj } from '@storybook/react';
import Profile from './index';
import Avatar1 from '~assets/avatars/Avatar1.svg';
import Avatar2 from '~assets/avatars/Avatar2.svg';

const meta: Meta<typeof Profile> = {
  title: 'Profile',
  component: Profile,
  argTypes: {
    size: { control: { type: 'number' }, description: '프로필 이미지의 크기' },
    src: { control: { type: 'text' }, description: '프로필 이미지 URL 또는 SVG 컴포넌트' },
    userId: { control: { type: 'number' }, description: '사용자 ID (있으면 클릭 가능)' },
  },
};

export default meta;

type Story = StoryObj<typeof Profile>;

export const Default: Story = {
  args: {
    size: 48,
    src: Avatar1,
    userId: 1,
    testID: 'default-profile', *// testID 추가*
  },
};

export const NonClickable: Story = {
  args: {
    size: 140,
    src: Avatar2,
    testID: 'nonclickable-profile', *// testID 추가*
  },
};
```

- 각 스토리에 **testID**를 부여하여 특정 상태의 컴포넌트를 명확히 구분했다.
<br><br>

```jsx
export const Default: Story = {
  args: {
    size: 48,
    src: Avatar1,
    userId: 1,
    testID: 'default-profile', *// testID 추가*
  },
};

export const NonClickable: Story = {
  args: {
    size: 140,
    src: Avatar2,
    testID: 'nonclickable-profile', *// testID 추가*
  },
};
```

- **`Default`** 스토리는 클릭 가능한 프로필을 나타내며, **`NonClickable`** 스토리는 클릭 불가능한 프로필을 나타낸다.

  움짤로 보면 클릭 가능 불가능 여부를 쉽게 알 수 있다.

  <img src="https://github.com/user-attachments/assets/63d35dd3-02ac-4dcc-9bf9-3819ae423c01" width="50%" alt="20250122_163510">


<br><br>
## **테스트 코드에서 testID 활용**

```jsx
import { render, fireEvent } from '@testing-library/react-native';
import Profile from './index';
import Avatar1 from '~assets/avatars/Avatar1.svg';

describe('Profile Component', () => {
  it('renders and responds to touch when userId is provided', () => {
    const { getByTestId } = render(
      <Profile size={64} src={Avatar1} userId={1} testID="profile-touchable" />
    );

    const profileTouchable = getByTestId('profile-touchable');
    expect(profileTouchable).toBeTruthy();

    fireEvent.press(profileTouchable);
    *// 추가적인 동작 확인 로직 작성 가능*
  });
});
```

테스트 환경에서는 다음과 같이 **testID** 를 사용해 요소를 선택하고 상호작용할 수 있다. 



<br><br><br>

## **testID는 꼭 'testID'라는 이름을 사용해야 할까?**
<br>

그리고 testID 라는 단어는 고유명사가 아니다. UI를 테스트 하기 위한 하나의 개념으로 이해할 수 있다. 

때문에 기본적으로 'testID'라는 이름을 사용하지만, 다른 도구나 프레임워크에서는 이를 대체하는 속성을 정의할 수 있다!
<br>

그 예시로 **React Testing Library**에서는 `data-testid`라는 속성을 주로 사용한다 


참고 : https://www.educative.io/answers/what-is-the-data-testid-attribute-in-testing 

<br>

그치만 React Native 환경에서는 testID가 기본적으로 지원어서 일관성 유지 부분에서는 그대로 쓰는데 좋다.

