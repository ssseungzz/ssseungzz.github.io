---
title: "ModelMapper와 MapStruct 알아보기"
categories:
  - Language
tags:
  - java
  - modelmapper
  - mapstruct
toc: true
toc_label: "Contents"
toc_sticky: true
---

## ModelMapper & MapStruct

### ModelMapper, MapStruct는 무엇이고, 왜 사용할까?

* 둘 다 한 객체에서 다른 객체로의 변환(`mapping`)을 편리하게 할 수 있도록 도와주는 라이브러리이다.

* 왜 사용할까❓

  * 스프링으로 개발을 하다 보면 `dto`와 `entity` 간의 변환이 필요한 일이 아주아주아주아주 많다.
  * 그떼마다 빌더를 사용하거나 하는 등의 방법으로 하나하나 바꿔 주는 건 너무 **귀찮다**.
  * 개발자가 직접 하다 보니 실수가 있을 수 있다.
  * 필드가 많아질수록 코드가 지저분해진다.
  * 필드에 변화가 생기면 그때마다 코드 수정이 필요하다.

  ➡️ 직접 변환 코드를 작성하면 생산성과 유지 보수성이 떨어질 수밖에 없다. 그러니 생고생하지 말고 `ModelMapper`나 `MapStruct` 같은 변환 라이브러리를 사용해 보도록 하자.

### ModelMapper 사용하기

