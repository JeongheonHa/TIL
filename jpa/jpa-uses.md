# JPA 활용 1, 2 정리

## 1. 엔티티를 노출하지 마라.
보통 View와 비즈니스 로직은 별개의 라이프 사이클을 갖기 때문에 서로 독립적인 개발이 필요하다. 이 때 엔티티가 변경되어 api 스펙까지 변경 된다면 독립적인 개발이 불가능하다. 따라서, api와 통신하는 별도의 DTO를 만들어 독립적인 개발이 가능하게 만들어준다. (요청, 응답 둘다)

## 2. *ToOne 조회 및 최적화

### 2.1 DTO 적용 + 페치 조인으로 조회
*ToOne 관계는 페치 조인을 사용하여 N + 1 문제 해결

```java
// 주문 조회
@GetMapping("/api/v3/simple-orders")
public List<SimpleOrderDto> ordersV3() {
    List<Order> orders = orderRepository.findAllWithMemberDelivery();
    List<SimpleOrderDto> result = orders.stream()
            .map(o -> new SimpleOrderDto(o))
            .collect(toList());

    return result;
  }
```

```java
public List<Order> findAllWithMemberDelivery() {
    return em.createQuery(
                "select o from Order o" +
                " join fetch o.member m" +
                " join fetch o.delivery d", Order.class)
            .getResultList();
}
```

### 2.2 DB에서 DTO로 직접 조회
엔티티 자체가 아닌 필요한 데이터만 골라 가져오기 싶은 경우 유용 (ex. api 스펙에 맞는 데이터를 한번에 조회)
- API 스펙에 맞춘 코드가 리포지토리에 들어가기 때문에 재사용성이 떨어진다. (API 스펙이 바뀌면 해당 리포지토리를 수정해야한다.)
- 따라서 일반 비즈니스 로직과 api 스펙에 맞춘 로직의 계층을 분리하는 것이 좋다.

```java
// OrderApiController
@GetMapping("/api/v4/simple-orders")
public List<OrderSimpleQueryDto> ordersV4() {
    return orderSimpleQueryRepository.findOrderDtos();
}
```

```java
// OrderSimpleQueryRepository
public List<OrderSimpleQueryDto> findOrderDtos() {
    return em.createQuery(
                "select new jpabook.jpashop.repository.order.simplequery.OrderSimpleQueryDto(o.id, m.name, o.orderDate, o.status, d.address)" + 
                " from Order o" +
                " join o.member m" +
                " join o.delivery d", OrderSimpleQueryDto.class)
            .getResultList();
}
```

```java
public class OrderSimpleQueryDto {
    private Long orderId;
    private String name;
    //...
}
```

### 2.3 쿼리 방식 선택 권장 순서
1. 엔티티로 접근
    1. 페치 조인으로 쿼리 수 최적화 (대부분의 성능 이슈 해결)
3. 그래도 안되면 DTO로 직접 조회
4. DTO로 직접 접근해도 안되면 JPA가 제공하는 NativeSQL이나 JDBC Template을 사용해서 SQL을 직접 사용

## 3. OneToMany 조회 및 최적화

### 3.1 DTO 적용 + distinct + 페치 조인으로 한번에 가져오기
주문 엔티티를 가져오는 시점에 쿼리를 한번만 날려 N + 1 문제를 해결했지만 페이징 불가

```java
@GetMapping("/api/v2/orders")
public List<OrderDto> ordersV2() {
    List<Order> orders = orderRepository.findAllWithTeam();
    List<OrderDto> result = orders.stream()
            .map(o -> new OrderDto(o))
            .collect(toList());
    return result;
}
```

```java
// OrderRepository
public List<Order> findAllWithItem() {
    return em.createQuery(
                "select distinct o from Order o" +
                " join fetch o.member m" +
                " join fetch o.delivery d" +
                " join fetch o.orderItems oi" +
                " join fetch oi.item i", Order.class)
            .getResultList();
}
```

- 컬렉션 내부의 엔티티도 api에 스펙 중 하나이므로 DTO로 변환 해야한다. (OrderItem -> OrderItemDto)

```java
public class OrderDto {
    private Long orderId;
    //...
    private List<OrderItemDto> orderItems;

    public OrderDto(Order order) {
        //...
        orderItems = order.getOrderItems().stream()
                .map(orderItem -> new OrderItemDto(orderItem))
                .collect(toList());
    }
}
```

```java
public class OrderItemDto {
    private String itemName;
    //...
}
```

### 3.2 BatchSize로 가져오기
ManyToOne 관계와 *ToOne 관계의 쿼리를 따로 보내 데이터를 가져온다. (페이징까지 고려해 N + 1 문제 해결)

```yaml
# application.yml 설정 정보
spring: 
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 1000
```

```java
// OrderRepository
public List<Order> findAllWithMemberDelivery(int offset, int limit) {
    return em.createQuery(
                "select o from Order o" +
                " join fetch o.member m" +
                " join fetch o.delivery d", Order.class)
            .setFirstResult(offset)
            .setMaxResults(limit)
            .getResultList();
}
```

