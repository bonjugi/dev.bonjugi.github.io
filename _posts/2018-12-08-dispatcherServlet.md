---
layout: post
title:  "dispatcherServlet 에 대해서"
date:   2018-12-07
excerpt: ""
tag:
- spring
- dispatcherservlet
---



 
### HTTP 요청과 응답을 DispatcherServlet이 처리하는 순서

1. DispatcherServlet 의 HTTP 요청 접수
2. DispatcherServlet (프론트 컨트롤러)에서 세부 컨트롤러로 HTTP 요청 위임
3. 컨트롤러의 모델 생성과 정보 등록
4. 컨트롤러의 결과 리턴: 모델과 뷰
5. DispatcherServlet의 뷰 호출과 6. 모델 참조
7. HTTP 응답 돌려주기

프로젝트 초기 구성시 스프링 설정을 하는사람이 아닌 이상, 컨트롤러/모델/뷰 생성단계인 3,4 단계의 역할만 하면 되기 때문에
나머지 단계에서 어떻게 처리 되는지 관심갖기가 어렵고, 애써 배우더라도 실습할 기회가 없기때문에 금새 잊어버리기 십상입니다.
그 나머지 단계에서의 역할은 모두 DispatcherServlet에서 담당하고 있기 때문에 DispatcherServlet 을 집중적으로 분석하여 
스프링에서는 어떻게 request 와 response가 이루어 지는지 알수 있도록 합시다.



#### 1. DispatcherServlet 의 HTTP 요청 접수
web.xml에 servlet-mapping 패턴을 기록하면, 사용자 http 요청시 요청된 URI 와 매치되는 경우 해당 요청을 접수하게 됩니다.
servlet-mapping 패턴은 /app/* 처럼 모든 확장자를 적을수도 있고, *.do 같이 특정 확장자만 적을수도 있습니다.
```xml
	<servlet-mapping>
		<servlet-name>MyAdmin</servlet-name>
		<url-pattern>/app/*.do</url-pattern>
	</servlet-mapping>
```



#### 2. DispatcherServlet에서 컨트롤러로 HTTP 요청 위임 
HandlerMapping 전략으로 어떤 URL이 들어오면, 어떤 컨트롤러 오브젝트(핸들러 오브젝트 라고도 한다) 가 처리할지 매핑하고,
HandlerAdapter 전략으로 모든 타입의 컨트롤러 오브젝트의 메소드를 호출하는 단계로 나눌수 있습니다.
이 외에도 ViewResolver 전략, HandlerExceptionResolver 전략 등 많은 전략이 있습니다.

이러한 전략들은 실제로 전략패턴으로 적용되어 있어서 해당 전략을 얼마든지 DI로 확장할수 있기 때문입니다.
```xml
	<bean id="requestMappingHandlerAdapter"
		class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
		<property name="webBindingInitializer" ref="webBindingInitializer" />
		<property name="messageConverters" ref="messageConverters" />
	</bean>
```

따라서 각각의 전략들은 여러가지 구현체들이 이미 존재하고 있으며 동시에 여러개를 등록할수가 있습니다.
예를들어 HandlerMapping과 HandlerAdapter 는 [HandlerAdapter와 HandlerMapping 종류 및 특징](https://bonjugi.github.io/HandlerAdapter와-HandlerMapping-종류-및-특징) 
처럼 많이 있지만, 그중에서도 디폴트이며 가장 대중적으로 사용되는 쌍으로는, @RequestMapping 을 이용하여 어노테이션 기반으로 URL을 매핑할수 있는 RequestMappingHandlerMapping 와 RequestMappingHandlerAdapter 가 있습니다.


#### 3. 컨트롤러의 모델 생성과 정보 등록
모델은 그냥 맵이라고 생각하면 쉽습니다. 이름도 ModelMap 으로 되어져 있습니다.
key value 형태로 작성이 가능하고 컬렉션도 가능합니다.

> 컨트롤러가 하는 순차적 작업
> 1. 사용자 요청 해석
> 2. 비즈니스 로직 수행하도록 서비스계층에게 위임
> 3. 결과를 받아 모델로 생성    < Model
> 4. 엑셀, PDF, html, json 등 어떤 뷰로 response 할지 결정 < View

#### 4. 컨트롤러의 결과 리턴: 모델과 뷰
컨트롤러가 뷰 오브젝트를 직접 리턴할수도 있지만, 보통은 논리적인 이름을 리턴해주면 dispatcherServlet 에서 뷰 리졸버가 이를 이용해 뷰 오브젝트를 생성 해 줍니다.
대표적으로 JSP/JSTL 뷰가 있습니다. JSP 파일로 만들어진 뷰 템플릿과 JstlView 클래스로 만들어진 뷰 오브젝트를 HTML로 만들어줍니다. 
잘 아시겠지만 이 경우에 컨트롤러에서 리턴해야 할 논리적 이름은 JSP 파일 네임 이죠.
아무튼 생성된 뷰 오브젝트와, 위 3번에서 생성된 모델정보를 ModelAndView로 dispatcherServlet에게 리턴 하게 됩니다.


#### 5. DispatcherServlet의 뷰 호출과 6. 모델 참조
컨트롤러로부터 리턴된 모델과 뷰를 받아서 뷰오브젝트에게 모델을 전달해주고 최종 결과물을 HttpServletResponse에 담습니다.
어떤 뷰 오브젝트가 선택될지는 ViewResolver 전략에 의해 결정됩니다.
디폴트 ViewResolver는 InternalResourceViewResolver 입니다. 

#### 7. HTTP 응답
뷰 생성까지 모든작업을 마쳤으면 DispatcherServlet은 별도로 후처리기가 등록된것이 있는지 확인 하고 처리한 후, 
HttpServletResponse 에 담긴 최종 결과물을 톰캣에 건네줍니다. 해당 정보를 바탕으로 톰캣은 브라우저 와 같은 클라이언트에게 결과를 전송하고 마칩니다.
   

 
### Servlet 컨테이너 가 생성하고 관리하는 오브젝트 이다.
스프링 컨텍스트에서 관리하는 오브젝트가 아닌 톰캣이 생성하고 관리하는 오브젝트 입니다.
따라서 직접 DI는 불가능하지만 DispatcherServlet 내부에 웹 어플리케이션 컨텍스트이 있고, 
이 컨텍스트로부터 개발자가 추가하거나 설정을 수정한 전략이 담긴 오브젝트가 있다면 이를 가져와서 디폴트 설정을 대신하는 구조 입니다.



### DispatcherServlet 의 전략은 무수히 많고 앞으로도 추가 될 것이다.
DI를 통해 계속 확장이 가능하고 세밀하게 제어가 가능하지만, 이미 전략들은 무수히 많아 ViewResolver 전략만 해도 Excel, PDF, HTML, Json, XML 등 매우 다양합니다.
HandlerMapping, HandlerAdapter 전략도 발전해 오면서 여러가지가 있습니다. (최신것을 선호하긴 하지만..) 
앞으로도 지속적으로 전략들이 추가될것으로 예상해볼수 있습니다.
 
### 무수히 많은 전략이 있더라도 디폴트가 존재하기 때문에 걱정없다.
대체로 선호되는 전략들이 디폴트로 설정되어 있습니다.
또한 각 설정들을 한번에 처리해주는 `<mvc:annotation-driven />` 같은 묶음 태그도 있습니다. 


