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

### 엔티티 클래스 개발1

#### 연관관계 주인 설정 - mappedBy = "테이블명" , JoinColumn(name = "외래키")
- 연관관계의 주인이 아닌 곳에 mappedBy 사용 
- Ex) Member와 Order의 경우에 Order가 외래키를 가지고 있어서 연관관계의 주인이다.
- Member에서는 주인이 아니라 거울이라고 mappedBy로 order 테이블 정의

```
@OneToMany(mappedBy = "order")
private List<Order> orders = new ArrayList<>();
```

- Order에서는 연관관계 설정해주고, @JoinColumn으로 member_id 연결

```
@ManyToOne
@JoinColumn(name = "member_id")
private Member member;
```

#### Jav 8 이상 LocalDateTime 시 날짜 애노테이션 불필요
- LocalDateTime을 쓰면 날짜 매핑 애노테이션을 쓰지 않아도 되고, Hibernate가 처리해준다.

#### Item 부모 추상 클래스 상속 전략 및 자식 클래스 설정
- Item의 자식으로 Album, Book, Movie 존재

- Item 부모 클래스에서 설정 값 @Inheritance 싱글 전략과 @DiscriminatorColumn으로 어떤 이름으로 쓸지 설정

```
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "dtype")
@Getter @Setter
public abstract class Item {
...
}
```

- Alumn, Book, Movie 자식 클래스 설정 값 @DiscriminatorValue("B") 어떤 값 사용할지 선택

```
@Entity
@DiscriminatorValue("B")
@Getter @Setter
public class Book extends Item {
...
}
```

#### Address @Embedded 설정

- 공통 값으로 설정하는 클래스에서 @Embeddable 설정, 사용하는 곳에서 @Embedded 서렂ㅇ
- 두 애노테이션 중 하나만 사용해도 무방, 직관적으로 두 개 사용


- Address 클래스에서 @Embeddable 설정

```
@Embeddable
@Getter
public class Address {
...
}
```

- 사용하는 클래스에서 @Embedded 설정

```
@Embedded
private Address address;
```

#### Enum 값 사용시 주의사항
- @Enumerated 태그를 사용하는데 기본값이 EnumType.ORDINAL이라서 숫자로 들어가서 변경이 있으면 예상치 못한값이 발생 가능성
- EnumType.STRING 권장

```
@Enumerated(EnumType.STRING)
private DeliveryStatus status; // READY, COMP
```
