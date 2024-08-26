---
layout: post
comments: true
title: "[WEB][Spring] 스프링 N + 1 문제"
subtitle: 단방향 연관 관계에서의 N + 1 문제 해결 방법(패치 조인)
description: 
date: 2024-08-26
categories: WEB JAVA
background: '/img/port.jpg'
---

## [WEB][Spring] 스프링 N + 1 문제

### 1. 서론

<img src="/img/posts/spring/1.png" width="80%">

프로젝트를 진행 중에, 스프링에서 N + 1 문제가 발생했다. 현재 사진에서 보이는 것과 같이, 쿼리 시간이 약 30초 씩이나 걸리는 문제가 있었는데 
과거에 했던 프로젝트에서 해당 문제가 N+1 문제 때문에 발생한다는 것을 이미 알고 있었기 때문에 트러블 슈팅에 어려움을 겪지는 않았지만,
이번에 문제를 해결하면서 다시 한 번 정리해 보고자 했다.

<img src="/img/posts/spring/2.png" width="80%">

이는 해당 N + 1 문제를 해결한 뒤에 나온 API 테스트 결과이다. 30초에 달하던 쿼리 시간이 1.7초로 줄어든 것을 볼 수 있다.


### 2. 배경

<img src="/img/posts/spring/3.png" width="80%">

문제가 되는 테이블은 다음과 같았다. (테이블 설계나 구조, 그리고 네이밍 컨벤션등이 이상하다고 지적한다면, 부끄럽지만 맞다. 
하지만, 이는 이미 구축된 시스템을 수정하는 과정에서 이미 데이터가 저장된 테이블 구조를 바꾸기 어려워서 그대로 사용하게 되었다.)
아무튼, 구조를 살펴보자면

1. 주문을 저장하는 orders 라는 테이블이 존재하고
2. 각각 주문의 상세 내역을 저장하는 orderitem 라는 테이블이 존재한다. 그리고 이 테이블은 orders 테이블과 1:N 관계를 가지고 있다.
orderItem 테이블은 orders 테이블의 id를 참조하는 외래키를 가지고 있다. 
하지만 orderitem 테이블은 orders 테이블의 id를 참조하는 외래키를 가지고 있지 않다.

이를 스프링 JPA로 구현하면 다음과 같다.

#### Order.java
~~~ java

import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;
import java.time.LocalDateTime;

@Entity
@Table(name = "orders")
@Getter @Setter
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "customerID")
    private Customer customer;

    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "kioskID", nullable = false)
    private Kiosk kiosk;

    @Column(name="date_time",nullable = false)
    private LocalDateTime dateTime;

    @Column(name="total_price",nullable = false)
    private long totalPrice;

    @Column(name="is_packaged",nullable = false)
    private boolean isPackaged;

    @Column(name="payment_uid",nullable = false)
    private String paymentUid;

    // 결제 환불을 위한 join
    @OneToOne(cascade = CascadeType.REMOVE)
    @JoinColumn(name = "order_module_dto")
    private OrderModuleDTO orderModuleDTO;
}
~~~

#### OrderItem.java

~~~ java

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;

import java.math.BigDecimal;

@Entity
@Getter @Setter
@Table(name = "orderitem")

public class OrderItem {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    @ManyToOne
    @JoinColumn(name = "orderID", nullable = false)
    private Order order;

    @ManyToOne
    @JoinColumn(name = "menuID",nullable = false)
    private Menu menu;

    @ManyToOne
    @JoinColumn(name = "custom_optionID")
    private CustomOption customOption;

    @Column(nullable = false)
    int quantity;

    @Column(nullable = false)
    Long price;

}

~~~

3. 여기서 ordermoduledto 라는 테이블이 존재하는데, 이 테이블은 orders 테이블과 1:1 관계를 가지고 있다.
ordermoduledto 테이블은 orders 테이블의 id를 참조하는 외래키를 가지고 있다.

### 3. 문제 발생

그리고 문제가 발생했는데, order_complete를 이용해서 완료되지 않은 orders를 가져오는 API를 만들었는데,
이 API를 호출하면서 orders 테이블을 찾고, 그 과정에서 orders와 연결된 ordermoduledto 테이블을 추가적으로 찾는 N + 1 이 발생했고,
또 다시 orderitem 테이블을 쿼리하면서 약 30초간 쿼리 시간이 걸리는 문제가 발생했다.

