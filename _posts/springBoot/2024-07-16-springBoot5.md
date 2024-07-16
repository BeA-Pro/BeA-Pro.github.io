---
title: Chapter 5. 애플리케이션 설정과 검사
author: BeAPro
date: 2024-07-16 12:20:00 +0900
categories: [Study,Spring Boot]
tags: [Spring Boot, JPA]
image:
  path: /assets/img/title-image/springBoot.png
  alt: SpringBoot
---

디버깅은 개발자로 입문할 때부터 배워서 개발자로 일하는 동안 개선하고 확장해야 할 기본 기량이다.
디버깅 기능을 아직 숙달하지 못했다면, 디버깅하는 여러 방법을 가능한 빨리 찾아보고 공부하는 것이 좋다.
디버깅은 개발에서 매우 중요하며 시간을 엄청나게 절약해준다.

코드 디버깅은 중요하지만, 애플리케이션 내 동작을 구축, 식별, 분리하는 한 단계에 불과하다. 동적이고 분산된 애플리케이션이 많아지면 다음 작업을 수행해야 한다.

- 애플리케이션의 동적 설정과 재설정
- 현재 설정과 출처의 확인과 결정
- 애플리케이션 환경과 헬스 지표의 검사와 모니터링
- 실행 중인 애플리케이션의 로깅 수준을 일시적으로 조정해 오류 원인 식별

## **@Value**

`@Value` 어노테이션은 아마 설정(configuration)을 코드에 녹이는 가장 간단한 접근방식이다.
패턴 매칭과 SpEL(Spring Experssion Language)을 기반으로 구축되어 간단하고 강력하다.

>pring Expression Language (SpEL)은 런타임에 객체 그래프를 조회하고 조작할 수 있는 강력한 표현 언어이다. 주로 Spring 프레임워크 내에서 애플리케이션 구성의 동적 동작을 제공하기 위해 사용된다. SpEL 표현식은 빈 정의를 지정하고, 구성 속성을 관리하며, 애플리케이션의 동적 측면을 처리하는 데 사용될 수 있다.

`application.properties` 아링에 속성 하나를 정의한다.

```
greeting-name=Dakota
```

이 속성을 사용하는 방법을 보여주기 위해 `greeing` 애플리케이션에 사용자 관련 작업을 처리하는 `@RestController`를 추가로 만든다.

```java
@RestController
@RequestMapping("/greeting")
class GreetingController{

	@Value("${greeting-name: Mirage}")
	private String name;

	@GetMapping
	String getGreeting(){
		return name;
	}
}
```

`@Value` 어노테이션이 멤버 변수 name에 적용되고, 어노테이션에 문자열 타입을 단일 매개변수로 하여 속성값 SpEL형식으로 적었다.

애플리케이션을 실행하고 `/greeting`을 쿼리하면, 예상대로 "Dakota"로 응답값을 반환한다.

속성 기본값을 사용하는지 검증하기 위해, applictaion.properties에 속성 정의를 `#`으로 주석 처리하고 애플리케이션을 재실행한다.

```
#greeting-name=Dakota
```

그럴 경우 "Mirage"를 반환한다.

- ${greeting-name}: 이 부분은 Spring이 greeting-name이라는 구성 속성을 찾도록 지시
- : Mirage: 만약 greeting-name이라는 속성이 구성 파일(application.properties 등)이나 환경 변수에 정의되어 있지 않으면, 기본값으로 Mirage를 사용하겠다는 의미

맞춤설정 속성과 @Value를 함께 사용하면, 한 속성값을 다른 속성값으로 파생하거나 설정 할 수 있다.
속성 중첩이 작동하는 법을 시연하려면 최소 두 개의 속성이 필요하다.

application.properties에 greeting-coffee라는 두 번째 속성을 만든다.

```
greeting-name=Dakota
greeting-coffee=${greeeting-name} is drinking Cafe Cereza
```

다음으로는 GreetingController에  coffee/greeting 관련 필드와 엔드포인트로 coffee 데이터를 조회하는 코드를 추가한다.

