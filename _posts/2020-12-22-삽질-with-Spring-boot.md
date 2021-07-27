---
title: 그간의 삽질 with Spring boot
categories:
  - Develop
tags:
  - springboot
toc: true
toc_label: "Contents"
toc_sticky: true
---

그간 프로젝트를 진행하면서 겪었던 소소한 문제(?)들을 정리해 보고자 한다.

### 의존 자동 주입 시 동일한 타입을 가진 빈 객체가 여러 개 존재하는 경우

의존성 주입을 하기 위해서 자동 주입을 사용할 수 있다. 자동 주입을 사용하면 스프링이 알아서 의존 객체를 찾아 주입해 준다. 보통 자동 주입을 위해서 `@Autowired` 어노테이션을 사용한다. 사용 범위는 아래와 같다.

* 생성자에 사용하기
* 필드에 사용하기
* 설정 메소드(`setXXX`)에 사용하기

`@Autowired` 어노테이션을 사용하면 스프링은 타입을 이용해서 의존 대상 객체를 검색한다. 해당 타입에 할당할 수 있는 빈 객체를 찾아서 주입 대상으로 선택하게 되는 것이다. 이때, 동일한 타입을 가진 빈 객체가 두 개 이상 존재하는 경우 어떤 빈을 주입해야 할지 알 수 없어서 예외를 발생시킨다. (예외적으로, `@Autowired`가 적용된 필드나 설정 메소드의 프로퍼티 이름과 같은 이름을 가진 빈 객체가 존재하는 경우에는 이름이 같은 빈 객체를 주입받는다.) 이런 문제를 해결하기 위해 `@Qualifier` 어노테이션을 사용할 수 있다. 스프링에서는 설정에서 빈의 한정자 값을 설정하고, `@Autowired` 어노테이션이 적용된 주입 대상에 `@Qualifier` 어노테이을 설정하고 그 값으로 앞서 설정한 한정자를 사용한다. 스프링 부트에서는 별도의 설정 없이 `@Qualifier` 어노테이션을 사용하는 것으로 충분하다. 빈 객체를 찾을 때 타입을 먼저 본 후 Qualifier로 지정된 id를 확인해서 의존성을 주입하게 된다.

### Spring Boot - JPA - MySQL 연동

의존성 설정하기(`build.gradle`)

```
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
compile 'mysql:mysql-connector-java'
```

`application.properties`

```
server.address=localhost
server.port=8080

# API 호출시, SQL 문을 콘솔에 출력한다.
spring.jpa.show-sql=true

# DDL 정의시 데이터베이스의 고유 기능을 사용합니다.
# ex) 테이블 생성, 삭제 등
spring.jpa.generate-ddl=true

# MySQL 을 사용할 것.
spring.jpa.database=mysql

# MySQL 설정
spring.datasource.url=jdbc:mysql://localhost:3306/DBNAME?useSSL=false&characterEncoding=UTF-8&serverTimezone=UTC
spring.datasource.username=db아이디
spring.datasource.password=db비번
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# MySQL 상세 지정
spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect
```



* MySQL 8.x 이후로 public key retrieval is not allowed 문제가 발생하는 경우 jdbc url에 `allowPublicKeyRetrieval=true&useSSL=false` 추가