---
layout: post
title:  "DispatcherServlet url-mapping 이 / 으로 된 이유"
date:   2018-12-07
excerpt: ""
tag:
- spring
- servlet-mapping
- default-servlet
- InternalViewResolver
---




#### `/` 인가 `/*` 인가?
통상적으로 RESTful 하게 url을 작성하기 위해서는 필연적으로 `/` 으로 매핑 패턴을 변경해야합니다.
`/*` 으로 생성해야 해야하는것이 아닌가 싶기도 하지만 `/` 를 써야 하는데는 다 이유가 있었습니다.
url 패턴을 보고 매핑을 담당하는 web.xml 에 대해서 알아보겠습니다. 


#### `/*` 를 쓰면 안되는 이유

##### web.xml 은 2종류 
web.xml 종류는 어플리케이션의 web.xml과 서블릿컨테이너(톰캣)의 web.xml 두가지가 있습니다.
어플리케이션 web.xml은 프로젝트 생성시, 톰캣의 web.xml은 TOMCAT_HOME/conf/web.xml 에 있습니다.
Tomcat (서블릿 컨테이너) 의 web.xml 에는 default servlet과 jsp servlet 이 내장되어 있습니다. 


```xml
<!-- 톰캣의 web.xml -->
<servlet> 
    <servlet-name>default</servlet-name> 
    <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
    ... 
    <load-on-startup>1</load-on-startup> 
</servlet>
<servlet-mapping> 
    <servlet-name>default</servlet-name> 
    <url-pattern>/</url-pattern> 
</servlet-mapping>


<servlet>
    <servlet-name>jsp</servlet-name>
    <servlet-class>org.apache.jasper.servlet.JspServlet</servlet-class>
    ...
    <load-on-startup>3</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>jsp</servlet-name>
    <url-pattern>*.jsp</url-pattern>
    <url-pattern>*.jspx</url-pattern>
</servlet-mapping>
```