### 3.3 DB에서 DTO로 직접 조회 - v1
ManyToOne 관계와 *ToOne 관계의 조회 쿼리를 따로 보낸다. (N + 1 문제 발생)

```java
// OrderQueryRopository
public List<OrderQueryDto> findOrderQueryDtos() { 
    List<OrderQueryDto> result = findOrders(); // *ToOne 관계의 DTO 따로 가져오기

    result.forEach(o -> { // ManyToOne 관계의 프록시 컬렉션 래퍼<Dto>에서 가져오기 (N + 1 문제 발생, 1번의 주문에 프록시 컬렉션 래퍼에서 Dto를 한번씩 다 조회 (N))
                List<OrderItemQueryDto> orderItems = findOrderItems(o.getOrderId());
                o.setOrderItems(orderItems);
            });

    return result;
}

private List<OrderQueryDto> findOrders() {
    return em.createQuery(
                "select new jpabook.jpashop.repository.order.query.OrderQueryDto(o.id, m.name, o.orderDate, o.status, d.address)" +
                " from Order o" +
                " join o.member m" +
                " join o.delivery d", OrderQueryDto.class)
            .getResultList();
}

private List<OrderItemQueryDto> findOrderItems(Long orderId) {
        return em.createQuery(
                "select new jpabook.jpashop.repository.order.query.OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count)" +
                " from OrderItem oi" +
                " join oi.item i" +
                " where oi.order.id = : orderId", OrderItemQueryDto.class)
                .setParameter("orderId", orderId)
                .getResultList();
}
```

### 3.4 DB에서 DTO로 직접 조회 - v2. 컬렉션 조회 최적화
ManyToOne 관계와 *ToOne 관계의 조회 쿼리를 따로 보내며 ManyToOne 관계에서는 in 쿼리를 사용한다. (N + 1 문제 해결)
- BatchSize의 내부 동작을 구현하는 방식인 것 같다.
- orderIds를 한번에 보내느냐 여러 번 보내느냐에 따라 쿼리의 횟수가 달라질 수 있을 것 같다.

```java
// OrderQueryRopository
public List<OrderQueryDto> findAllByDto_optimization() {
    List<OrderQueryDto> result = findOrders(); // 1. *ToOne 관계의 DTO 조회
    
    List<Long> orderIds = toOrderIds(result); // 2. order의 식별자 값을 리스트로 전달

    Map<Long, List<OrderItemQueryDto>> orderItemMap = findOrderItemMap(orderIds); // 3. orderIds를 in 쿼리로 전달하여 Map<ID, List<DTO>>로 반환

    result.forEach(o -> o.setOrderItems(orderItemMap.get(o.getOrderId()))); // 4. 루프를 돌면서 컬렉션 추가(추가 쿼리 실행X)

    return result;
}

// 
private List<Long> toOrderIds(List<OrderQueryDto> result) {
    return result.stream()
            .map(o -> o.getOrderId())
            .collect(Collectors.toList());
}

private Map<Long, List<OrderItemQueryDto>> findOrderItemMap(List<Long> orderIds) {
    List<OrderItemQueryDto> orderItems = em.createQuery(
                    "select new jpabook.jpashop.repository.order.query.OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count)" +
                    " from OrderItem oi" +
                    " join oi.item i" +
                    " where oi.order.id in :orderIds", OrderItemQueryDto.class)
                .setParameter("orderIds", orderIds)
                .getResultList();

    return orderItems.stream()
            .collect(Collectors.groupingBy(OrderItemQueryDto::getOrderId)); // id -> List<Dto> 형태로 반환하여 4번 과정에서 O(1)로 매핑할 수 있다. 
}
```

### 3.5 DB에서 DTO 직접 조회 - v3. 플랫 데이터 최적화
- 쿼리는 한번이지만 조인으로 인해 DB에서 애플리케이션에 전달하는 데이터에 중복 데이터가 추가 되므로 상황에 따라 v2버전 보다 성능이 더 느릴 수 있다.
- 애플리케이션에 추가 작업이 상당히 많다.
- 페이징 불가

```java
// OrderApiController
@GetMapping("/api/v6/orders")
public List<OrderQueryDto> ordersV6() {
    List<OrderFlatDto> flats = orderQueryRepository.findAllByDto_flat();
    return flats.stream()
            .collect(groupingBy(o -> new OrderQueryDto(o.getOrderId(), o.getName(), o.getOrderDate(), o.getOrderStatus(), o.getAddress()), 
                     mapping(o -> new OrderItemQueryDto(o.getOrderId(), o.getItemName(), o.getOrderPrice(), o.getCount()), toList())))
            .entrySet().stream()
            .map(e -> new OrderQueryDto(e.getKey().getOrderId(),e.getKey().getName(), e.getKey().getOrderDate(), e.getKey().getOrderStatus(), e.getKey().getAddress(), e.getValue()))
            .collect(toList());
}
```

