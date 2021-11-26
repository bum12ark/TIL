## 개요

### APIs

| 기능 | 마이크로서비스 | URI(API Gateway 사용 시) | HTTP Method |
| --- | --- | --- | --- |
| 상품 목록 조회 | Catalogs Microservice | /catalog-service/catalogs | GET |
| 사용자 별 상품 주문 | Orders Microservice | /order-service/{user-_id}/orders | POST |
| 사용자 별 주문 내역 조회 | Orders Microservice | /order-service/{user-_id}/orders | GET |

### 프로젝트생성

- 의존성 추가
    
    ![Untitled](https://user-images.githubusercontent.com/72686708/143583971-abfe0dba-aa92-4529-992c-6760e0426d76.png)

## 기능 구현

### OrderController java

```java
@RestController
@RequestMapping("/order-service")
@RequiredArgsConstructor
public class OrderController {

    private final Environment environment;
    private final OrderService orderService;

    @GetMapping("/health_check")
    public String status() {
        return String.format("It's Working in Order Service on PORT %s",
                environment.getProperty("local.server.port"));
    }

    @PostMapping("/{userId}/orders")
    public ResponseEntity createOrder(@PathVariable("userId") String userId,
                                      @RequestBody RequestOrder requestOrder) {

        OrderDto orderDto = new OrderDto(requestOrder.getProductId(), requestOrder.getQty(), requestOrder.getUnitPrice());

        Order order = orderService.createOrder(orderDto);

        ResponseOrder responseOrder = new ResponseOrder(order.getProductId(), order.getQty(), order.getUnitPrice(),
                order.getTotalPrice(), order.getCreatedAt(), order.getOrderId());

        return ResponseEntity.status(HttpStatus.CREATED).body(responseOrder);
    }

    @GetMapping("/{userId}/orders")
    public ResponseEntity getOrder(@PathVariable("userId") String userId) {
        Iterable<Order> findOrdersByUserId = orderService.getOrdersByUserId(userId);

        List<ResponseOrder> results = new ArrayList<>();

        findOrdersByUserId.forEach(order -> {
            results.add(new ResponseOrder(order.getProductId(), order.getQty(), order.getUnitPrice(),
                    order.getTotalPrice(), order.getCreatedAt(), order.getOrderId()));
        });

        return ResponseEntity.status(HttpStatus.OK).body(results);
    }
}
```

### OrderServiceImpl java

```java
@Service
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService {

    private final OrderRepository orderRepository;

    @Override
    public Order createOrder(OrderDto orderDto) {
        orderDto.setOrderId(UUID.randomUUID().toString());
        orderDto.setTotalPrice(orderDto.getQty() * orderDto.getUnitPrice());

        Order order = Order.builder()
                .orderId(orderDto.getOrderId())
                .productId(orderDto.getProductId())
                .qty(orderDto.getQty())
                .totalPrice(orderDto.getTotalPrice())
                .unitPrice(orderDto.getUnitPrice())
                .userId(orderDto.getUserId())
                .build();

        return orderRepository.save(order);
    }

    @Override
    public Order getOrderByOrderId(String orderId) {
        return orderRepository.findByOrderId(orderId);
    }

    @Override
    public Iterable<Order> getOrdersByUserId(String userId) {
        return orderRepository.findByUserId(userId);
    }
}
```

### Order entity

```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class Order implements Serializable {

    @Id @GeneratedValue
    private Long id;

    @Column(nullable = false, length = 120, unique = true)
    private String productId;
    @Column(nullable = false)
    private Integer qty;
    @Column(nullable = false)
    private Integer unitPrice;
    @Column(nullable = false)
    private Integer totalPrice;

    @Column(nullable = false)
    private String userId;
    @Column(nullable = false, unique = true)
    private String orderId;

    @Column(nullable = false, updatable = false, insertable = false)
    @ColumnDefault(value = "CURRENT_TIMESTAMP")
    private LocalDateTime createdAt;

    @Builder
    public Order(String productId, Integer qty, Integer unitPrice, Integer totalPrice, String userId, String orderId) {
        this.productId = productId;
        this.qty = qty;
        this.unitPrice = unitPrice;
        this.totalPrice = totalPrice;
        this.userId = userId;
        this.orderId = orderId;
    }
}
```