````xml
<!-- 어플리케이션의 web.xml -->
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/*</url-pattern>
</servlet-mapping>
````

#### 어플리케이션의 web.xml 의 매핑 패턴을 `/*` 으로 적용하게 되면 `*.jsp` 매핑 패턴을 덮게 된다.
mvc1 구조와 같이 http://*.jsp 와 같이 직접 파일을 호출하여 랜더 하는것을 기억할지 모르겠습니다.
 *.jsp 파일이 렌더하는 기능은 없습니다. jsp servlet 이 내장된 엔진을 통해 렌터해주는 것 입니다.

/ 보다는 *.jsp 가 매핑 조건이 까다로워 *.jsp가 우선순위를 갖지만, /* 와 *.jsp 는 조건이 같아 *.jsp 를 덮게 됩니다.
> 매핑조건이 까다로운게 우선됩니다.
> 매핑조건이 같다면 어플리케이션 서블릿에 있는 url mapping 이 우선됩니다.

결국  http://*.jsp 을 호출하면 DispatcherServlet 에 먼저 매핑되면서 *.jsp 를 매핑하는곳이 없으므로 404가 뜨게 됩니다.
```text
20-Dec-2018 17:50:46.321 경고 [http-nio-8080-exec-10] org.springframework.web.servlet.PageNotFound.noHandlerFound No mapping found for HTTP request with URI [/WEB-INF/views/index.jsp] in DispatcherServlet with name 'dispatcher'
```


#### `*.jsp` 파일패스 호출이 아닌, `/hello` 와 같은 컨트롤러 매핑으로 호출하는건 영향이 없는것인가?
그렇지 않습니다. 아래와 같이 /hello 를 매핑한 컨트롤러가 있다고 가정 하겠습니다.
```java
@Controller
public class MyHelloController {
	@RequestMapping("/hello")
	public String hello(ModelMap modelMap){
		System.out.println("컨틀롤러 접근!");
		return "index";
	}
}
```
디폴트 뷰 리졸버인 InternalViewResolver 를 일반적으로 사용하는데, (또는 JstlViewResolver 또는 TilesViewResolver)
이 리졸버는 컨트롤러에서 만든 Model 과 함께 InternalResourceView를 DispatcherServlet에 넘겨 줍니다.
DispatcherServlet 받은 모델앤드뷰를, /WEB-INF/views/index.jsp 로 foward 를 하는 구조 입니다. (JSP 가 요즘 등한시 되는 이유인데 나중에 별도로 포스팅 하겠습니다.)
즉 위와 관련된 뷰를 사용하는 컨트롤러 에서는 결국 *.jsp 가 매핑되어 있어야 jsp servlet으로 렌더가 가능한 것 입니다.
실제로 톰캣의 *.jsp를 주석시, http://*.jsp 패턴으로 접근이 불가한것은 당연하고, http://~/hello 로 접근하더라도 `컨트롤러 접근!` 로그까지만 찍힌뒤, 
매핑을 찾을수 없다는 에러가 발생하면서 브라우저는 404를 리턴하여 위 에러를 다시 만나게 됩니다.
```text
// 다시만난 에러메세지..
20-Dec-2018 17:50:46.321 경고 [http-nio-8080-exec-10] org.springframework.web.servlet.PageNotFound.noHandlerFound No mapping found for HTTP request with URI [/WEB-INF/views/index.jsp] in DispatcherServlet with name 'dispatcher'
```

결론은 /* 패턴을 사용하게 되면 톰캣에 내장된 jsp servlet을 사용할수 없게 되므로 하지 말아야 합니다.
 

#### `/` 을 사용할수 있게 해주는 `<mvc:default-servlet-handler/>`
 / 패턴은, 톰캣의 web.xml 에도 있고 default 서블릿에도 있습니다.
하지만 매핑조건 우선순위 선정에서 어플리케이션 서블릿에 있는 매핑이 우선시 되므로 `/` 하위의 모든 요청은 DispatcherServlet 을 거쳐가게 됩니다. (/ 하위는 사실상 모든요청 이죠)
default 서블릿 에 대해서 잠깐 얘기하자면, 어디에도 매핑되지 않은 패턴을 처리하는 종착점 입니다.
이것을 DispatcherServlet 이 모두 가로채게 되면서 컨트롤러매핑이 되지 않는 *.js *.css *.png 등등 확장자를 가진 리소스 모두를 처리하지 못하게 됩니다.
원래라면 default servlet 이 담당해야 할 일을 할줄도 모르는 녀석이 다 가로채고 있는 꼴 입니다.


`<mvc:default-servlet-handler/>` 을 이용하면 DispatcherServlet 의 url mappping을  `/*` 으로 해도 됩니다.
아래와 같은 동작원리를 지니게 됩니다.


1. 모든 처리를 DispatcherServlet 이 받아낸다. 
같은 URL 매핑인 경우 톰캣의 서블릿보다 .jsp 같은 확장자를 가진 URL 말고는 모두 DispatcherServlet 이 관할한다.
> 매핑조건이 까다로운게 우선이기 때문에, DispatcherServlet 의 / 보다 톰캣의 *.jsp 가 더 먼저 처리 된다.
	
2. @Controller가 담당하는 URL 은 컨트롤러가 알아서 처리 한다.
> 여기까진 당연한 얘기를 한거다. .js, .css 같이 컨트롤러 매핑이 되지 않는것들을 어떻게 처리할수 있는지가 남았다.

3. DefaultServletHttpRequestHandler 라는게 개입되어, 컨트롤러에 매핑되지 않는 요청은 디폴트 서블릿 에게로 넘긴다. (직접 처리하는것은 아니다)
DefaultServletHttpRequestHandler 는 url mapping이 /** 로 매핑되어 있지만 우선순위가 가장 낮아서, 컨트롤러의 모든 핸들러 매핑을 먼저 거친 후 들어오게 된다. 
> 그런데 넘긴다는게 쉬운얘긴 아닌다. 서블릿 2.0 부터는 하나의 서블릿이 다른 서블릿을 직접 찾아서 실행하는게 막혔기 떄문이다.
> 실제로 ServletContext.getServlets() 나 getServletNames() 같은 메소드는 null을 리턴하도록 되어있다.
> 대신에 getNamedDispatcher(String name) 처럼 이름으로 RequestDispatcher 를 가져와서 default servlet 에게로 포워드 하는 방식이다.


















