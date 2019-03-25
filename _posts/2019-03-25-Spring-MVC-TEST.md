---
layout: post
title:  "Spring MVC TEST"
date:   2019-03-25
excerpt: ""
tag:
- spring
- MVC
- TEST
- ControllerTest
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
-  @ContextConfiguration 
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

---

# 2부 Spring-MVC 테스트하기
 1부에서 2개의 클래스레벨 어노테이션을 추가하여  스프링 통합테스트를 할수 있었다.
- @RunWith(SpringJUnit4ClassRunner.class)  
- @ContextConfiguration(classes = {RootConfig.class})

하지만 위 설정에서는 서블릿 구성이 없다. Mvc 테스트를 하려면 서블릿 구성이 필요할텐데 어떻게 테스트 할수 있을까? 단순히 WebConfig를 추가한다고 될까?
```
Caused by: java.lang.IllegalArgumentException: A ServletContext is required to configure default servlet handling
```
WebConfig는 단지 @Controller 빈과 DispatcherServlet에 쓰일 빈들에 대한 설정만 있을테니 서블릿설정과는 무관하다. 이것을 해결할수 있는 Mock이 있다.
> 컨트롤러 빈을 사용해야만 하는 테스트를 위해 WebConfig를 추가해야 될 상황도 있다. 이때는 @WebAppConfiguration을 추가해야하는데 아래서 설명하겠다.


## Spring MockMvc
스프링은 MockMvc를 이용해서 서블릿이 있는것처럼 테스트 할수 있게 해준다. 

### MockMvc 초기화
MockMvc를 초기화 하기 위해선 아래 2가지중에 선택할수 있다.
1. standaloneSetup
SUT Controller를 목으로 만들어 준다. *단위테스트* 에 적합.

2. webAppContextSetup
실제 webApplicationContext 를 주입해서 내 wac 이용하기.
빈팩토리에 있는것들을 참고할수 있으므로 *통합테스트* 에 적합.

```
@Before  
public void setup(){
	// 아래 2개중 택1  
    mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();  
	// mockMvc = MockMvcBuilders.standaloneSetup(new LoginController()).build();  
}
```

### MockMvc를 이용한 테스트 코드
MockMvc 를 초기화 했다면 이것을 이용하여 컨트롤러와 메세지를 주고받을수 있게 된다.
```
@Test  
public void insert() throws Exception {  
   mockMvc.perform(get("/login/loginPage"))  
         .andExpect(status().isOk())  
         .andDo(print());  
}
```
> status() 외에 다른 어설션들이 많은데 그것은 다음기회에

## MockMvcBuilders.standaloneSetup 자세히
SUT Controller의 객체를 목객체로 만들어주는 방식이다. 
```
mockMvc = MockMvcBuilders.standaloneSetup(new LoginController()).build();  
```
그런데 보통 Controller는 Service 를 의존하고 있는데 이런 의존성이 전혀 없는 상태에서 시작하기 때문에 의존성 주입을 직접 해줘야 한다.
> 물론 SUT 메소드가 다른 클래스를 의존하는게 없다면 고민하지 않아도 된다.

만약 생성자 주입이 가능하다면 이렇게 될것이다.
```
mockMvc = MockMvcBuilders.standaloneSetup(new LoginController(service)).build();  
```

그런데 보통은 `필드 자동와이어링`을 걸기 때문에 의존성 주입을 Mockito 같은 프레임워크에 의지하기도 한다. @Mock과 @InjectMocks 을 이용할수 있게되는데 설명하자면 다음과 같다.
```
@RunWith(MockitoJUnitRunner.class)  // 러너 변경
@ContextConfiguration(classes = {RootConfig.class})  
public class MyBlacklistControllerTest {    
  MockMvc mockMvc;
  
  @Mock // 목객체 생성
  TService tService;  
  
  @InjectMocks // 목객체 주입  
  LoginController loginController;    
  
  @Before  
  public void setup(){  
	  mockMvc = MockMvcBuilders.standaloneSetup(loginController).build();  
  }  
  
  @Test  
  public void insert() throws Exception {  
      mockMvc.perform(get("/login/test"))  
            .andExpect(status().isOk())  
            .andDo(print());  
  }  
}
```
Mockito는 유용하지만 프레임워크도 알아야 한다. 그래서인지 요즘은 생성자 주입 방식을 선호하는 분들도 더러 있다. (Intelij도 그렇다)


## MockMvcBuilders.webAppContextSetup 자세히
내가 지정한 WebApplicationContext를 이용하여 만들어내는 방식이다.
```
mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
```

WebApplicationContext 를 빈으로부터 받아내려면 2가지가 필요하다.
1. @ContextConfiguration(classes = {RootConfig.class,**WebConfig.class**})

2. **@WebAppConfiguration**

아까 1번만 해봤을때 서블릿구성이 없다는 에러가 났었다. 2번이 그것을 해결해 주는것이다. 


### @WebAppConfiguration
@ContextConfiguration 가 ApplicationContext를 로드할때, WebApplicationContext 으로 로드 되게 해준다.
```
@Autowired  
WebApplicationContext wac;   // 이제 WebApplicationContext 사용 가능하다.
```


### 결론.
스프링 MVC테스트를 위해 다음과 같이 알아보았다.
1. 스프링 러너 등록 
- @RunWith(SpringJunit4Runner.class)

2. 어플리케이션 컨텍스트 등록
- @ContextConfiguration(classes = {RootConfig.class})
	
3. MockMvc 를 초기화 방법 2가지
- mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
(WebApplicationContext 필요)
- mockMvc = MockMvcBuilders.standaloneSetup(loginController).build();
(의존성 stub 처리 필요)

4. 서블릿 컨텍스트 추가 등록 및 등록시 필요한 @WebAppConfiguration 
- @ContextConfiguration(classes = {RootConfig.class, WebConfig.class})
- @WebAppConfiguration


다 역할이 있어 테스트범위를 어떻게 잡느냐에 따라 사용 여부가 다르다.
1,2번은 어플리케이션컨텍스트 이용하려면 필요.
3번은 MVC테스트시 필요.
4번은 WebApplicationContext 를 직접 입력해서 통테할때 필요.

테스트 수행시간이 중요치 않다면 다 등록해도 돌아는 갈것이다.