```java
@RestController
@RequestMapping("/greeting")
class GreetingController{

	@Value("${greeting-name: Mirage}")
	private String name;
	
	@Value("${greeting-coffee: ${greeting-name} is drinking Cafe Ganador}")
	private String coffee;

	@GetMapping
	String getGreeting(){
		return name;
	}

  @GetMapping("/coffee")
	String getNameAndCoffee(){
		return coffee;
	}
}
```
위 코드의 경우 /greeting/coffee에 쿼리하면 `Dakota is dringking Cafe Cereza`를 받아온다.

@Value 사용에도 제한이 있다. greeting-coffee에는 기본값이 있어서 application.properties에 해당 속성 어노테이션 처리한다고 해도, GreetingController 내 @Value 어노테이션은 coffee 멤버 변수를 사용해 해당 속성값을 기본값을 적절하게 처리한다.
하지만 속성 파일에서 greeting-name과 greeting-coffee를 모두 어노테이션 처리하면, 속성값을 정의하는 환경 소스가 없게 되므로 에러가 발생한다.

## **@ConfigurationProperties**

@Value는 유연하지만 단범이 있기 때문에, 스프링 개발팀에서는 @ConfigurationProperties를 만들었다.
이는 속성을 정의하고 관련 속성을 그룹화해서, 도구로 검증 가능하고 타입 세이프한 방식으로 속성을 참조하고 사용한다.

예를 들어 코드에서 사용하지 않는 속성이 앱의 application.properties 파일에 정의됐다면, IDE에서 사용되지 않는 속성이라는 의미로 하이라이트가 표시된다.
또한, 속성이 문자열로 정의됐지만 다른 타입의 멤버 변수와 연결된 경우, IDE는 타입 불일치를 알린다.

```java
@ConfigurationProperties(prefix =  "greeting")
class Greeting{
	private String name;
	private String coffee;

	public String getName(){
		return name;
	}

	public void setName(String name){
		this.name = name;
	}

	public String getCoffee(){
		return coffee;
	}

	public void setCoffee(String coffee){
		this.coffee = coffee;
	}
}
```
@ConfigurationProperties 어노테이션을 추가하고 모든 greeting 속성에 사용할 prefix를 지정한다. 이 어노테이션은 설정 속성에서만 사용하도록 클래스를 설정한다.
또, @ConfigurationProperties 어노테이션이 달린 클래스를 애플리케이션이 처리하도록 추가 설정이 필요하다.

```java
@SpringBootApplication
@ConfigurationPropertiesScan
public class SbrRestDemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(SbrRestDemoApplication.class, args);
	}

}
```
기본 애플리케이션 클래스에 @ConfigurationPropertiesScan 어노테이션을 기본 애플리케이션 클래스에 추가해야한다. 그래야 @ConfigurationProperties 어노테이션을 가진 클래스가 빈으로 등록된다.

그다음 어노테이션 프로세서로 메타데이터를 생성하기 위해서 IDE가 @ConfigurationProperties 클래스와 application.properties 파일에 정의된 관련 속성을 연결하도록 프로젝트의 pom.xml 빌드 파일에 다음 의존성을 추가한다.

```java
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-configuration-processor</artifactId>
  <optional>true</optional>
</dependency>
```
이후 의존성을 업데이트하고 설정 프로세서를 포함하기 위해 IDE에서 프로젝트를 빌드하고 속성명을 아래와 같이 수정한다.

```java
greeting.name=Dakota
greeting.coffee=${greeting.name} is drinking Cafe Cereza
```

```java
@RestController
@RequestMapping("/greeting")
class GreetingController {
	private final Greeting greeting;

	public GreetingController(Greeting greeting) {
		this.greeting = greeting;
	}

	@GetMapping
	String getGreeting() {
		return greeting.getName();
	}

	@GetMapping("/coffee")
	String getNameAndCoffee() {
		return greeting.getCoffee();
	}
}
```
GreetingController의 자체 멤버 변수 nmae, coffee와 @Value 어노테이션을 사용하지 않고, 대신 greeting.name과 greeting.coffee 속성을 관리하는 greeting 빈에 대한 멤버 변수를 생성하고 다음 코드와 같이 생성자 주입을 거쳐 GreetingController에 greeting 빈을 주입한다.

