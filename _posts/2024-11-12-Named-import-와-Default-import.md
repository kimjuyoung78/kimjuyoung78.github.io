---
author: 주영
date: 2024-11-12 19:10:00 +0800
categories: [Concept]
tags: [DDang(댕댕) Project, React]
render_with_liquid: false
---

<br/>
<br/>
<br/>

Toggle 공통 컴포넌트를 import 해서 사용하는 와중, import 방식이 헷갈려 한번 정리해 보고자 한다.

<img src=“https://github.com/user-attachments/assets/6467fdf8-18e5-4881-820c-a5b5acab6208” width=40% />

<br/>
<br/>
<br/>

```tsx
import { ToggleArea } from '~components/ToggleArea'
```

와

```tsx
import ToggleArea from '~components/ToggleArea'
```

의 차이점이 뭘까?

<br/>
<br/>

## **Export 방식의 차이**
<br/>

**[1] Default Export**

```jsx
*// 파일당 하나만 가능*
export default function ToggleArea() {
  *// ...*
}

*// 임포트 시*
import ToggleArea from '~components/ToggleArea'
```
<br/>

**[2] Named Export**

```jsx
*// 여러 개 가능*
export function ToggleArea() {
  *// ...*
}

*// 임포트 시*
import { ToggleArea } from '~components/ToggleArea'
```
<br/>
<br/>
<br/>

## **주요 차이점**

### **개수 제한**

- Default export는 파일당 하나만 가능하다.
- Named export는 여러 개를 export할 수 있다.
<br/>

### **임포트 방식**

- Default export는 **중괄호 없이 원하는 이름으로 임포트 가능**하다.
- Named export는 반드시 **중괄호**를 사용하고 **export된 이름과 동일하게** 임포트해야 한다.

<br/>

### **사용 시나리오**

- Default export는 **모듈당 하나의 주요 기능을 내보낼 때** 사용합니다
- Named export는 **여러 관련 기능을 함께 내보낼 때** 사용합니다

따라서 현재 상황에선 **ToggleArea 컴포넌트가 default export로 선언되어 있기 때문에 중괄호 없이 임포트해야 한다.**

<br/>

### Named export 예시

```jsx
// React hooks
import { useState, useEffect, useContext } from 'react';

// React Router
import { BrowserRouter, Route, Switch, useParams } from 'react-router-dom';

// Styled Components
import { ThemeProvider, css } from 'styled-components';

// Custom components or functions
import { Button, Input, Card } from './components';
import { fetchData, processData } from './utils';

// Redux
import { useSelector, useDispatch } from 'react-redux';
```
<br/>

### Default export 예시

```jsx
// React
import React from 'react';

// Axios for HTTP requests
import axios from 'axios';

// Lodash utility library
import _ from 'lodash';

// Custom components
import Header from './components/Header';
import Footer from './components/Footer';

// Configuration file
import config from './config';

// Main stylesheet
import styles from './styles/main.css';
```
<br/>

### 혼합사용하는 경우

```typescripts
// React and hooks
import React, { useState, useEffect } from 'react';

// Redux and hooks
import { createStore } from 'redux';
import { Provider, useSelector, useDispatch } from 'react-redux';

// Styled-components and utilities
import styled, { css, ThemeProvider } from 'styled-components';

// Custom module with default and named exports
import MyModule, { helperFunction, CONSTANT_VALUE } from './MyModule';
```
<br/>
<br/>
<br/>

표로 정리해보면 다음과 같다
<br/>

| 특징 | Named Import | Default Import |
| --- | --- | --- |
| **구문** | `import { ComponentName } from './file'` | `import ComponentName from './file'` |
| **다중 import** | 가능 </br>(예: `import { ComponentA, ComponentB } from './file'`) | 불가능 </br>(파일당 하나만) |
| **Export 방식** | `export { ComponentName }` 또는</br> `export const ComponentName = ...` | `export default ComponentName` |
| **Import 시 이름** | 정확한 이름 사용 필요 </br>(변경 시: `import { ComponentName as AnotherName }`) | 자유롭게 이름 지정 가능 |
| **파일당 개수** | 여러 개 가능 | 하나만 가능 |
| **예시** | `import { Link } from "react-router-dom";` | `import React from 'react';` |
| **사용 시나리오** | 라이브러리나 유틸리티 함수처럼 여러 개의 관련 항목을 내보낼 때 유용 | 주로 한 파일에 하나의 주요 컴포넌트가 있을 때 사용 |


용도에 맞게 잘 활용하도록 하자!