* [공식 문서](http://modelmapper.org/)를 보면 알 수 있듯 기본적인 사용 방법은 간단하다. 
* 의존성 추가

```xml
<dependency>
			<groupId>org.modelmapper</groupId>
			<artifactId>modelmapper</artifactId>
			<version>2.4.2</version>
</dependency>
```

* 엔티티와 dto 작성
* `ModelMapper` 가져다가 사용하기
  * 테스트용으로 간단하게 작성했지만 실제로 사용할 때는 필요한 설정을 추가해서 스프링 빈으로 등록해서 사용하면 된다.
  * 새로운 객체로 생성할 수도 있고, 기존 객체에 매핑할 수도 있다. 기존 객체에 매핑하는 경우도 `modelMapper.map(member, testDto)`와 같이 작성하면 된다.

```java
public class ModelMapperTest {

    @Test
    void modelMapperWorkTest() {
        ModelMapper modelMapper = new ModelMapper();
        Member member1 = new Member(1L, "Kim", 20);
        TestDto testDto = modelMapper.map(member1, TestDto.class);
        assertThat(member1.getAge()).isEqualTo(testDto.getAge());
    }
}
```

#### 상세하게 설정하기

* 매핑 설정을 바꿔 더 쓰기 편하게 만들 수 있다.
* `TypeMap`을 통해 설정할 수 있다.

```java
TypeMap typeMap = modelMapper.createTypeMap(Test.class, TestDto.class);
```

* 어떤 설정들이 가능한지 살펴보자.

```java
typeMap.addMapping(Source::getFirstName, Destination::setName);
typeMap.addMapping(Source::getAge, Destination::setAgeString);
typeMap.addMapping(src -> src.getCustomer().getAge(), PersonDTO::setAge);
```

* `source`의 `getter`와 `dest`의 `setter`를 매칭시켜 프로퍼티 매핑이 가능하다. 필드의 이름이 서로 다른 경우에도 이를 사용하면 매핑이 가능해진다.

```java
typeMap.addMappings(mapper -> mapper.map(Source::getFirstName, Destination::setName));
typeMap.addMappings(mapper -> mapper.map(Source::getAge, Destination::setAgeString));
```

* 람다식도 사용 가능하며, 타입이 달라도 사용할 수 있다.

```java
typeMap.addMappings(mapper -> mapper.skip(Destination::setName));
```

* 스킵하고 싶은 필드는 스킵할 수도 있다.
* 코드 체이닝이 가능하므로 `TypeMap`을 생성하고 바로 매핑을 추가해 주면 조금 더 코드를 깔끔하게 작성할 수 있다.

##### Converter

* 매핑할 때 변화를 주고 싶다면(대문자로 바꾸기 등) 컨버터를 사용할 수 있다.
* 아래 코드는 문자열을 대문자 문자열로 변환한다.
  * `using`에 사용할 컨버터를 전달한다.
  * 컨버터를 생성하지 않고 `using`에 람다식을 전달할 수도 있다.

```java
Converter<String, String> toUppercase =
    ctx -> ctx.getSource() == null ? null : ctx.getSource().toUppercase();
typeMap.addMappings(mapper -> mapper.using(toUppercase).map(Person::getName, PersonDTO::setName));
typeMap.addMappings(mapper -> mapper.using(ctx -> ((String) ctx.getSource()).toUpperCase())
	.map(Person::getName, PersonDTO::setName));
```

##### Providers

* 프로바이더는 매핑에 우선해서 목적지 필드의 인스턴스를 제공할 수 있다. ~~왜 필요한 건지는 잘 모르겠다..~~
  * 컨버터와 마찬가지로 `with`에 람다식을 전달해서 사용할 수도 있다.

```java
Provider<Person> personProvider = req -> new Person();
typeMap.addMappings(mapper -> mapper.with(personProvider).map(Source::getPerson, Destination::setPerson));
typeMap.addMappings(mapper ->
	mapper.with(req -> new Person()).map(Source::getPerson, Destination::setPerson));
```

##### Conditional Mapping

* 매핑을 하되 어떤 조건을 두고 싶을 때 사용할 수 있다.
  * `null`인 경우 스킵하고 싶다거나...

```java
Condition notNull = ctx -> ctx.getSource() != null;
typeMap.addMappings(mapper -> mapper.when(notNull).map(Person::getName, PersonDTO::setName));
typeMap.addMappings(mapper -> mapper.when(notNull).skip(PersonDTO::setName));
```

> 여기까지가 기본적인 기능이고, 각 기능마다 다른 세부적인 기능을 더 제공하기도 하니 필요하면 공식 문서를 찾아보자.

##### +) Tokenizer

* 네이밍 컨벤션이 다른 경우 토크나이저 설정을 통해서 매핑시킬 수 있다.

```java
modelMapper.getConfiguration()
  .setSourceNameTokenizer(NameTokenizers.UNDERSCORE)
  .setDestinationNameTokenizer(NameTokenizers.UNDERSCORE)
```

* 이외에도 `setMethodAccessLevel`, `setFieldAccessLevel`등 다양한 설정이 가능하다.

#### 매핑 전략

* `ModelMapper`는 세 가지 매핑 전략을 제공한다.

1. `MatchingStrategies.STANDARD`

* source 속성을 destination 속성과 지능적으로 일치시킬 수 있으므로 든 destination 속성이 일치하고 모든 source 속성 이름에 토큰이 하나 이상 일치해야 한다.
* 토큰을 어떤 순서로도 일치시킬 수 있고, 모든 destination 속성들이 매치되어야 한다. 모든 source 속성은 최소 하나 이상의 이름이 매치되어야 한다.

2. `MatchingStrategies.STRICT`

* source와 destination 속성의 이름이 정확히 일치할 때만 매핑한다.

3. `MatchingStrategies.LOOSE`

* 계층 구조의 마지막 destination 속성만 일치하도록 해서 source 속성을 destination 속성에 일치시킬 수 있다.
* 토큰을 어떤 순서로도 일치시킬 수 있다. 마지막 destination 속성 이름은 모든 토큰이 일치해야 한다.
* 마지막 source 속성 이름에는 일치하는 토큰이 하나 이상 있어야 한다.

### MapStruct 사용하기

* `MapStruct`는 `lombok`과 유사하게 annotation Processor를 사용한다.
* 의존성 추가

```xml
	<properties>
		<java.version>11</java.version>
		<org.mapstruct.version>1.3.1.Final</org.mapstruct.version>
	</properties>
  
...
		<!-- lombok -->
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
		</dependency>

		<!-- mapstruct -->
		<dependency>
			<groupId>org.mapstruct</groupId>
			<artifactId>mapstruct</artifactId>
			<version>${org.mapstruct.version}</version>
		</dependency>
...    
    <build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.8.1</version>
				<configuration>
					<source>${java.version}</source>
					<target>${java.version}</target>
					<annotationProcessorPaths>
						<path>
							<groupId>org.projectlombok</groupId>
							<artifactId>lombok</artifactId>
							<version>${lombok.version}</version>
						</path>
						<path>
							<groupId>org.mapstruct</groupId>
							<artifactId>mapstruct-processor</artifactId>
							<version>${org.mapstruct.version}</version>
						</path>
					</annotationProcessorPaths>
				</configuration>
			</plugin>
		</plugins>
	</build>
```

* **`path` 순서 주의! 저 순서여야 동작한다. ** [참고](https://stackoverflow.com/questions/47676369/mapstruct-and-lombok-not-working-together)
* `@Mapper` 어노테이션을 사용해서 매핑에 사용할 매퍼 인터페이스를 정의해 준다. 구현체는 알아서 만들어 준다.
  * 이미 생성된 객체에 매핑하려는 경우 리턴 타입을 `void`로 하는 `write` 메소드를 작성해 준다.
  * `@BeforeMapping`과 `@AterMapping`으로 매핑하기 이전과 이후 특정 로직을 설정해 줄 수 있다.

```java
@Mapper
public interface MemberMapper {
    TestDto toTestDto(Member member);
    @AfterMapping
    void setAge(@MappingTarget TestDto testDto) {
      testDto.setAge(testDto.getAge() - 1);
    }
}
```

* 아래는 생성된 구현체이다.

```java
@Generated(
    value = "org.mapstruct.ap.MappingProcessor",
    date = "2021-08-08T18:22:01+0900",
    comments = "version: 1.3.1.Final, compiler: javac, environment: Java 13.0.1 (Oracle Corporation)"
)
public class MemberMapperImpl implements MemberMapper {

    @Override
    public TestDto toTestDto(Member member) {
        if ( member == null ) {
            return null;
        }

        TestDto testDto = new TestDto();

        testDto.setName( member.getName() );
        testDto.setAge( member.getAge() );

        return testDto;
    }
}
```

#### 상세하게 설정하기

* 필드의 이름이 다르면 `@Mapping` 어노테이션으로 매핑을 설정해 줄 수 있다.

```java
@Mapper
public interface MemberMapper {
    @Mapping(source = "name", target = "realName")
    TestDto toTestDto(Member member);
}
```

* 여러 객체를 합쳐서 매핑할 수도 있다.
  * 당연히 destination 객체에 해당 필드가 있어야 한다.

```java
@Mapper
public interface TestMapper {
  TestDto toTestDto(Member member, int bonus);
}
```

* 여러 객체를 합칠 때 destination 객체의 필드 이름과 source 객체의 이름이 다르면 `@Mapping` 어노테이션으로 지정해 줄 수 있다.

```java
@Mapper
public interface TestMapper {
  @Mapping(source = "bonus", target = "realBonus")
  TestDto toTestDto(Member member, int bonus);
}
```

* 매핑되지 않는 속성을 무시할 수도 있다.

```java
@Mapper
public interface TestMapper {
  @Mapping(target = "bonus", ignore = true)
  TestDto toTestDto(Member member);
}
```

* `default` 메소드를 통해 매핑 코드를 직접 구현할 수도 있다.

```java
@Mapper
public interface TestMapper {
    ObjectMapper OBJECT_MAPPER = new ObjectMapper();
  
    default String toString(Object obj) {
        try {
            return OBJECT_MAPPER.writeValueAsString(obj);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

#### 매핑 전략

* `ModelMapper`와 마찬가지로 매핑 전략을 설정할 수 있다.

1. `nullValueMappingStrategy`

* source 자체가 `null`일 때의 전략으로 `NullValueMappingStrategy.RETURN_DEFAULT`, `NullValueMappingStrategy.RETURN_NULL`이 옵션이다.

2. `nullValuePropertyMappingStrategy`

* source의 필드가 `null`일 때의 전략으로 `NullValuePropertyMappingStrategy.SET_TO_NULL`, `NullValuePropertyMappingStrategy.SET_TO_DEFAULT`, `NullValuePropertyMappingStrategy.IGNORE`이 옵션이다.

#### 매핑 정책

* 매핑 전략 외에도 매핑 정책이 존재한다.

1. `unmappedSourcePolicy`

* source의 필드가 타겟의 필드에 매핑되지 않을 때의 정책으로 `ReportingPolicy.IGNORE`, `ReportingPolicy.ERROR`, `ReportingPolicy.WARN`이 있다.

2. `typeConversionPolicy`

* 타입 변환 시 유실이 발생할 수 있을 때의 정책이다. 옵션은 위와 같다.

3. `unmappedTargetPolicy`

* target의 필드가 매핑되지 않을 때의 정책이다. 

### 둘 중에 뭘 사용해야 할까?

* 둘 다 쓰기 어렵지 않고 다양한 기능을 제공해 주는데, 무엇을 사용하는 게 나을까?
* 개인 취향에 따라 다르겠지만 성능 면에서는 `MapStruct`가 더 우수하고 많이 쓰인다고 한다. 그 외에 다른 장점도 있다.
  * annotation processor를 사용해 컴파일 시 매핑 코드를 생성하므로 컴파일 시 오류를 확인할 수 있다.
  * `ModelMapper`와 달리 리플렉션을 사용하지 않기 때문에 매핑 속도가 빠르다.
  * 디버깅이 쉽고 생성된 매핑 코드를 확인할 수 있다.
* 그렇지만 `ModelMapper`도 충분히 훌륭한 라이브러리이니 사용성이나 성능 등을 고려해 각자의 상황에 맞는 것으로 사용하면 될 것 같다.
* 이제 지겨운 `builder`나 `setter` 등을 탈출해 보자. 👍



#### References

* https://meetup.toast.com/posts/213
* https://mapstruct.org/
* http://modelmapper.org/user-manual/