다행히도, orderitem 테이블을 쿼리할 때는 N + 1 문제가 발생하지 않았다. 그 이유는 orderitem 테이블이 orders 테이블과 연결되어 있지만 
캐시되어 있기 때문이다.


### 4. 문제 해결

이 문제를 해결하기 위해서 일단 나는 *패치 조인*을 이용해서 해결했다.

#### OrderCompleteRepository.java


~~~ java 
import ac.su.kiosk.domain.OrderComplete;
import jakarta.transaction.Transactional;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface OrderCompleteRepository extends JpaRepository<OrderComplete, Long> {
    List<OrderComplete> findAllByOrderId(Long orderId);

    @Transactional
    @Modifying
    @Query("update OrderComplete oc set oc.complete = true where oc.id = :id")
    void updateById(Long id);

    @Query("select " +
            "oc from OrderComplete oc " +
            "JOIN FETCH oc.order o " +
            "JOIN FETCH o.orderModuleDTO omd " +
            "where oc.complete = :target")
    List<OrderComplete> findAllByComplete(Boolean target);

~~~

1. 처음에는 OrderComplete와 Order 테이블을 패치해 봤는데, 35초에 달하는 쿼리 속도가 약 28초 정도로 줄긴 했지만, 여전히 느리다.
그리고 ordermoduledto 테이블을 패치해 봤는데, 이 결과 약 1.7초로 줄어들어서 문제가 해결되었다.

2. **Where in** 을 이용해서 해결했는데, 이건 직접적으로 N + 1 쿼리를 줄이는 것은 아니지만,
이미 존재하던 코드의 방식으로는 select * from orderitem where orderID = ? 이런 식으로 쿼리를 날리는데,
이걸 where in을 이용해서 select * from orderitem where orderID in (?,?,?,...) 이런 식으로 쿼리를 날리는 방식으로 변경했다.
이는 타 프로젝트에서 쿼리 시간을 개선할 때 가장 큰 효과를 보았던 방식이다.

3. 그리고 **batch size**를 50으로 설정했는데, 프로젝트가 크지 않아서 이 정도의 배치 사이즈로도 큰 효과를 보았다.

### 5. 패치 조인

일단 패치 조인을 이용할 때, 내가 신경쓰였던 점은 List<OrderComplete> findAllByComplete(Boolean target); 이 메소드에서
리턴해야 할 객체가 List<OrderComplete> 인데, 

~~~ sql

    "select " +
            "oc from OrderComplete oc " +
            "JOIN FETCH oc.order o " +
            "JOIN FETCH o.orderModuleDTO omd " +
            "where oc.complete = :target"
            
~~~

이런 식으로 패치 조인을 이용하면 List<OrderComplete> 가 아닌 List<Object[]> 가 리턴될 수 있지 않을까? 라는 걱정이었다.
왜냐면 결과가 OrderComplete, Order, OrderModuleDTO 세 개의 객체가 조인된 객체가 리턴되기 때문이다.

하지만 이런 걱정은 굳이 할 필요가 없었다. 스프링 JPA는 이런 경우에도 List<OrderComplete> 가 리턴된다.

    즉, JPQL 쿼리에서 
    
    select oc from OrderComplete oc... 

    와 같이 특정 엔티티를 선택하면, 
    결과는 List<OrderComplete>로 반환되고, 이 경우 JPA는 OrderComplete 객체를 생성하고 
    패치 조인으로 로드된 관련 엔티티(Order와 OrderModuleDTO)는 해당 객체의 필드에 자동으로 매핑된다.


그리고, 이번에 공부할 수 있었던 건, **OrderComplete와 Order와는 연결되어 있지만, OrderModuleDTO는 연결되어 있지 않아도
패치 조인을 이용해서 Order를 타고 Order에 연결되어 있는 OrderModuleDTO를 가져올 수 있었다는 점이다.**

### 6. 마치며

N + 1 매핑을 해결하는 다양한 방법중에, 패치 조인을 이용한 방법을 정리해 보았다. 엔티티 그래프나 그 외의 
다양한 방법들이 존재하지만 과거에 이 방법을 사용했을 때 가장 효과적이었기 때문에 이 방법을 사용했고,
엔티티 그래프나 그 외의 방법들도 한 번 정리해 보고 싶다.
