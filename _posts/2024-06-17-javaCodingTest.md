---
title: 자바 코딩 테스트 기술 정리
author: BeAPro
date: 2024-06-16 13:53:00 +0900
categories: [Java]
tags: [Java]
image:
  path: /assets/img/title-image/java.png
  alt: deepdive
---
## **절댓값 구하기**
`Math.abs`를 사용하여 숫자의 절댓값을 구할 수 있다.

```java
int a = -5;
int absA = Math.abs(a);
```

## **최댓값, 최솟값 구하기**
자바에서는 `Math.max`와 `Math.min`을 사용하여 최댓값과 최솟값을 구할 수 있다.

```java
int max = Math.max(a, b);
int min = Math.min(a, b);
```

## **Integer 최댓값, 최솟값**
`Integer`에서 `MAX_VALUE`와 `MIN_VALUE`를 사용하여 `int`형의 최댓값과 최솟값을 불러올 수 있다.
```java
int maxValue = Integer.MAX_VALUE;
int minValue = Integer.MIN_VALUE;
```

## **Sort 정렬 기준 커스컴**
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

## **우선순위 큐**

```java
// 선언 방법
PriorityQueue<Integer> pq = new PrirorityQueue<>();

// 더하는 법
pq.add(10);

// 상위 값 확인하는 법
System.out.println(pq.peek());

// 요소 제거하는 법
System.out.println(pq.poll());
```
사용자가 선언한 클래스를 값으로 받는 경우 `Comparator`를 우선순위를 커스텀해줘야 한다.
```java
class Person {
    String name;
    int age;

    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return name + " (" + age + ")";
    }
}

public class Main {
    public static void main(String[] args) {
        // 나이 순으로 정렬하는 Comparator
        Comparator<Person> ageComparator = new Comparator<Person>() {
            @Override
            public int compare(Person p1, Person p2) {
                return Integer.compare(p1.age, p2.age);
            }
        };
        PriorityQueue<Person> pq = new PriorityQueue<>(ageComparator);
    }
}
```




## **입력 빠르게 받기**
입력 값이 많을 때, `Scanner`를 사용하면 시간초과가 날 수 있다.
이런 경우 `BufferedReader`를 사용한다.

```java
public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        
    }
}
```
주의해야 할 점은 `readLine()` 메서드를 통해 값을 받을 경우 데이터 형식이 `String`으로 고정된다는 것이다.
따라서 값을 입력 받고 알맞은 데이터 타입으로의 변환이 필요하다.

```java
int i = Integer.parseInt(bf.readLine());
```
