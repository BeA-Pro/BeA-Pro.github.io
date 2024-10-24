---
title: 프리코스 1주차 회고록
author: BeAPro
date: 2024-10-25 03:00:00 +0900
categories: [회고록,우테코]
tags: [회고록]
image:
  path: /assets/img/title-image/memoir.png
  alt: memoir
---
# **우테코 1주차 회고록**
우테코 1주 차가 시작되었다.
문제는 문자열에서 구분자를 추출하고, 기본 구분자들과 해당 구분자들을 이용해 숫자를 나눠 값을 더하는 것이었다.
문제 자체는 쉬웠고 github에 익숙해지는 게 주목적이라고 생각한다.  

나는 해당 문제를 해결하기 위해서 필요한 과정을 정리했다.
1. 문자열을 커스텀 구분자 입력부와 숫자 입력부로 나눈다.
2. 커스텀 구분자 입력부에서 커스텀 구분자를 추출한다.
3. 커스텀 구분자와 기본 구분자를 사용하여 숫자 입력부를 나눈다.
4. 나눠진 숫자 입력부의 요소들이 모두 양수인지 확인한다.
5. 숫자 입력부의 값들을 모두 더한다.

처음에는 하나의 클래스로 위 과정을 모두 해결하려고 했지만 클래스의 역할이 명확하지 않았다.
따라서 문자열을 나누는 역할을 하는 `Splitter` 클래스와 문자열을 양수로 변환하고 더하는 역할을 하는 `Calculator` 클래스로 나누어 구현했다.

## 클래스 설명

### Splitter
문자열을 나누고, 그 과정에서 조건을 만족하지 않으면 에러를 발생시킨다.

- 멤버 변수
  - static #DEFAULT_DELIMITERS : 구분자들을 저장

- 메서드
  - static splitInput (input) : input을 커스텀 구분자 입력부와 숫자 입력부로 나눈다.
  - static addDelimiter(delimiterSection) : delimiterSection에서 커스텀 구분자 입력부를 추출하여 `#DEFAULT_DELIMITERS`에 저장한다.
  - static splitNumberSection(numberSection) : numberSection을 `#DEFAULT_DELIMITERS`에 있는 구분자로 나눈다.
  - static getDelimiters() : `#DEFAULT_DELIMITERS`를 반환한다.
 
### Calculator
문자열로 주어진 숫자가 양수인지 확인하고 변환한다.
양수로 이루어진 숫자 배열의 값을 모두 더한다.
- 메서드
  - static checkIsPositiveNumber(numStrArray) : numStrArray에 있는 문자열이 양수인지 확인하고 변환한다.
  - static addNumbers(numArray) : numArray에 있는 모든 요소들의 합하여 반환한다.


## **결과**
![Desktop](/assets/img/memoir/2024-10-25-01.png){: width="500" height="300" }

## **느낀점**
문제에 대한 명확한 가이드라인이 없었고, 내가 짠 코드에 너무 많은 예외가 존재한다고 생각해서 이게 맞나 고민했다.
다행히 예제 테스트는 단 두개로 엄청 후하게 테스트했다.
고민하며 두려워하기보다는 그냥 코드를 짜는 게 맞는 것 같다.

내 코드에서 아쉬웠던 점은 `splitNumberSection(numberSection)`에서 숫자 입력부를 구분자로 나누고 결과를 저장할 때 flatMap을 사용했는데 map 메서드를 사용해도 됐다.  
그리고 `#DEFAULT_DELIMITERS`의 변수 명도 커스텀 구분자가 들어가기 때문에 그냥 `#DELIMITERS`로 명명하는 게 맞다고 생각한다.
다음에는 좀 더 정확하게 코딩하기 위해 노력해야겠다.

