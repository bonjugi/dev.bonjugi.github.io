---
layout: post
title:  "@JsonProperty 사용기"
date:   2018-11-28
excerpt: "Spring에서 사용되는 Jackson 라이브러리의 @JsonProperty 사용기"
tag:
- json
- serialize
- desirialize
comments: true
---


### fasterxml의 Jacson 라이브러리를 사용하자.
마구잡이로 등록된 pom.xml 을 보던중 Jackson 라이브러리가 2종류가 있었습니다.
1. org.codehaus.jackson.annotate.JsonProperty 
2. com.fasterxml.jackson.annotation

마침 1번 코드하우스의 1.8, 1.9 버전은 [스프링 4.1부터 지원하지 않는다고 합니다.](https://github.com/hyunjun19/axu4j/issues/32)
1번 out!

### 직렬화와 역직렬화
Json<->Object 변환 과정의 표현은 역직렬화, 직렬화라 합니다.
1. Json 스트링으로 response 하기 위해 객체를 직렬화 한다. - Serialize 
2. 반대로, request 하기 위해 Json 스트링을 객체로 역직렬화 한다. - Deserialize


### 직렬화 과정만 처리하고 역직렬화는 무시하는 방법
Response시에 DateTime 타입의 날짜가 있을때, 포맷팅된 String 형식의 데이터를 추가로 넘겨주고 싶을땐 필드를 추가하고싶지 않을때가 있습니다.
이런 경우처럼 @JsonProperty를 이용하여 어떻게 정보를 가공할수 있는지 알아 보겠습니다.

#### Jackson 2.6 미만
JsonIgnore를 필드와 setter에 추가하여 처리한 모습 입니다.
getter, setter를 모두 추가하고, setter와 필드에 `@JsonIgnore` 를 추가하는 트릭입니다.
Json 으로 Serialize 되었을때만 formatDate가 노출되고, Request 시에는 없어도 무방한 코드가 되었습니다.
하지만 불필요한 필드와 setter가 코드상에 남아있다는 단점이 있습니다. 
```java
public class SerializeDemo{

      @JsonIgnore
      private DateTime formatDateTime;


      @JsonProperty
      public String getFormatDateTime() {
        return formatDateTime;
      }

      @JsonIgnore
      public void setFormatDateTime(DateTime formatDateTime) {
        this.formatDateTime= formatDateTime;
      }
    }
```

#### Jackson 2.6 이상
2.6에서는 어노테이션에 access 인자가 추가 되었습니다.
getter 메소드만 생성해도 동작 합니다.
* READ_ONLY - 역직렬화에만 사용 (from Json) 
* WRITE_ONLY - 직렬화에서만 사용 (to Json)
* READ_WRITE - 둘다
* AUTO (?)
  
```java
public class SerializeDemo {
	
	@JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    public String getFormatDateTime() {
        return formatDateTime;
    }
}
```

@JsonAnyProperty , @JsonOrderProperty 등 다른 어노테이션은 아래 링크 확인 하면 좋습니다.
> [Jackson Annotation Example](https://www.baeldung.com/jackson-annotations)
