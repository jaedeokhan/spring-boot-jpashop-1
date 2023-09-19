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

#### JPA 쿼리 파라미터 남기기
##### 1. logging yml 사용

```
logging:
  level:
    org.hibernate.SQL: debug
    org.hibernate.type: trace# 파라미터 로깅
```

##### 2. 외부 라이브러리 사용 - https://github.com/gavlyukovskiy
- P6Spy

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

### 엔티티 클래스 개발2

- 다대다 매핑(Cateogry - Category_Item - Item)

#### Category


```
public class Category {
'''
    @ManyToMany
    @JoinTable(name = "category_item",
            joinColumns = @JoinColumn(name = "category_id"),
            inverseJoinColumns = @JoinColumn(name = "item_id")
    )
    private List<Item> items = new ArrayList<>();
'''
}
```

#### Item

```
public abstract class Item {
'''
    @ManyToMany(mappedBy = "items")
    private List<Category> categories = new ArrayList<>();
'''
}
```

#### 실무에서 주의사항
1. Setter 지양
2. `@Id` id를 `@Column(name = "member_id")` 로 명확하게 지정
3. 실무에서는 `@ManyToMany` 지양
    1. 중간 테이블에는  값을 더 넣을수가 없다.
4. Address는 불변한 값이기에 생성자로 생성하게 해주는게 좋다. protected로 기본 생성자를 만들어서 사람들이 막 new를 하지 못하게 안전하게 만들자.
    1. JPA가 이런 제약을 두는 이유는 JPA 구현 라이브러리가 객체를 생성할 때 리플렉션 기능을 허용해야해서 기본 스펙으로 있다.

### 엔티티 설계시 주의점

1. 엔티티에는 가급적 Setter 사용하지 말자
    - 변경 포인트가 많아서 유지보수가 어렵다.
2. 모든 연관관계는 지연로딩으로 설정
    - 즉시 로딩(EAGER)은 예측이 어렵고, 어떤 SQL이 실행될지 추적하기 어렵다. 특히 JPQL N+1 문제가 발생한다.
    - @XToOne(OneToOne, ManyToOne) 관계는 기본이 즉시로딩이므로 직접 지연로딩으로 설정해야 한다.
3. 컬렉션은 생성의 Best Pratice는 필드에서 바로 초기화하는 것
    - null 문제에서 안전
    - 하이버네이트는 엔티티 영속화 할 때, 컬렉션은 감싸서 하이버네이트가 제공하는 내장 컬렉션으로 변경한다.
4. 테이블, 컬럼명 생성 전략
    - Spring Boot에서는 SpringPhysicalNamingStrategy 사용
      - 카멜 -> 언더 스코어

#### CasCadeType.ALL을 통한 영속 전파로 일괄 등록
주문(Order)을 하나 등록하면 주문아이템(OrderItem) 3개가 있을 때 CasCadeType.ALL로 설정을 하면
Order만 등록을 하면 OrderItem 3개를 영속 전파로 자동 등록을 해준다.

```
package jpabook.jpashop.domain;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

import static javax.persistence.FetchType.*;

@Entity
@Table(name = "orders")
@Getter @Setter
public class Order {
...
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL) // 영속을 전파해서 ALL로 하면 order만 저장해도 orderItem 저장
    private List<OrderItem> orderItems = new ArrayList<>();

    @OneToOne(fetch = LAZY, cascade = CascadeType.ALL) // Order만 저장해도 영속 전파해서 Devliery 자동 저장
    @JoinColumn(name = "delivery_id")
    private Delivery delivery;
...
}

```

#### 연관관계 편의 메서드
객체 동기화를 해주기 위한 편의 메서드

현재 회원과 주문은 양방향 관계를 가지고 있다.
양방향 관계를 가지고 있으면 member에서 주문 리스트를 추가하고, order에서 회원을 추가해야 하는 2가지 동작이 있다.
편하게 사용하기 위해서 외래키를 가지고 있는 테이블에서 동기화 하는 편의 메서드를 생성한다.

```
package jpabook.jpashop.domain;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

import static javax.persistence.FetchType.*;

@Entity
@Table(name = "orders")
@Getter @Setter
public class Order {
...
    //== 연관관계 편의 메서드==//
    public void setMember(Member member) {
        this.member = member;
        member.getOrders().add(this);
    }

    public void addOrderItem(OrderItem orderItem) {
        orderItems.add(orderItem);
        orderItem.setOrder(this);
    }

    public void setDelivery(Delivery delivery) {
        this.delivery = delivery;
        delivery.setOrder(this);
    }

}

```
