---
layout: post
title:  "Spring 테스트하기 @RunWith @ContextConfiguration"
date:   2019-03-25
excerpt: ""
tag:
- Spring
- RunWith
- ContextConfiguration
- TEST
---


# 1부 Spring 테스트하기
## @RunWith(SpringJUnit4ClassRunner.class)

SpringJUnit4ClassRunner가 스프링 통합테스트를 위한 환경을 제공 한다.
@ContextConfiguration 으로 지정된 ApplicationContext 설정파일을 읽어 싱글톤으로 생성한다. 따라서 Junit 이 @Test 케이스들을 수행 할 때 마다 새로운 오브젝트를 생성함에도 ApplicatoinContext는 한번만 로드된다.

특성상 @ContextConfiguration 와 쌍으로 붙어다니며, @ContextConfiguration 미선언시, @ContextConfiguration 를 추가해 보라며 에러가 난다.
```
Cannot load an ApplicationContext with a NULL 'contextLoader'. Consider annotating your test class with @ContextConfiguration or @ContextHierarchy.
```
> 디폴트로 라도 ApplicationContext 를 만들어줄줄 알았는데..

따라서 스프링 통합테스트를 진행하려면 이 두가지는 반드시 필요하니 쌍으로 기억하자.
- @Runwith(SpringJUnit4ClassRunner.class)
- @ContextConfiguration 

> 그래서 다른 러너를 쓰고싶은데 (예를들면 MockitoJUnitRunner) 빈팩토리를 써야할 상황에선 어떻게 해야할지 모르겠다.. 


## @ContextConfiguration
ApplicatoinContext 로드에 참조할 Configuration 파일을 지정할수 있다. (classes 또는 locations)
```
@ContextConfiguration(classes = {RootConfig.class})  
public class MyBlacklistControllerTest {
}
```
classes 방식과  locations 방식을 동시에는 쓸수 없다. 만약 둘다 쓰고자 한다면 @Configuration 클래스에서 xml을 @ImportResource로 추가 하여 classess 로 이용하면 된다.
```
@Configuration
@ImportResource(value = {"/spring/application-context.xml", "/spring/datasource-context.xml"})  
public class RootConfig {
	...
}
```
@ContextConfiguration 에 Config파일을 아무것도 지정하지 않는다면 xml로더나 annotationConfig로더가 참조할 파일이 없어 ApplicationContextInitializers 가 초기화 되지 않는 에러가 난다.
```
Neither GenericXmlContextLoader nor AnnotationConfigContextLoader was able to detect defaults, and no ApplicationContextInitializers were declared for context configuration 
```
