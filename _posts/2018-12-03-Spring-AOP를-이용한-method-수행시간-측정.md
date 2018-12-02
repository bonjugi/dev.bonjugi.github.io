---
layout: post
title:  "Spring AOP를 이용한 method 수행시간 측정"
date:   2018-12-03
excerpt: "Aspect Oriented Programming 에 대해서 간략히 알아보고 Spring AOP에 대해서 알아본 후 이를 이용하여 method 수행시간 측정 코드를 알아봅니다."
tag:
- spring
- aop
- arround
- 
comments: true
---

Aspect Oriented Programming (관점지향 프로그래밍) 이라 하며 (이하 AOP)
Inversion of Control 과 함께 Spring 핵심기술중 하나 입니다.
무려 핵심 기술이라는 칭호를 갖고 있음에도 특이한 동작 방식과, 어려운 용어 때문에 나와 관계없는것이라 생각하고 관심을 두지 않았었습니다.
이번 기회에 시간이 여유가 있어 정리 해 보고자 합니다.
>[JIHUN님 블로그](https://hunit.tistory.com/188)
>[전자정부프레임워크](http://www.egovframe.go.kr/wiki/doku.php?id=egovframework:rte:fdl:aop)

### AOP 에 대해 한줄로 요약하면? 
산재된 공통 관심사 `Cross-Cutting Concern` 를 비즈니스 로직 `Primary Concern` 으로부터 분리하여 모듈화 `Aspect` 한다.
- ![Image Alt 텍스트](/assets/img/upload/aop.png)
> Service들로부터 Security, Transaction, Other 를 분리 해낸 모습 입니다.
> 횡단으로 분리하는 모습이라 횡단관심사 라고도 합니다.

    
JAVA 에서는 이것의 구현체로 AspectJ 와 Spring AOP가 있습니다.
AspectJ 에 비해 Spring AOP가 좀더 경량화 되고 쓰기 쉽다고 이해하면 됩니다. (코드가 경량화 되었다는것은 아니고 설정이 일정부분 강제되어있어 자유도가 적지만 쓰기쉽다 입니다.) 
저는 Spring AOP에 대해서만 다룰 예정입니다.

### AOP 용어 설명
AOP 근간을 알기 위해 용어를 먼저 정리해야 합니다. @Transactional 에 비유하여 설명해 보겠습니다.
완벽히 숙지할 필요는 없습니다. 저도 며칠 후면 모를것 입니다.
- Aspect : 분리되는 관심사와 시점 지점 등등 AOP 관점에 의해 분리된 관점 그 자체로 이해해도 됩니다. @Transactional 에 비유하자면 그 자체로 보면 됩니다.
- Join Point : 관점(Aspect)를 삽입하여 실행 가능한 어플리케이션의 특정 지점을 말 합니다. @Transactional을 예로들자면, Method 가 실행 될때 입니다. 
- Pointcut : 어떤 결합점을 사용할 것이지를 결정하기 위해 패턴 매칭을 이용하여 룰을 정의 합니다. @Transactional을 예로 들자면 `Aspect를 넣기 위해 DAO 라는 이름으로 끝나는 모든 Class 를 대상으로한다` 정도가 될것 같습니다.
- Advice : Join Point 시점에 (또는 Join Point 전,후 등등 시점에) 삽입될 코드 입니다. 비유하자면 begin close commit 등등의 코드들이 되겠습니다.
- Target : 충고받는 객체 (이래라 저래라 들을 불쌍한 객체 ㅠ.ㅠ) @Transactional 충고를 듣는 DAO객체가 있겠습니다.
- Weaving : 관점들을 어떤 시점에 엮을지 입니다. 컴파일시, 클래스로드시, 런타임시 등이 있습니다.

Join point가 좀 어렵습니다. 메소드 실행시점 뿐만 아니라 생성자 생성시, 필드에서 값을 가져갔을시 등등이 있습니다.
하지만 우리가 알아야할 Spring AOP 에서의 Join Point는 메소드 실행시점 뿐 입니다.

### Spring AOP
스프링의 경우 Proxy Bean을 생성하여 그것을 호출하는 개념 입니다.
순서는 다음과 같습니다. 
1. 컴파일, 클래스로드 시점을 모두 지나고
2. 런타임에서 Bean을 생성하고 (이후로는 Proxy의 대상이 되는 Bean 이므로 이후 Target Bean 이라고 하겠습니다.)
3. Advice 를 래핑하여 Proxy Bean을 만듭니다.
4. 소비자는 Target Bean을 직접 호출하는것이 아니라 Advice가 포함된 Proxy Bean을 호출하게 됩니다.


#### AspectJ 와 비교시 Spring AOP의 단점 
이미 컴파일 시점을 지나버리기 때문에 Weaving이 런타임시점으로 강제되며, 
Target Bean 을 상속받아야 하는 Proxy Bean은 Target Bean의 시그니처를 벗어나는 동작을 할수가 없으므로 메소드 전후로 래핑하는 것 밖에 없습니다.
(그래서 대상 메소드는 반드시 public 이어야 합니다.)
그래서 아쉽게도 Lombok 같은 에디터 어드바이스나 pre-compile 어드바이스가 불가능 합니다.
또 프록시빈 생성 과정이 추가되기 때문에 최초 빈이 생성될때 약간의 부하가 추가됩니다. (..만, 빈생성이 잦은건 아니니까 무시해도 됩니다)

#### AspectJ 와 비교시 Spring AOP의 장점
하지만 별도의 설정이 없으며 위 용어들과 개념을 무시해도 될만큼 설정이 편하다는 장점이 있습니다.
즉 Spring AOP 에서 처리하는 모든 Join Point는 메소드 실행 시점이라고 봐야하고, Point cut은 어떤 메소드 실행시점인지 에 대한것으로 봐야합니다. 개념이 간소한 편 입니다.
 
 

### Spring AOP 에서의 사용법 
먼저 @Aspect 와 @Component 두개가 필요합니다.
내가 @Aspect다 라고 명시할수 있는 @Aspect 안에 @Component가 포함되어있었으면 좋았을텐데...
아무튼 2개 다 추가 해 줍시다.
```java
@Component
@Aspect
public class AspectUsingAnnotation {
}
```
#### @PointCut
아래 코드의 @Pointcut 에 명시된 anyPublicOperation 처럼 대상 메소드를 지정할수 있습니다.
tradingOperation() 안에는 Advice할 코드를 구현하면 됩니다.
 
```java
@Component
@Aspect
@Slf4j // lombok
public class AspectUsingAnnotation {
    @Pointcut("anyPublicOperation()")
    private void tradingOperation() {
    	// advice..
    }
    
}
``` 
> 드디어 PointCut 이라고 명시하는 코드가 나왔습니다. 
> PointCut 은 표현식이기 때문에 여기서 모두 설명하기 어려우므로 아래 링크의 `포인트컷 정의 예제` 를 참고 해 주세요. 
> [전자정부프레임워크](http://www.egovframe.go.kr/wiki/doku.php?id=egovframework:rte:fdl:aop:aspectj)



#### @PointCut의 상위 호환. @Arround. 하지만 개념적으로는 그냥 PointCut
Join Point가 메소드 실행시점밖에 없는 Spring AOP 에게 감사하게도 메소드실행 전(Before),후(After),전+후(Arround) 에 Advice할수 있는 기능 이 있습니다.
@PointCut 의 상위 호환으로 보시면 됩니다.
순서는 다음과 같습니다.
- @Before
- @Around (대상 메소드 수행 전)
- 대상 메소드
- @Around (대상 메소드 수행 후)
- @After(finally)
- @AfterReturning

실행시간을 체크하기 위해 Arround를 이용하여 Target method의 원래 기능의 앞뒤로 코드를 작성하여 원하는 기능을 구현할수 있습니다.

```java
@Aspect
public class AspectUsingAnnotation {
    @Around("targetMethod()")
    public Object aroundTargetMethod(ProceedingJoinPoint thisJoinPoint) throws Throwable {
        System.out.println("AspectUsingAnnotation.aroundTargetMethod start.");
        long time1 = System.currentTimeMillis();
        Object retVal = thisJoinPoint.proceed();
 
        System.out.println("ProceedingJoinPoint executed. return value is [" + retVal + "]");
 
        retVal = retVal + "(modified)";
        System.out.println("return value modified to [" + retVal + "]");
 
        long time2 = System.currentTimeMillis();
        System.out.println("AspectUsingAnnotation.aroundTargetMethod end. Time(" + (time2 - time1) + ")");
        return retVal;
    }
}
```

### PointCut 대상을 어노테이션으로 사용하기
PointCut 표현식에는 @annotation 을 이용하여 특정 어노테이션을 구현한 메소드를 대상으로 지정할수 있습니다.
먼저 custom annotation을 만든 후,
```java
// 아래 어노테이션을 넣어야 하는 이유는 AOP 의 내용이 아니므로 나중에 별도로 정리하여 포스팅 하겠습니다.
@Retention(RetentionPolicy.CLASS)
@Target(ElementType.METHOD) 
public @interface ElapseTimeLogging {
}
```
아래처럼 PointCut에 어노테이션 이름을 명시 해주면 됩니다.
```java
@Aspect
public class AspectUsingAnnotation {
    @Around("@annotation(ElapseTimeLogging)")
    public Object aroundTargetMethod(ProceedingJoinPoint thisJoinPoint)  throws Throwable {
        // advice..
    }
}
```

