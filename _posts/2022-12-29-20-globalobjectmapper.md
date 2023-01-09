---
layout: post
title:  "ObjectMapper 전역 설정"
date:   2022-12-29 00:01:03
categories: BACKEND
tags: spring objectmapper
cover: spring.png
---

Spring으로 API 개발하면서 자주 사용하게 되는 **ObjectMapper**를 기본값이 아닌 설정으로 사용할 때 설정하는 방법이다. 간혹 매번 `new ObjectMapper()` 해서 사용하는 사람들이 있는데..(엄청 많음) 
<span class="text-danger">**Thread safe**</span> 하므로
전역으로 한개 만 설정하여 사용해도 된다. (**RestTemplate**도 동일하다)

**LocalDateTime Serializer**의 경우 일관성이 없거나 필드에 일일히 `@JsonFormat` 또는 `@DateTimeFormat` 설정 해줘야 하는데, 번거로우니
글로벌로 동작하는 **ObjectMapper Bean** 자체를 재설정하여 알아서 포맷 되도록 한다.

> Spring Boot 버전 낮을 경우엔 JavaTimeModule 관련(jsr310) 별도 추가 해야 쓸 수 있다.

```java

@Configuration
public class WebConfig {

    .....

    // 전역 Json Serialize/DeSerialize 설정 (세부 값은 환경 및 서비스에 따라 조절) 
    @Bean 
    public ObjectMapper ObjectMapper() { 
        ObjectMapper objectMapper = new ObjectMapper(); 
        objectMapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS); 
        objectMapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES); 
        objectMapper.configure(JsonGenerator.Feature.WRITE_BIGDECIMAL_AS_PLAIN, true);

        JavaTimeModule javaTimeModule = new JavaTimeModule(); 
        javaTimeModule.addSerializer(LocalDateTime.class, new LocalDateTimeSerializer(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"))); 
        javaTimeModule.addSerializer(LocalDate.class, new LocalDateSerializer(DateTimeFormatter.ofPattern("yyyy-MM-dd"))); 
        javaTimeModule.addSerializer(LocalTime.class, new LocalTimeSerializer(DateTimeFormatter.ofPattern("HH:mm:ss"))); 
        javaTimeModule.addDeserializer(LocalDateTime.class, new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"))); 
        javaTimeModule.addDeserializer(LocalDate.class, new LocalDateDeserializer(DateTimeFormatter.ofPattern("yyyy-MM-dd"))); 
        javaTimeModule.addDeserializer(LocalTime.class, new LocalTimeDeserializer(DateTimeFormatter.ofPattern("HH:mm:ss")));

        objectMapper.registerModule(javaTimeModule); 
        return objectMapper; 
    }

    ....
}
```