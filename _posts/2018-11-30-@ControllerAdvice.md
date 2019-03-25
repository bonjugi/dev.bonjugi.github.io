---
layout: post
title:  "@ControllerAdvice 사용기"
date:   2018-11-30
excerpt: "@Controller 사용기"
tag:
- intelij
- hot deploy
comments: true
---



### 전역 Exception 처리가 가능

@Component를 상속하고 있어 Bean으로 등록이 가능한 @ControllerAdvice는 @ExceptionHandler 와 함께 사용하여 


```java

@Slf4j
@ControllerAdvice
@Controller
public class DefaultExceptionHandler {

	@ResponseBody
	@ExceptionHandler(MethodArgumentNotValidException.class)
	public Response handleMethodArgumentNotValidException(MethodArgumentNotValidException e){
		log.error("MethodArgumentNotValidException!  = " + e);
		String eMessage = e.getBindingResult().getAllErrors().get(0).getDefaultMessage();
		return new Response(new MomenTApiException(HttpStatus.BAD_REQUEST.value(), eMessage));
	}

	@ResponseBody
	@ExceptionHandler(BindException.class)
	public Response handleBindException(BindException e){
		log.error("handleBindException! = " + e);
		String eMessage = e.getBindingResult().getAllErrors().get(0).getDefaultMessage();
		return new Response(new MomenTApiException(HttpStatus.BAD_REQUEST.value(), eMessage));
	}
}
```