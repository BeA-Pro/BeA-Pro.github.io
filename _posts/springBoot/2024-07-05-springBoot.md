---
title: Chapter 4. 데이터베이스 액세스
author: BeAPro
date: 2024-07-04 15:20:00 +0900
categories: [Study,Spring Boot]
tags: [Spring Boot, JPA]
image:
  path: /assets/img/title-image/springBoot.png
  alt: SpringBoot
---

## **데이터베이스 액세스**

### DB 액세스를 위한 자동 설정 프라이밍
스프링 부트는 개발자가 계속해서 반복적으로 수행하는 코드와 사용 패턴의 80~90%를 최대한 단순화 하는 것을 목표로 하는 프레임워크이다.
사용 패턴을 식별하면, 적절한 기본 구성을 사용해 필요한 빈을 자동으로 초기화한다. 간단한 사용자 맞춤 기능으로는 사용 패턴에 따라 여러 속성값을 제공하거나 하나 이상의 맞춤형 빈을 제공하는 기능 등이 있다. 만약 자동 설정이 기존에 식별된 것이 아닌 새로운 변경 사항을 감지하면, 개발자의 지시에 따라 이후 과정을 처리한다. DB 액세스는 자동 설정 프라이밍의 완벽한 예이다.


### DB 의존성 추가하기

스프링 부트 애플리케이션에서 DB에 액세스하려면 다음 사항이 필요하다.

- 실행 중인 DB - 접속 가능한 DB이거나 개발하는 애플리케이션의 내장 DB
- 프로그램상에서 DB 액세스를 가능하게 해주는 DB 드라이버(일반적으로 DB 공급/관리 업체가 제공)
- 원하는 DB에 액세스하기 위한 '스프링 데이터' 모듈

H2는 자바로 작성도니 실행 속도가 빠른 DB로 JPA 기술 명세를 준수하기 때문에 마이크로소프트, SQL, MySQL, Oracle, PostgreSQL 같은 JPA를 사용할 수 있는 DB와 동일한 방식으로 애플리케이션을 연결한다. H2는 디스크 기반 영속성 DB로 변경하거나 JPA DB로 변경하는 것이 가능하다.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```
애플리케이션이 H2 DB와 상호작용하기 위해서는 프로젝트의 `pom.xml`의 `<dependencies>` 위치에 위 처럼 두 개의 의존성을 추가해야 한다.

> 컴파일 시 필요 없는 라이브러리에는 스코프를 <scope>runtime</scope>으로 설정하는 것이 좋다.
{: .prompt-tip}

### @Entity 사용하기

H2는 JPA-호환 DB이므로 JPA 어노테이션을 추가해 사용할 수 있다.

```java
@Entity
class Coffee{
	@Id
	private String id;
	private String name;

    public Coffee(){};
    ...

    public void setId(String id){this.id = id;}
}
```

커피는 영속 가능 엔티티(Persistable Entity)이므로 Coffee 클래스에 `@Entity` 어노테이션을 추가한다. 또 기존 id 멤버 변수를 DB 테이블의 ID 필드로 표시하기 위해 `@Id` 어노테이션을 추가한다.
JPA를 사용해 DB에 데이터를 생성할 때는 기본 생성자가 필요하므로 선언해주고, 기본 생성자를 사용하려면 모든 멤버 변수를 `final`이 아닌 변경가능하게 만들어야 된다.
`setId()`메서드를 만들어야 JPA가 이를 이용하여 id 값을 갱신할 수 있다.

> 기본적으로 JPA는 클래스에서 어떤 접근 방식을 사용할지 결정하기 위해 @Id 어노테이션이 어디에 있는지 확인한다.
> JPA가 사용하는 getter/setter는 반드시 자바빈 명명 규칙을 따라야 한다.
{: .prompt-info}

### 저장소 만들기

Coffee가 저장, 조회할 수 있는 유효한 JPA 엔티티로 정의했으니 DB에 연결하여 저장해주면 된다.
DB와 상호작용하거나 자바 유틸리티나 클라이언트 애플리케이션에서 직접 데이터 스토어에 액세스 하는 경우, DB 열기와 닫기 등의 작업을 수행할 PersistenceUnit, EntityManagerFactory, EntityManager API와 관련된 추가 단계를 수행하는 등 반복되는 작업을 수행한다.

이런 반복 작업을 해결하기 위해 스프링 데이터는 저장소(Repository) 개념을 도입하였다.
`Repository`는 스프링 데이터에 정의되어 다양한 DB를 위한 추상화 인터페이스이다.

```java
interface CoffeeRepository extends CrudRepository<Coffee, String> {};
```
위 표현식은 스프링 부트 애플리케이션 내의 간단한 표현식 중 하나이다.
스프링 부트의 자동 설정은 클래스 경로(이 경우 H2)의 DB 드라이버, 애플리케이션에 정의된 저장소 인터페이스, JPA 엔티티인 Coffee 클래스 정의를 고려해 사용자를 대신해서 DB 프록시 빈을 생성한다.

### DataLoader

```java
@Component
class DataLoader{
	private final CoffeeRepository coffeeRepository;
	public DataLoader(CoffeeRepository coffeeRepository){
		this.coffeeRepository = coffeeRepository;
	}

	@PostConstruct
	private void loadData(){
				this.coffeeRepository.saveAll(List.of(
				new Coffee("Cafe Cereza"),
				new Coffee("Cafe Ganador"),
				new Coffee("Cafe Lareno"),
				new Coffee("Cafe Tres Pontas")
		));
	}

}
```

### Get

```java
@GetMapping
Iterable<Coffee> getCoffees(){
    return coffeeRepository.findAll();
}
@GetMapping("/{id}")
Optional<Coffee> getCoffeeById(@PathVariable String id){
    return coffeeRepository.findById(id);
}
```
> Optional은 Java 8에서 도입된 클래스로, 주로 메서드의 반환 값이 null일 수 있음을 명시적으로 표현하기 위해 사용된다.
{: .prompt-info}

### Post
```java
@PostMapping
Coffee postCoffee(@RequestBody Coffee coffee){
    return coffeeRepository.save(coffee);
}
```

### Put
```java
@PutMapping("/{id}")
ResponseEntity<Coffee> putCoffee(@PathVariable String id, @RequestBody Coffee coffee){
    return (!coffeeRepository.existsById(id))
    ? new ResponseEntity<>(coffeeRepository.save(coffee), HttpStatus.CREATED)
    : new ResponseEntity<>(coffeeRepository.save(coffee), HttpStatus.OK);
}
```


### Delete
```java
@DeleteMapping("/{id}")
void deleteCoffee(@PathVariable String id){
    coffeeRepository.deleteById(id);
}
```