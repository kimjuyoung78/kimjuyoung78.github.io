---
author: 주영
date: 2024-06-23 11:40:00 +0800
categories: [Algorithm]
tags: [Algorithm, Javascript]
render_with_liquid: false
image: assets/img/programmers.jpg
---
# [프로그래머스] 비밀지도 - JS 풀이 및 함수 분해

<br>

![image](https://github.com/user-attachments/assets/bbd58b86-912b-4876-8cc7-628f28c647cc)


## 문제 설명  
네오는 평소 프로도가 비상금을 숨겨놓는 장소를 알려줄 비밀지도를 손에 넣었습니다.  
그런데 이 비밀지도는 숫자로 암호화되어 있어 위치를 확인하기 위해서는 암호를 해독해야 합니다.  
다행히 지도 암호를 해독할 방법을 적어놓은 메모도 함께 발견했습니다.

<br>

지도는 한 변의 길이가 n인 정사각형 배열 형태로, 각 칸은 "공백"(" ") 또는 "벽"("#") 두 종류로 이루어져 있습니다.  
전체 지도는 두 장의 지도를 겹쳐서 얻을 수 있습니다. 각각 "지도 1"과 "지도 2"라고 하겠습니다.  
지도 1 또는 지도 2 중 어느 하나라도 벽인 부분은 전체 지도에서도 벽이 됩니다.  
지도 1과 지도 2에서 모두 공백인 부분은 전체 지도에서도 공백이 됩니다.

<br>

"지도 1"과 "지도 2"는 각각 정수 배열로 암호화되어 있습니다.  
암호화된 배열은 지도의 각 가로줄에서 벽 부분을 1, 공백 부분을 0으로 부호화했을 때 얻어지는 이진수에 해당하는 값의 배열입니다.

<br>

---

## 입력 형식

- 입력으로 지도의 한 변 크기 n 과 2개의 정수 배열 arr1, arr2가 들어옵니다.
- 1 ≦ n ≦ 16
- arr1, arr2는 길이 n인 정수 배열로 주어집니다.
- 정수 배열의 각 원소 x를 이진수로 변환했을 때의 길이는 n 이하입니다.  
  즉, 0 ≦ x ≦ 2^n - 1을 만족합니다.

<br>

## 출력 형식

- 원래의 비밀지도를 해독하여 '#', 공백으로 구성된 문자열 배열로 출력합니다.

<br>

---

## 입출력 예제

### 예제 1

- n: 5  
- arr1: [9, 20, 28, 18, 11]  
- arr2: [30, 1, 21, 17, 28]  
- 출력: ["#####", "# # #", "### #", "# ##", "#####"]

<br>

### 예제 2

- n: 6  
- arr1: [46, 33, 33 ,22, 31, 50]  
- arr2: [27 ,56, 19, 14, 14, 10]  
- 출력: ["######", "###  #", "##  ##", " #### ", " #####", "### # "]

<br>

---

## 코드

```jsx
function solution(n, arr1, arr2) {
  // 인수분해 함수
  let arr1_n = factorization(arr1, n);
  let arr2_n = factorization(arr2, n);
  // 벽 통합 함수
  let arr3 = Integration(arr1_n, arr2_n);
  // 1 -> '#' 변환 함수
  arr3 = Transformation(arr3);

  return arr3;
}

// 1. 인수분해 함수
function factorization(arr, n) {
  return arr.map(num => {
    let binary = num.toString(2);
    return binary.padStart(n, '0');
  });
}

// 2. 벽 통합 함수
function Integration(arr1, arr2) {
  return arr1.map((a, b) => {
    let combined = '';
    for (let i = 0; i < a.length; i++) {
      combined += (parseInt(a[i]) | parseInt(arr2[b][i])).toString();
    }
    return combined;
  });
}

// 3. 1->'#' 변환 함수
function Transformation(arr) {
  return arr.map((row) => {
    return row.replace(/1/g, '#').replace(/0/g, ' ');
  });
}
```

<br>

---

## 풀이 방법

문제 자체가 길어서 꼼꼼하게 읽는 게 첫 번째!  
바로 손코딩하지 않고, 아이패드에 그려 가면서 이해(특히 인수분해, 진수 변환을 오랜만에 기억에서 끄집어내느라...)  
번호를 매겨서 코드 작성 순서를 정했다.  
계산할 게 많으니 함수를 여러 개로 나눠서 차례로 호출시키는 방법을 사용했다.  
이후에는 문법 싸움이었다... 각 함수 안에서 문자열과 배열로 씨름하는 게 너무 힘들었다.  
아직 JS로 문제 푸는 게 약해서 그런 듯하다. (학교에서 맨날 C로 자료구조, 알고리즘 했으니...)  
함수를 하나씩 뜯어보자.

<br>

---

### 1. 인수분해 함수

```jsx
function factorization(arr, n) {
  return arr.map(num => {
    let binary = num.toString(2); // 2진수 문자열로 변환
    return binary.padStart(n, '0'); // n 길이에 맞게 0으로 패딩
  });
}
```

- 일단 10진수로 주어지는 정수들을 2진수로 변환한다.
- 총 n자리 수이니 앞자리만큼 빈 부분을 0으로 채워준다.

<br>

---

### 2. 벽 통합 함수

```jsx
function Integration(arr1, arr2) {
  return arr1.map((a, b) => {
    let combined = '';
    for (let i = 0; i < a.length; i++) {
      combined += (parseInt(a[i]) | parseInt(arr2[b][i])).toString();
    }
    return combined;
  });
}
```

- map 메소드로 배열을 펼친 다음, combined 배열에다가 합친 배열을 넣는다.
- 각 이진수 문자열의 각 비트 순회
- 현재 비트를 정수로 변환, OR 연산 수행

<br>

---

### 3. 1->'#' 변환 함수

```jsx
function Transformation(arr) {
  return arr.map((row) => {
    return row.replace(/1/g, '#').replace(/0/g, ' ');
  });
}
```

- 이진수 문자열을 지도의 형태로 변환한다.
- 1은 '#'으로, 0은 공백(' ')으로 변환

<br>

---

## 정리

- 문제를 꼼꼼히 읽고, 단계별로 함수로 분리해서 해결
- 각 함수에서 JS의 문자열, 배열 메소드 활용
- 2진수 변환, 패딩, OR 연산, 문자열 치환 등 다양한 JS 메소드 경험 가능

<br>

증말... 메소드의 향연이다...