@ConfigurationProperties 빈이 관리하는 속성은 환경과 환경 속성에 사용될 수동 있는 잠재적 소스에서 속성값을 얻는다.
@Value 기반 속성과 유일하게 다른 점 하나는 어노테이션이 달린 멤버 변수에 기본값을 지정할 수 없다는 것이다.
애플리케이션의 application.properties 파일이 보통 애플리케이션 기본값을 정의하는 데 사용되므로, 기본값 지정 기능이 없어도 유용하다.
다양한 배표 환경에서 환경마다 다른 속성값이 필용한 경우, 속성값은 다른 소스(예: 환경 변수 또는 명령 줄 매개변수)를 통해 애플리케이션 환경에 적용된다.
그냥 @Value 대신 @ConfigurationProperties를 사용하자.

## **잠재적 서드 파티 옵션**

@ConfigurationProperties를 사용하면 서드 파티 컴포넌트를 감싸고 해당 속성을 애플리케이션 환경에 통합 할 수 있다.

```java
class Droid {
	private String id, description;

	public String getId() {
		return id;
	}

	public void setId(String id) {
		this.id = id;
	}

	public String getDescription() {
		return description;
	}

	public void setDescription(String description) {
		this.description = description;
	}
}
```
시연용 POJO(Plain Old Java Object)를 생성한다. 이 기능은 직접 컴포넌트를 생성하는 대신 프로젝트에 외부 의존성을 추가하고, 컴포넌트 문서를 참조해 스프링 빈을 생성할 때 가장 유용하다.

> POJO는 "Plain Old Java Object"의 약자이다. 이는 Java에서 특정 프레임워크나 규칙에 종속되지 않은 순수한 자바 객체를 의미한다. POJO는 특별한 요구사항이나 제약 없이 단순히 필드와 메서드만으로 구성된 객체를 가리킨다.
{: .prompt-info}

다음 단계는 실제 서드 파티 컴포넌트와 동일한 방식으로 진행된다. 바로 컴포넌트를 스프링 빈으로 인스턴스화하는 것이다. 정의된 POJO로부터 스프링 빈을 생성하는 방법은 여러 가지지만, 이 특정 사례에서는 @Configuration 어노테이션이 달린 클래스 내에서 직접 또는 메타 어노테이션을 통해 @Bean 어노테이션이 달린 메서드를 생성하는 방법이 가장 적합하다.

여러 메타 어노테이션 중에서 @Configuration을 포함하는 메타 어노테이션이 바로 @SpringBootApplication이다.

이제 빈 생성 방법과 요구되는 @ConfigurationProperties 어노테이션과 prefix 매개변수를 보여준다. 

```java
@SpringBootApplication
@ConfigurationPropertiesScan
public class SbrRestDemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(SbrRestDemoApplication.class, args);
	}

	@Bean
	@ConfigurationProperties(prefix = "droid")
	Droid createDroid() {
		return new Droid();
	}
}
```
application.properties에 다음 속성을 추가한다.

```java
droid.id=BB-8
droid.description=Small, rolling android. Probably doesn't drink coffee
```

이제 Droids 속성을 사용해 작동하는지 하인할 수 있는 @GetMapping 메서드가 있는 @RestController를 만든다.

```java
@RestController
@RequestMapping("/droid")
class DroidController {
	private final Droid droid;

	public DroidController(Droid droid) {
		this.droid = droid;
	}

	@GetMapping
	Droid getDroid() {
		return droid;
	}
}
```
새로운 /droid 엔드포인트에 쿼리하면 적절한 응답을 받을 수 있다.

이처럼 @ConfigurationProperties를 사용하면 외부 라이브러리, 프레임워크를 감싸고 해당 속성을 애플리케이션 환경에 통합 할 수 있다.

## **자동 설정 리포트**

스프링 부트는 자동 설정을 통해 많은 작업에서 개발자를 대행한다. 선택한 기능, 의존성 또는 코드의 일부 기능을 수행하는데 필요한 빈을 사용해 애플리케이션을 설정한다.
또 사용 용도에 따라 기능 구현에 필요한 자동 구성을 오버라이딩한다.

JVM의 유연성 덕분에 여러 방법 중 하나인 디버그 플래그로 자동 설정 리포트를 간단히 생성한다.

포지티브 매치 항목을 나열하는 자동 설정 리포트는 `Positive matches`라는 제목으로 나열된다.
네거티브 매치는 스프링 부트의 자동 설정이 수앻아지 않은 동작과 이유를 보여준다.
조건을 충족하지 않고도 생성되는 `Unconditional classes`를 나용하는 부분은 유용한 정보가 된다.
