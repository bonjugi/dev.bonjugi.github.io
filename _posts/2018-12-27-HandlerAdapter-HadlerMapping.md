---
layout: post
title:  "HandlerAdapter와 HandlerMapping 종류 및 특징"
date:   2018-12-27
excerpt: ""
tag:
- spring
- HandlerMapping
- HandlerAdapter
---


개념상 FrontController 라 불리며, 스프링에서는 DispatcherServlet 구현체로 존재한다.
DispatcherServlet은 요청을 각각의 역할에 맞는 Controller 로 배분하는 기능을 담당하고 있다.



Dispatcher 서블릿에 등록되는 컨트롤러는 여러가지 타입으로 등록될수 있기 때문에 여러개의 어댑터가 존재한다.

### HandlerAdapter


#### 1. Servlet 타입의 컨트롤러를 호출할수 있는 SimpleServletHandlerAdapter
	.extends HttpServlet으로 서블릿을 직접 구현했을때 사용할수 있는 어댑터.
	.모델과 뷰를 리턴하지 않는다. MVC의 모델과 뷰라는 개념을 알지 못하는 표준 서블릿을 그대로 사용했기 때문.
	.그래서 결과를 그냥 HttpServletResponse 에 setAttribute 하는 방식으로 되어있다.
#### 2. HttpRequestHandler 와 HttpRequestHandlerAdapter
	.전형적인 서블릿 스펙을 준수할 필요 없이 HTTP 프로토콜을 기반으로 한 전용 서비스를 만들려고 할때 사용
	.RMI를 구현할수 있다.
	.모델과 뷰 개념이 없는 HTTP 기반의 RMI와 같은 로우레벨 서비스를 개발할수 있다는 사실정도만 기억하고 넘어가자.
#### 3. Contrller 타입의 컨트롤러를 호출할수 있는 SimpleControllerHandlerAdapter
	.Spring 3.1 등장 이후 @RequestMapping 과 같은 어노테이션 기반이 본격적으로 등장하기 전에 주로 사용되었음.
	.Controller Interface를 구현하기만 하면 되는 강력한점이 있고 상속하여 구현하기만 된다
	.하지만 직접 상속하여 구현하기 보다는, AbstractController 라는, 웹에서 주로 사용되는 기능들이 이미 구현되어있는 상위 클래스가 있다. 이것을 상속하여 구현하는것을 권장한다.
#### 4. 모든 타입의 컨트롤러를 호출할수 있는 AnnotationMethodHandlerAdapter (중요)
	.모든 타입의 컨트롤러를 사용할수 있다.
	.어노테이션으로 매핑할수 있고, 덕분에 메소드 단위로 매핑이 가능하여 클래스 하나에 여러개를 매핑할수 있다.
	.다른 어댑터처럼 HandlerMapping을 자유롭게 등록해 사용할수 있는것과 달리, 반드시 DefaultAnnotationHandlerMapping 을 등록해야 한다. 둘다 동일한 어노테이션을 사용하기 때문이다.
	.@RequestMapping 어노테이션을 사용할수 있다. 현재 MVC 패턴에서 가장 많이 쓰인다.
	.어노테이션이 많은것을 제외하면 가장 코드가 단순해진다.

### HandlerMppaing


#### 1. BeanNameUrlHandlerMapping


#### 2. ControllerBeanNameHandlerMapping
#### 3. ControllerClassNameHandlerMapping
#### 4. SimpleUrlHandlerMapping
#### 5. DefaultAnnotationHandlerMapping








3.1 이후, 아래의 매핑과 어댑터는 변경이 있으나 큰 차이점은 없는것 같다(정확히는 모르겠다) 
	> 참고. http://wonwoo.ml/index.php/post/1582
DefaultAnnotationHandlerMapping -> RequestMappingHandlerMapping
AnnotationMethodHandlerAdapter -> RequestMappingHandlerAdapter