```java
// OrderQueryRepository
public List<OrderFlatDto> findAllByDto_flat() {
      return em.createQuery(
                    "select new jpabook.jpashop.repository.order.query.OrderFlatDto(o.id, m.name, o.orderDate, o.status, d.address, i.name, oi.orderPrice, oi.count)" +
                    " from Order o" +
                    " join o.member m" +
                    " join o.delivery d" +
                    " join o.orderItems oi" +
                    " join oi.item i", OrderFlatDto.class)
                .getResultList();
}
```

### 3.6 쿼리방식 선택 권장 순서
1. 엔티티 조회 방식으로 접근
    1. 페치조인으로 쿼리 수 최적화
    2. 컬렉션 최적화
        1. 페이징 필요 : BatchSize
        2. 페이징 필요 X : 페치 조인 + distinct 사용
2. 엔티티 조회 방식으로 해결이 안되면 DTO 조회 방식 사용
3. DTO 조회 방식으로 해결이 안되면 NativeSQL or Jdbc Template 사용


## 4. OSIV 와 성능 최적화
- `Open Session In View`: Hibernate
- `Open EntityManager In View`: JPA

<img src="https://user-images.githubusercontent.com/108064146/235926080-031b3379-6a3f-45de-8661-b3f67870ce5a.png">

### 4.1 OSIV ON

<img src="https://user-images.githubusercontent.com/108064146/235923362-89f58851-e042-4319-833b-6b61e6b93801.png">

- spring.jpa.open-in-view: true (기본값)
- 지금까지 View Template이나 API 컨트롤러에서 지연로딩이 가능했던 이유는 OSIV 전략이 true일 경우 최초 데이터베이스 커넥션 시작 시점부터 API 응답이 끝날 때 까지 영속성 컨텍스트와 커넥션을 유지시켜주기 때문에다.
- 하지만, OSIV 전략은 너무 오랫동안 커넥션을 갖고있기 때문에 실시간 트래픽이 중요한 애플리케이션에서는 커넥션이 모자랄 수 있다.

### 4.2 OSIV OFF

<img src="https://user-images.githubusercontent.com/108064146/235923397-abe6455a-b658-4369-aa7a-32026112652d.png">

- spring.jpa.open-in-view: false
- 트랜잭션이 종료될 때 영속성 컨텍스트를 닫고, 커넥션도 반환하여 커넥션을 낭비하지 않는다.
- 하지만, 모든 지연로딩을 트랜잭션 안에서 처리해야하기 때문에 하나의 서비스 계층에서 단순한 비즈니스 로직의 쿼리와 복잡한 조회 쿼리가 함께 전달되어 쿼리의 복잡성을 늘린다.

### 4.3 커맨드와 쿼리 책임 분리 (CQRS)
커맨드와 쿼리를 분리하면 OSIV를 끈 상태로 복잡성을 관리할 수 있다.

- 보통 비즈니스 로직은 몇 개의 엔티티를 등록/수정(command) 하는 것이 전부이기 때문에 성능상에는 문제가 되지 않는다. (ex. 회원 가입)
- 복잡한 화면을 출력하기 위한 쿼리(select query)는 화면에 맞추어 성능을 최적화하는 것이 중요하다.
- 따라서, 이 둘의 관심사를 분리하는 것이 유지보수 관점에서 좋다.

```
// 계층 구조 예시
- service (p)
    - OrderService (c) : 핵심 비즈니스 로직
    - OrderQueryService (c) : 화면이나 api에 맞춘 서비스 (주로 읽기 전용 트랜잭션 사용)
```

<img src = "https://user-images.githubusercontent.com/108064146/235936752-f0aef786-6c9d-4521-a7fc-ffc92da2cf58.JPG">

```java
// OrderApiController
@GetMapping("/api/orders")
public List<OrderDto> orders() {
    return orderQuerySerivce.orders();
}
```

```java
// OrderQueryService
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class OrderQueryService {

    private final OrderRepository orderRepository;

    public List<OrderDto> orders() {
        List<Order> orders = orderRepository.findAllWithItem();

        List<OrderDto> result = orders.stream()
                .map(o -> new OrderDto(o))
                .collect(Collectors.toList());

        return result;
    }
}
```

- 즉, 복잡한 조회 쿼리를 담당하는 Service 계층 따로 단순한 커맨드 쿼리를 담당하는 Service 계층 따로 만들어 조회 Service 계층에 @Transactional를 걸어주어 지연로딩을 초기화함과 동시에 쿼리의 복잡성을 낮출 수 있다.

- @Transactional(readOnly = true): 조회만 담당하는 Service 계층에 readOnly를 활성화하면 스냅샷을 생성하지 않고 변경 감지를 수행하지 않아 성능 최적화를 할 수 있다.

## Reference
> 실전! 스프링 부트와 JPA 활용1, 2 [김영한]
