---
title: "Jackson custom se/deserializer 만들기"
categories:
  - Develop
tags:
  - springboot
  - java
toc: true
toc_label: "Contents"
toc_sticky: true
---

### Jackson custom se/deserializer 만들기

* `Jackson`을 사용해서 직렬화를 하는데, 기본 설정 외에 커스터마이징을 해야 하는 일이 생겼다.
* 필요한 필드에 어노테이션을 붙이는 것보다 아예 `customSerializer`를 만드는 게 의외로 더 깔끔하고 쉬워서 기록겸 정리해 둬 본다.
* `objectMapper`는 여러 개의 `module`을 가지고 있고, `module`은 여러 개의 `customSerializer` (실제 역/직렬화에 쓰임)를 가지고 있다. 따라서 `Jackson`을 사용한 역/직렬화 시에 원하는 대로 커스터마이징을 추가하고 싶다면 `customSerializer`를 만들어 `module`에 추가하고, 그 `module`을 `objectMapper`에 추가하면 된다.



#### 그러면 custom serializer는 어떻게 만들지?

* `JsonSerializer` 클래스를 상속하고 `serialize`를 오버라이드하면 된다.

````java
public class MemberSerializer extends JsonSerializer<Member> {
    @Override
    public void serialize(Member member, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
      jsonGenerator.writeStartObject(); // `{` 을 연다.
      jsonGenerator.writeFieldName("name"); // 필드 이름을 쓴다.
      jsonGenerator.writeString(member.getName()); // 값을 쓴다.
      // ...
      jsonGenerator.writeEndObject(); // `}` 닫는다.
    }
}
````

* 다 작성했다면 `module`에 해당 `serializer`를 추가하고, 그 `module`을 `objectMapper`에 추가한다.

```java
void customSerializerTest() throws JsonProcessingException {
    SimpleModule module = new SimpleModule();
    module.addSerializer(Member.class, new MemberSerializer());
    ObjectMapper mapper = new ObjectMapper();
    mapper.registerModule(module);
    mapper.writeValueAsString(new Member(1L, "kim", 24));
}
```

