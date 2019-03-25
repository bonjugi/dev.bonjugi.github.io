---
layout: post
title:  "dispatcherServlet 에 대해서"
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

#### web.xml 은 2종류 
web.xml 종류는 어플리케이션의 web.xml과 서블릿컨테이너(톰캣)의 web.xml 두가지가 있습니다.
어플리케이션 web.xml은 프로젝트 생성시, 톰캣의 web.xml은 톰캣홈/conf/web.xml 에 있습니다.
문제는 두개가 상호간에 간섭이 없다면 헷갈리지 않는데 두개 다 url을 갖고 좌지우지 하려하니 혼란스러움이 있었습니다. 


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

#### 어플리케이션의 web.xml `/*` 매핑 패턴은 톰캣의 `*.jsp` 매핑 패턴을 덮게 된다.
*.jsp 패턴의 url은 톰캣에 내장된 JspServlet의 담당 입니다. 
그래서 mvc1 구조와 같이 http://~~/*.jsp 와 같이 직접 파일을 호출 하더라도 렌더하는데 문제가 없습니다.
하지만 어플리케이션에서 /* 을 매핑하면 톰캣의 *.jsp 패턴을 덮게 됩니다.
결국 DispatcherServlet 이 매핑되는데, 어플리케이션 내의 어떤 컨트롤러에도 *.jsp 를 매핑하는곳이 없으므로 404가 뜨게 됩니다.
```text
20-Dec-2018 17:50:46.321 경고 [http-nio-8080-exec-10] org.springframework.web.servlet.PageNotFound.noHandlerFound No mapping found for HTTP request with URI [/WEB-INF/views/index.jsp] in DispatcherServlet with name 'dispatcher'
```

#### `*.jsp` 파일패스 호출이 아닌, `/hello` 와 같은 컨트롤러 매핑패턴으로 호출하는건 영향이 없는것인가?
그렇지 않습니다. 아래와 같이 /hello 를 매핑한 컨트롤러가 있다고 가정하겠습니다.
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
디폴트 뷰 리졸버인 InternalViewResolver 를 일반적으로 사용하는데, 
이 리졸버는 컨트롤러에서 만든 Model 과 함께 InternalResourceView를 DispatcherServlet에 넘겨줍니다.
DispatcherServlet는 뷰와 모델을 받고, /WEB-INF/views/index.jsp 로 foward 를 하는 구조 입니다.
즉 컨트롤러로 매핑된 URL 도 결국은 *.jsp를 사용할수밖에 없습니다.실제로 톰캣의 *.jsp를 주석시, http://~~/*.jsp 패턴으로 접근이 불가한것은 당연하고, http://~/hello 로 접근하더라도 `컨트롤러 접근!` 로그가 찍힌뒤, 매핑을 찾을수 없다는 에러가 발생하면서 브라우저는 404를 노출하게 됩니다.
```text
// 다시만난 에러메세지.. ㅠㅠ
20-Dec-2018 17:50:46.321 경고 [http-nio-8080-exec-10] org.springframework.web.servlet.PageNotFound.noHandlerFound No mapping found for HTTP request with URI [/WEB-INF/views/index.jsp] in DispatcherServlet with name 'dispatcher'
```
> 하지만 왜 이름이 dispatcher인지 모르겠네요.. 담당인 톰캣의 JspServlet의 이름인 jsp가 찍혀야 할것 같은데..  


#### 그렇다면 `/` 는 어떻게 동작 하길레 가능하단 것인가?
/ 패턴은, 톰캣의 web.xml 에도 있고 default 서블릿에도 있습니다.
default 서블릿 에 대해서 잠깐 얘기하자면, 어디에도 매핑되지 않은 패턴을 처리하는 종착점이라고 보면 됩니다.
어플리케이션의 매핑 패턴이 우선하게 되어 모든 URL 패턴을 DispatcherServlet으로 매핑되게 됩니다.
간단하게 해결되었지만, 문제는 .css, .js 등 정적 리소스까지 담당하게 되는것이 문제인데, 이것에 대한 해결책은 
`<mvc:default-servlet-handler/>` 라는게 있습니다.
이것은 DispathcerServlet이 처리하지 못하는것은 다시 디폴트 서블릿에게 넘겨버리는. 달면 삼키고 쓰면 뱉는 설정 입니다.  

