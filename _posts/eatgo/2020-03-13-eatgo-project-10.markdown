---
layout: post
title: "프로젝트 모듈화/진짜 영속화"
excerpt: 스프링부트를 이용한 eatgo 프로젝트-day9 (프로젝트 모듈화/진짜 영속화)
---

## 프로젝트 분리

기존 하나의 모듈이였던 프로젝트를 아래와 같이 분리해주었다.

- common
- admin-api
- customer-api 

common에는 domain layer(domain, repository)  
admin-api에는 admin의 application(service)/interface(controller) layer  
customer-api에는 customer의 application(service)/interface(controller) layer


그리고 admin과 customer의 build.gradle에서 common의 의존성을 추가해준다.

```groovy
implementation project(':eatgo-common')
```

이제 빌드를 해보자.  
`./gradlew build`
에러가 난다.

![build_error](../../images/20200313/build_error.PNG){:width="500"}

common의 build.gradle에 다음을 추가하여 해결하였다.

```groovy
jar {
    enabled = true
}
bootJar {
    enabled = false
}
```

이제 프로젝트 분리가 끝났다.

![project_directory](../../images/20200313/project_directory.PNG){:width="200"}


## 진짜 영속화

지금까지는 JPA를 사용하고 있기는 했지만 따로 설정을 안해주어 H2 DB를 사용할때 in-memory 방식이였다.

이제 application.properties파일을 변경하여 file 형식으로 바꿔보자

우선 기존 application.properties을 yml형식으로 바꿔주었다.

```yaml
spring:
  datasource:
    url: jdbc:h2:~/data/eatgo
  jpa:
    hibernate:
      ddl-auto: update
```

spring.datasource.url을 db를 h2로, 파일의 경로를 정해주었다.
spring.jpa.hibernate.ddl-auto= update로 해주었다.

이제 hibernate의 sessionFactory가 올라갈 때 변경된 스키마가 있으면 업데이트를 해준다.  
설정값은 create, create-drop, update, validate 가 있다.