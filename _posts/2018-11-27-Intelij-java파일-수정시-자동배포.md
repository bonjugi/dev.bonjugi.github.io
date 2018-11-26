---
layout: post
title:  "Intelij java 파일 수정시 자동 배포"
date:   2018-11-26
excerpt: "Intelij는 기본적으로 파일 수정시 자동 배포가 이루어지지 않습니다. 설정을 바꿔서 자동배포가 이뤄지도록 합니다."
tag:
- intelij
- hot deploy
comments: true
---

이전 포스팅 [Intelij-resource파일-수정시-자동배포](/Intelij-resource파일-수정시-자동배포/ "Intelij-resource파일-수정시-자동배포/")
에서는 resource 파일 수정시 바로 적용하는 설정을 알아 봤습니다.
이번에는 java파일 수정시 리빌드 및 재배포까지 이뤄지도록 하는 설정을 알아 볼텐데요.
사실 dev-tools 의 설명이라고 봐야할것 같습니다.

### dev-tools?
dev-tools에는 Property Default, Automatic restart, Live Reload 같은 큼직한 기능들이 있습니다.
Property Default는 thymeleaf같은 템플릿 엔진 사용시에 캐싱을 disable 해주고
Live Reload는 view page 소스코드를 수정하는 즉시 브라우저가 리로드 되는 기능 입니다.
제가 주요하게 볼것은 자동 재배포. 즉 Automatic restart 기능에 대해서만 설명하겠습니다.
다른 주요 기능들을 자세히 보시려면 아래 출처를 방문하여 확인해 주세요.
[출처 haviyj님의 블로그](http://haviyj.tistory.com/11)


### dev-tools 의존성 추가.
dev-tools에는 라이브리로드 등 
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
</dependency>
```

### registry 수정
앱이 실행중에 컴파일러가 자동 생성하는것을 허용 하도록 설정 변경 합니다.
![](/assets/img/upload/intelij-registry.png "registry캡처")



### Build project automatically 체크
옵션명 처럼 빌드를 자동적으로 해준다는것 입니다.

![](/assets/img/upload/build-project-automatically.png)

이렇게 하면 앱이 running중에도 java 소스 변경시 알아서 빌드하고 target에 deploy해줍니다.
재compile 및 deploy를 하기 때문에 어쨋든 시간이 소요 되긴 하지만, restart보다는 훨씬 빠르다고 합니다.
> 요 말은 출처가 기억이 안나네요..





