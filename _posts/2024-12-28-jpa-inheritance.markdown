---
layout: post
comments: true
title: "[WEB][JAVA] JPA 상속 및 심화"
subtitle: JPA 추상화 패턴
description: 
date: 2024-12-28
categories: WEB JAVA
background: '/img/port.jpg'
---

## [WEB][JAVA] JPA 상속 및 심화

### 1. 서론

JPA는 편리한 기능을 제공하는 프레임워크이다. 그렇다고 JPA가 **"단순 쿼리를 치지 않아서 좋다."** 라는 의미만 있는 것은 아니라고 생각한다. 

>> JPA는 객체지향적으로 데이터를 다루는 프레임워크이다.

여기에 초점이 맞춰져야 한다고 본다. 

네가 뭔데 심화된 내용을 적냐고 말하는 사람이 있을 수 있긴 하지만, 도서관에서 스프링 책을 찾아보거나 JPA 관련 포스트를 찾아보며 공부하면서 JPA를 단순 ORM 매핑 프레임워크로 설명하고 넘어갔기 때문에, 최근 DB 스키마를 작성하며 JPA에서 객체지향을 다루며 상속이나 다형성이 어떻게 구현되는지 이용하며 더 심화된 내용을 적는 것이다.

### 2. 상속

일단 다형성이란 무엇인가?

자바에서 다형성이란 객체지향적으로 프로그래밍을 할 때, 상위 클래스의 타입으로 하위 클래스의 인스턴스를 참조할 수 있는 것을 의미한다.

내가 생각하는 가장 흔한 예시로는 컬렉션 프레임워크인데,

```java

    class Test {
        public static void main(String[] args) {
            List<String> testList;
            testList = new ArrayList<>();
        }
    }

```

이런 식으로, 리스트 인터페이스를 구현한 클래스들을 사용할 때, 리스트 인터페이스의 메서드를 사용할 수 있는 것이다.
JPA에서도 마찬가지로 상속을 이용한 다형성을 이용할 수 있는데,

예를 들어서 이런 User 엔티티가 있다고 가정해보자.

```java

@Getter @Setter
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column
    private String email;

    // 생략
    // User는 스케쥴을 가지고 있다고 할 때를 생각해 보자.
    // 이런 식으로 스케쥴을 가지고 있는 것이다.
    @JsonManagedReference // 순환참조 방지(User에서 참조하는 WeddingSchedule을 참조하는 것을 방지)
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    private List<PersonalSchedule> clearedSchedule = new ArrayList<>();

    @Column
    private Sex sex;
    
    @Column
    private UserType userType;
}

```

뭐 이런 식으로, User가 어떤 일정을 가지고 있을 때, DB 스키마에 따라 정규화를 하거나 혹은 서비스의 로직에 따라 일정, 즉 스케쥴이 세분화된 타입이 있을 수가 있다.

예를 들어 회사 스케쥴, 즉 CompanySchedule 이 있을 수가 있고, 또 PersonalSchedule 이 있을 수도 있고, 더 세분화된 일정이라면 WeddingSchedule 이 있을 수도 있다. 

이런 상황에서, User가 이미 완료한 일정을 가지고 있을 때, 이런 식으로 다형성을 이용할 수 있다.

```java

// 생략
    private List<Schedule> clearedSchedule = new ArrayList<>();

```

그리고 Schedule 클래스를 상속받은 클래스들을 생성하고, 이런 식으로 다형성을 이용할 수 있다.


```java

@Getter @Setter
@Entity
@Table(name = "schedule")
public class Schedule {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private LocalDate startDate;
    private LocalDate endDate;

    @ManyToOne
    @JoinColumn(name = "place_id")
    private Place place; // 실제 타입은 구글 맵 API 또는 네이버 맵 API를 사용해야 함. 아직 잘 몰름..
    private String memo;

    // 생략

    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;

}


``` 
그리고 각각의 스케쥴 타입을 상속받은 클래스들을 생성하면


```java

@Getter @Setter
@Entity
@Table(name = "company_schedule")
@Inheritance(strategy = InheritanceType.JOINED)
public class CompanySchedule extends Schedule {

    @Column
    private String companyName;

    @Column
    private String companyAddress;

    @Column
    private String companyPhone;

    // 생략
}

@Getter @Setter
@Entity
@Table(name = "personal_schedule")
@Inheritance(strategy = InheritanceType.JOINED)
public class PersonalSchedule extends Schedule {

    @Column
    private String phone;

    // 생략
}

....

```

