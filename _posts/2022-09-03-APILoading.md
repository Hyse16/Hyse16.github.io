---
layout: single
title: "지연로딩과 조회 성능 최적화"
categories: StudyJPA-Application
toc: true
author_profile: true
sidebar:
    nav: "docs"
---
# 지연 로딩과 조회 성능 최적화
주문 + 배송정보 + 회원을 조회하는 API

## 엔티티를 직접 노출
````java

@RestController
@RequiredArgsConstructor

public class OrderSimpleApiController{

    private final OrderRepository orderRepository;

    @GetMaaping("/api/v1/simple-orders")
    public List<Order> ordersV1(){
        List<Order> all = orderRepository.findAllByString(new OrderSearch());
        for(Order order :all){
            order.getMember().getName();
            order.getDeliver().getAddress();
        }
        return all;
    }
}
/* 주의 : 엔티티를 직접 노출할 때는 양방향 연관관계가 걸린 곳은 꼭 

@JsonIgnore를 사용해야한다. 안그러면 무한 루프가 걸린다.*/
````

<br>

## 엔티티를 DTO로 변환
````java
@GetMapping("/api/v2/simple-orders")
public List<SimpleOrderDto> ordersV2(){
    List<Order> orders = orderRepository.findAll();
    List<SimpleOrderDto> result = orders.stream()
    .map(o-> new SimpleOrderDto(o))

    return result;
}

@Data
static class SimpleOrderDto{
    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;

    public SimpleOrderDto(Order order){
        orderId = order.getId();
        name = order.getMember().getName();
        orderDate = order.getOrderDate();
        orderStatus = order.getStatus();
        address = order.getDelivery().getAddress();
    }
}

/* 쿼리가 총 1+N+N번 실행된다 */
````

<br>

## 엔티티를 DTO로 변환 - 페치 조인 최적화
````java
@GetMapping("/api/v3/simple-orders")
public List<SimpleOrderDto> ordersV3(){
    List<Order> orders = 
    orderRepository.findAllwithMemberDelivery();
    List<SimpleOrderDto> result = orders.stream()
    .map(o-> new SimpleOrderDto(o))
    .collect(ToList());
    return result;
}


//orderRepository 추가 코드//
public Lsit<Order> findAllwithMemberDelivery(){
    return em.createQuery("select o from Order o"+
    "join fetch o.member m"+
    "join fetch o.delivery d",Order.class)
    .getResultList();
}

// 엔티티를 페치조인을 사용해서 쿼리 1번에 조회//
// 페치 조인으로 지연로딩X//
````

<br>

## JPA에서 DTO로 바로조회
````java
@GetMapping("/api/v4/simple-orders")
public List<OrderSimpleQueryDto> ordersV4(){
    return orderSimpleQueryRepository.findOrderDtos();
}

//OrderSimpleQueryDto 리포지토리 생성//

@Repository
@RequiredArgsConstructor
public class OrderSimpleQueryRepository{
    
    priavate final EntityManger em;

    public List<OrderSimpleQueryDto> findOrderDtos(){
        return em.createQuery(
            "select new jpabook.jpashop.repository.
            order.simplequery.OrderSimpleQueryDto
            (o.id,m.name,o.orderDate, o.status, d.address)"+
            "from Order o"+
            "join o.member m"+
            "join o.delivery d",
            OrderSimpleQueryDto.class)
            .getResultList();
    }
}

//OrderSimpleQueryDto //
 @Data
public class OrderSimpleQueryDto {
    private Long orderId;
    private String name;
    private LocalDateTime orderDate; 
    private OrderStatus orderStatus;
    private Address address;
      public OrderSimpleQueryDto(
        Long orderId, String name, LocalDateTime
        orderDate, OrderStatus orderStatus, Address address) {
          this.orderId = orderId;
          this.name = name;
          this.orderDate = orderDate;
          this.orderStatus = orderStatus;
          this.address = address;
} }
/* Select 절에서 원하는 데이터를 직접 선택하므로 용량 최적화, 리포지토리 재사용성 떨어짐, API스펙에 맞춘 코드가 리포지토리에 들어가는 단점*/
````

<br>

## 정리
엔티티를 DTO로 변환하거나, DTO로 바로 조회하는 두가지 방법은 각각 장단점이 있다. 

둘중 상황에 따라 방법을 선택하면 된다.
1. 엔티티를 DTO로 변환 하는 방법 선택
2. 필요하면 페치 조인으로 성능 최적화
3. 그래도 안되면 DTO로 직접 조회
4. 최후의 방법은 JPA가 제공하는 네이티브 SQL이나 스프링 JDBC Template를 사용