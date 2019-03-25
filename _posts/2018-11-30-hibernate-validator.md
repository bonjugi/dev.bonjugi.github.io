---
layout: post
title:  "hibernate-validator"
date:   2018-11-30
excerpt: ""
tag:
- intelij
- hot deploy
comments: true
---

validator 의존성 2가지가 있다.

둘다 JSR380 스펙에 맞춰 구현된 구현체인데, hibernate-validator 6.0.1 이상 버전에서는 validation-api 2.0 버전이 포함되어 있어서 hibernate-validator 하나만 추가해도 됩니다.

겹치는 어노테이션을 사용하더라도 한쪽엔 deprecated 표시까지 해주기도 하고, 중복 등록방지를 위해 6.0.1 이상을 사용합시다.



```xml
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>2.0.1.Final</version>
</dependency>

<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.0.7.Final</version>
</dependency>
```





### AssertTrue



### isVal




### hibernate 내장된 제약조건
http://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#section-builtin-constraints


