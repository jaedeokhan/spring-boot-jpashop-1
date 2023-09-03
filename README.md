## spring-boot-jpashop-1

### H2 데이터베이스 설치
- h2 데이터베이스
  - 파일 접근     : jdbc:h2:~/jpashop
  - 네트워크 접근 : jdbc:h2:tcp://localhost/~/jpashop
- 2.1 특정 버전 이하부터는 H2 console에서 .mv.db 파일 생성되지 않는다.
- h2-console에서 파일 URL인 jdbc:h2:~/jpashop를 사용해서 접근해도
  생성되지 않아서 application.yml에서 datasource 등록해서 생성

```
spring
  h2:
    console:
      enabled: true
  datasource:
    # jdbc:h2:tcp://localhost/~/jpashop
    url: jdbc:h2:~/jpashop
    username: sa
    pas sword:
```

### JPA와 DB 설정, 동작 확인

JPA 쿼리 파라미터 남기기
1. logging yml 사용

```
logging:
  level:
    org.hibernate.SQL: debug
    org.hibernate.type: trace# 파라미터 로깅
```

2. 외부 라이브러리 사용 - https://github.com/gavlyukovskiy
P6Spy

build.gralde에 추가
```
implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.6'
```

