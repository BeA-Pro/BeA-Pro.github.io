---
title: Chapter 3. 첫 번째 REST API
author: BeAPro
date: 2024-07-03 21:25:00 +0900
categories: [Study,Spring Boot]
tags: [Spring Boot]
image:
  path: /assets/img/title-image/springBoot.png
  alt: SpringBoot
---

## **REST가 무엇이며, 왜 중요할까요?**
API는 개발자가 작성한 사양/인터페이스이다. API를 통해 라이브러리, 다른 애플리케이션, 서비스 같은 다른 코드를 사용할 수 있다.  

REST는 Representational State Transfer의 약어이다. 애플리케이션 A가 애플리케이션 B와 통신할 때, A는 B에서 통신 시점의 현재상태(State)를 가져온다. A는 B가 통신 호출(communication calls)간 상태를 유지하리라고는 가정하지 않는다. A는 B에 요청할 때마다 관련 상태의 표현(representation)을 제공한다.(ex. https://api.example.com/users/123) 이런 동작 방식은 생존가능성(survivablility)과 회복탄력성(resilience)을 향상시킨다.  

이렇게 처리하면 통신 문제가 발생하거나 B가 충돌해 재시작돼도, A와 상호작용한 '현재 상태'를 잃어버리지 않기 때문이다. A가 다시 요청해 두 애플리케이션이 중단된 곳에서 통신을 이어간다.

> 위 내용은 비연결성과 관련이 높다.
{: .prompt-tip}

## **API, HTTP 메서드 스타일**

REST API는 주로 다음의 HTTP 메서드를 기반으로 한다.

- POST
- GET
- PUT
- PATCH
- DELETE

이는 생성(POST), 읽기(GET), 업데이트(PUT 및 PATCH), 삭제(DELETE)와 같이 리소스에 수행하는 일반적인 작업이다.

- OPTIONS
- HEAD

위에 두 메서드는 요청/응답 쌍에서 사용할 수 있는 통신 옵션을 조회하고(OPTIONS), 응답 메시지에서 바디를 뺀 응답 헤더를 조회하는 (HEAD) 데 사용합니다.

## **GET으로 시작하기**

### @RestController 개요

스프링 MVC는 뷰가 서버 렌더링 웹페이지로 제공된다는 가정하에, 데이터와 데이터를 전송하는 부분과 데이터를 표현하는 부분은 분리해 생성한다. 이러한 MVC의 여러 부분을 연결하는 데 `@Controller` 어노테이션이 도움이된다.

`@Controller`는 `@Component`의 스테레오타입이다. 따라서 애플리케이션 실행시, 스프링 빈(애플리케이션 내 객체로서 스프링 IoC 컨테이너에 의해 생성되고 관리됨) `@Controller` 어노테이션이 붙은 클래스로부터 생성된다.

`@Controller`가 붙은 클래스는 `Model` 객체를 받는다. `Model` 객체로 표현 계층(presentation layer)에 모델 기반 데이터를 제공한다. 또, `ViewResolver`와 함께 작동해 애플리케이션이 뷰 기술에 의해 렌더링된 특정 뷰를 표시하게 한다.

간단히 `@ResponseBody`를 클래스나 메서드에 추가해서 JSON이나 XML 같은 데이터 형식 처럼 형식화된 응답을 반환하도록 Controller 클래스에 지시할 수도 있다. 이렇게 하면 메서드의 객체 반환값이 웹 응답의 전체 바디가 된다.

`@ResController` 어노테이션은 `@Controller`와 `@ResponseBody`를 하나의 어노테이션으로 합쳐 쓴 것이다. 클래스에 `@RestController`로 어노테이션을 달아서 REST API를 만들 수 있다.

### GET
```java
@RestController
class RestApiDemoController{
	private List<Coffee> coffees = new ArrayList<>();

	public RestApiDemoController(){
		coffees.addAll(List.of(
				new Coffee("Cafe Cereza"),
				new Coffee("Cafe Ganador"),
				new Coffee("Cafe Lareno"),
				new Coffee("Cafe Tres Pontas")
		));
	}

	@GetMapping("/coffees")
	Iterable<Coffee> getCoffees(){
		return coffees;
	}
}
```

## **POST로 생성하기**

```java
// RestController에 추가
@PostMapping("/coffees")
Coffee postCoffee(@RequestBody Coffee coffee){
    coffees.add(coffee);
    return coffee;
}
```

## **PUT으로 업데이트하기**
일반적으로 PUT 요청은 파악된 URI를 통해 기존 리소스의 업데이트에 사용된다.

```java
// RestController에 추가
@PutMapping("/coffees/{id}")
Coffee putCoffee(@PathVariable String id, @RequestBody Coffee coffee){
    int coffeeIndex = -1;

    for(Coffee c : coffees){
        if(c.getId().equals(id)){
            coffeeIndex = coffees.indexOf(c);
            coffees.set(coffeeIndex,coffee);
        }
    }

    return (coffeeIndex == -1) ? postCoffee(coffee) : coffee;
}
```

## **DELETE로 삭제하기**

```java
@DeleteMapping("/coffees/{id}")
void deleteCoffee(@PathVariable String id){
    coffees.removeIf(c -> c.getId().equals(id));
}
```