이렇게 되면, User 엔티티는 이런 식으로 다형성을 이용할 수 있게 된다.
이런 경우에 JPA는 다음과 같이 DB에 저장하는데

<img src="/img/posts/java/jpa/2.png" width="80%">

Schedule 테이블의 id를 외래키로 가지고 있어서, 스케쥴의 공통적인 속성은 스케쥴에 저장되고, 각각의 스케쥴 타입의 고유한 속성은 각각의 스케쥴 타입에 저장된다.

그렇다고 해서 Schedule을 생성하고, CompanySchedule을 생성해서 저장하는 것이 아니라, 


```java
Schedule scadule = new Schedule(); 
scadule.setName("스케쥴 이름");
// 이런식으로 스케쥴의 공통적인 속성을 설정하고 회사 스케쥴에 지정하는 것
CompanySchedule companySchedule = new CompanySchedule()
companySchedule.setSchedule(schedule);

```

이런 식으로 생성하는 것이 아니라,

```java

CompanySchedule companySchedule = new CompanySchedule();

// Schedule 테이블에 저장되는 것
companySchedule.setName("회사 스케쥴 이름");
companySchedule.setStartDate(LocalDate.now());
companySchedule.setEndDate(LocalDate.now().plusDays(1));

// CompanySchedule 테이블에 저장되는 것
companySchedule.setCompanyName("회사 이름");
companySchedule.setCompanyAddress("회사 주소");
companySchedule.setCompanyPhone("회사 전화번호");

```

이런 식으로 생성이 된다는 것이다. CompanyShedule은 Schedule을 상속받았기 때문에, Schedule의 속성을 가지고 있고, 또 CompanySchedule의 고유한 속성도 가지고 있는 것이다.


#### 2.1 상속 전략

위에서 보여드린 예시는 JOINED 전략을 사용한 것이다. 
대부분 JPA에서 상속을 사용할 때 JOINED 전략을 사용하는데 그 이유는 **정규화**를 통해 데이터를 중복 저장하지 않기 위해서이다.

JOINED 전략은 부모 클래스와 자식 클래스를 모두 테이블로 생성하고, 자식 클래스의 고유한 속성을 저장하는 것이다.

JOINED 전략이 아닌 다른 전략을 사용하는 경우에는 정규화가 제대로 이루어지지 않는데, 예를 들어서 **TABLE_PER_CLASS** 전략과 **SINGLE_TABLE** 전략을 사용하는 경우를 한 번 보자.

이 경우는 SINGLE_TABLE 전략을 사용하는 경우이다.

<img src="/img/posts/java/jpa/3.png" width="80%">

이 경우는 TABLE_PER_CLASS 전략을 사용하는 경우이다.

<img src="/img/posts/java/jpa/4.png" width="80%">

SINGLE_TABLE 전략은 이렇듯 모든 자식 클래스의 속성을 하나의 테이블에 저장하는 것이고, TABLE_PER_CLASS 전략은 각각의 자식 클래스를 하나의 테이블로 생성하는 것이다.

간단히 예시를 들자면 **회사 이름**이라는 속성이 변경되는 경우를 생각해 볼 때(물론 그럴 일이 쉽사리 일어난다는 말은 아니다)

JOINED 전략을 사용하는 경우에는 회사 이름이 변경되면 CompanySchedule 테이블에 저장된 회사 테이블 하나만 변경될 테니 문제가 되지 않지만,
SINGLE_TABLE 전략을 사용하는 경우에는 회사 이름이 변경되면 모든 테이블에 저장된 회사 이름이 변경되는 것이고, TABLE_PER_CLASS 전략을 사용하는 경우에도 회사 이름이 변경되면 모든 테이블에 저장된 회사 이름이 변경될 것이고, 정규화가 제대로 이루어지지 않아 수천건의 쿼리가 나가게 될 것이다.

그렇기는 해도 SINGLE_TABLE 전략은 사용하는 경우가 있는데, JOIN이 발생하지 않기 때문에 성능이 좋다는 장점이 있고, 테이블이 가장 적어 관리가 편하다는 장점이 있다. 

