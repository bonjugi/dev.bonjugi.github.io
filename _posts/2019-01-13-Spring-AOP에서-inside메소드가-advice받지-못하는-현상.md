---
layout: post
title:  "Spring AOP에서 inside메소드가 advice받지 못하는 현상"
date:   2019-01-13
excerpt: ""
tag:
- spring
- AOP
- inside
- advice
---




### 이슈
Spring AOP 가 적용된 메소드 호출시 Advice가 적용되게끔 설정했으나,
호출된 메소드 내부에 또다른 메소드가 있을시 (이하 inside 메소드), inside 메소드는 Advice 되지 않는점.


### 원인
내부 메소드는 프록시빈이 아니기 때문. 


### 설명
Spring aop는 스프링빈 이 만들어 진 이후 해당 빈을 Advice가 포함된 프록시를 감싸서 만들어 낸다. (따라서 적용 대상은 반드시 Spring 빈으로 등록되어 있어야 한다는 특징도 있다) 
먼저 Advice 받을 빈을 만들자. (포인트컷은 생략. Advice는 그냥 메소드 호출 앞뒤로 로깅찍는것이라고 가정 하겠다.)

```java
@Service
public class Service {	
	public void temp(){
		inside();
	}
	public void inside(){

	}
}
```

이때 아래와 같이 service 인스턴스를 주입받을시, AOP 설정에 의해 프록시빈 proxy$Service가 주입될 것이다.
temp() 호출시에도 로깅이 찍히고, temp()가 호출하는 inside() 도 로깅이 찍히기를 기대 한다.

```java
public class Controller {
	@Autowired 
	Service service;		// 스프링 AOP는 프록시 기반 이므로, Service 클래스의 빈이 만들어질때 실제로는 proxy$Service 라는 프록시 인스턴스가 주입된다.

	public void test() {
		service.temp();
	}
}
```

하지만 결과는 temp()만 Advice가 적용되고, inside()는 아무런 로깅이 찍히지 않는다.
원인은 inside()는 proxt$Service 를 통해서 호출된것이 아니라 순수 자바 인스턴스에 의해서 동작하기 떄문에 AOP가 적용 되어있지 않은 것이다.
프록시 빈이 주입되는 과정은 다음과 같다.

1. Component scan 단계에서, Service 클래스를 바탕으로 service 평범한 스프링 빈이 생성됨. 
2. 위와 같은 시점에 @Autowired 같이 주입이 필요한 빈도 스프링 빈으로 주입됨.
3. AOP 설정이 있는경우, 포인트컷에 해당하는 스프링 빈 을 찾아서 프록시로 감쌈. (service -> proxy$Service 로 바뀜)
4. 주입된 service 는 proxy$Service 를 가르키게 됨.


콜스택은 아마도 이럴것이다 (까보진 못했지만..)
1. temp() 호출 과정.
	> proxy$Service.temp() -> service.temp() 	// 프록시 temp()가 원본 temp() 메소드 전후에 어드바이스 가능
2. inside() 호출 과정
	> service.temp() -> this.inside()			// 원본 temp()는 어드바이스가 불가능


따라서 inside() 도 프록시 빈으로 호출하면 된다.
1. temp() 호출 과정.
	> proxy$Service.temp() -> service.temp() 						// 프록시 temp()가 원본 temp() 메소드 전후에 어드바이스 가능
2. inside() 호출 과정
	> service.temp() -> proxy$Service.inside() -> service.insde()	// 원본 temp()는 어드바이스가 불가능


바뀌는 소스코드로는 다음과 같다.
```java
@Service
public class Service {	
	@Autowired 
	Service service;

	public void temp(){
		service.inside();		// proxy$Service
	}
	public void inside(){

	}
}
```


