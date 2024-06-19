---
title: 자바 코딩 테스트 기술 정리
author: BeAPro
date: 2024-06-16 13:53:00 +0900
categories: [Java]
tags: [Java]
image:
  path: /assets/img/blog/java.png
  alt: deepdive
---
## **최댓값, 최솟값 구하기**
자바에서는 `Math.max`와 `Math.min`을 사용하여 최댓값과 최솟값을 구할 수 있다.

```java
int max = Math.max(a, b);
int min = Math.min(a, b);
```

## **정렬 기준 커스컴**
`Arrays.sort` 메서드를 사용해서 정렬 할 때, 정렬 기준을 커스텀해야 하는 경우가 발생한다.
이 경우에는 `Comparator`에 `compare`메서드를 조건에 맞게 정의하면 된다.

```java
// 이차원 배열의 각 행을 조건에 맞게 배열하는 경우
Arrays.sort(data,new Comparator<int []>(){
  @Override
  public int compare(int[] o1, int[] o2){
      if(o1[col - 1] == o2[col - 1]){
          return o2[0] - o1[0]; // 내림차순
      } else{
          return o1[col - 1] - o2[col - 1]; // 오름차순
      }
  }
});
```