그러나 이런 전략들은 정규화가 제대로 이루어지지 않기 때문에 데이터를 중복 저장하게 되는 문제가 있다는 점은 명심해야 한다.

##### JOINED 전략
- 부모 클래스와 자식 클래스를 각각 테이블로 생성.
- 자식 클래스의 고유한 속성은 자식 테이블에 저장.
- 데이터 중복을 피하고 정규화를 유지.

##### SINGLE_TABLE 전략
- 모든 자식 클래스의 속성을 하나의 테이블에 저장.
- JOIN이 발생하지 않아 성능이 좋고, 테이블 수가 적어 관리가 편리.
- 정규화 위반 및 데이터 중복 발생.

##### TABLE_PER_CLASS 전략
- 각 자식 클래스를 별도의 테이블로 생성.
- 정규화 위반 및 데이터 중복 발생.


### 3. 그러면 인터페이스 구현은 안 돼?

**안된다.** JPA에서 인터페이스를 구현하는 것은 불가능하다. **JPA에서 상속 메커니즘은 반드시 클래스를 상속받아야 한다.**

이유는 여러가지가 있는데, 

1. 인터페이스는 구현체가 없기 때문에 상속을 할 수 없다.
2. 인터페이스는 필드가 없기 때문에 인터페이스를 엔티티로 매핑할 수는 없다.
3. 인터페이스는 추상 메서드만 가지고 있기 때문에 구현체가 없다.

### 4. 추상 클래스는 돼?

**된다.** 

```java

@MappedSuperclass
public abstract class BaseEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private LocalDate createdDate;
    private LocalDate modifiedDate;

    // 공통 메서드
    public void updateTimestamps() {
        this.modifiedDate = LocalDate.now();
    }
}

@Entity
public class User extends BaseEntity {
    private String username;
    private String password;
    // 기타 속성 및 메서드
}

@Entity
public class Product extends BaseEntity {
    private String name;
    private BigDecimal price;
    // 기타 속성 및 메서드
}

```

이렇게 추상 클래스를 상속받으면, 하위 클래스에서 추상 클래스의 필드를 사용할 수 있다.

하지만 추상 클래스는 일반 클래스와의 상속과는 다르게, 상속 전략 설정이 불가능하고(JOINED, SINGLE_TABLE, TABLE_PER_CLASS) 그 대신 @MappedSuperclass 어노테이션을 사용해야 한다.

@MappedSuperclass는 테이블과 매핑되지 않는다는 것이고, 즉 실제 DBMS에 테이블이 생성되지 않는다.


### 5. 정리

JPA의 상속 전략을 잘 이해하고 활용하는 것은 객체지향 설계를 DB에 잘 반영하기 위해 매우 중요하다. 다음은 JPA 상속 전략의 핵심 내용이다:

1. **JOINED 전략**
   - 부모 클래스와 자식 클래스를 각각의 테이블로 만든다.
   - 데이터 정규화를 유지하고, 상속 구조를 명확하게 표현할 수 있다.
   - 일반적으로 가장 많이 사용되는 전략이다.

2. **SINGLE_TABLE 전략**
   - 모든 자식 클래스의 속성을 하나의 테이블에 저장한다.
   - JOIN이 없어서 성능이 좋고, 테이블 수가 적어 관리하기 편하다.
   - 하지만 데이터 중복의 위험이 있다.

3. **TABLE_PER_CLASS 전략**
   - 각 자식 클래스를 별도의 테이블로 만든다.
   - 데이터가 중복될 수 있고, 정규화가 제대로 안 될 수 있다.

4. **추상 클래스 사용**
   - JPA에서는 인터페이스를 상속받을 수는 없지만, 추상 클래스로 공통 필드와 메서드를 상속받아 쓸 수 있다.
   - `@MappedSuperclass` 어노테이션으로 추상 클래스의 필드를 하위 엔티티에서 쓸 수 있다.
   - 추상 클래스는 테이블과 매핑되지 않고, 실제 DBMS에 테이블이 생성되지 않는다.

JPA의 상속 기능을 잘 활용하면 더 유연하고 유지보수하기 쉬운 애플리케이션을 만들 수 있다. 각 전략의 장단점을 잘 따져보고 애플리케이션에 맞는 적절한 전략을 선택하는 게 중요하다.