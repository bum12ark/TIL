## 개요

### APIs

| 기능           | 마이크로서비스        | URI(API Gateway 사용 시)  | HTTP Method |
| -------------- | --------------------- | ------------------------- | ----------- |
| 상품 목록 조회 | Catalogs Microservice | /catalog-service/catalogs | GET         |

### 프로젝트 생성

- 의존성 추가

  ![Untitled](https://user-images.githubusercontent.com/72686708/143407155-581ff2fd-6c42-425d-b4bd-49a3bcece8cb.png)

- application 파일

  ```yaml
  server:
    port: 0
  
  spring:
    application:
      name: catalog-service
    jpa:
      hibernate:
        ddl-auto: create-drop
      show-sql: true
      generate-ddl: true
      defer-datasource-initialization: true
  ```

  - Spring Boot 2.5에서는 SQL Script DataSource Initialization 기능이 변경되었습니다. 따라서, JPA에 의한 테이블 생성이 자동으로 되지 않을 수 있습니다.
  - 기본적으로, data.sql 스크립트는 Hibernate가 초기화 되기 전에 실행되어야 하는데, 테이블이 자동으로 생성되지 못해서 insert 구문의 오류가 발생할 수 있습니다.
  - 테이블을 생성하기 위해 schema.sql 스크립트를 추가하시거나, defer-datasource-initialization: true 아래와 같은 설정을 추가하시고 실행

- data.sql

  ```yaml
  INSERT INTO CATALOG(product_id, product_name, stock, unit_price)
  VALUES('CATALOG-001', 'Berlin', 100, 1500,);
  
  INSERT INTO CATALOG(product_id, product_name, stock, unit_price)
  VALUES('CATALOG-002', 'Seoul', 110, 1000, SYSDATE);
  
  INSERT INTO CATALOG(product_id, product_name, stock, unit_price)
  VALUES('CATALOG-003', 'Tokyo', 120, 2000);
  ```

### API Gateway Service에 마이크로 서비스 등록

```yaml
spring:
  application:
    name: apigateway-service
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://USER-SERVICE
          predicates:
            - Path=/user-service/**
        - id: catalog-service
          uri: lb://CATALOG-SERVICE
          predicates:
            - Path=/catalog-service/**
```

## 기능 구현 - **모든 상품 목록 불러오기**

### 컨트롤러 및 서비스 구현

```java
@RestController
@RequestMapping("/catalog-service")
@Slf4j @RequiredArgsConstructor
public class CatalogController {

    private final Environment environment;
    private final CatalogService catalogService;

    @GetMapping("/health_check")
    public String status() {
        return String.format("It's Working in Catalog Service on PORT %s",
                environment.getProperty("local.server.port"));
    }

    @GetMapping("/catalogs")
    public ResponseEntity getUsers() {
        Iterable<Catalog> findCatalogs = catalogService.getAllCatalogs();

        List<ResponseCatalog> results = new ArrayList<>();
        findCatalogs.forEach(catalog -> {
            results.add(new ResponseCatalog(catalog.getProductId(), catalog.getProductName(),
                    catalog.getUnitPrice(), catalog.getStock(), catalog.getCreatedAt()));
        });

        return ResponseEntity.status(HttpStatus.OK).body(results);
    }

}
```

```java
@Service
@Slf4j @RequiredArgsConstructor
public class CatalogServiceImpl implements CatalogService {

    private final CatalogRepository catalogRepository;

    @Override
    public Iterable<Catalog> getAllCatalogs() {
        return catalogRepository.findAll();
    }
}
```

### 엔티티 구현

```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class Catalog implements Serializable {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(nullable = false, length = 120, unique = true)
    private String productId;
    @Column(nullable = false)
    private String productName;
    @Column(nullable = false)
    private Integer stock;
    @Column(nullable = false)
    private Integer unitPrice;

    @Column(nullable = false, updatable = false, insertable = false)
    @ColumnDefault(value = "CURRENT_TIMESTAMP")
    private LocalDateTime createdAt;
}
```

- `@ColumnDefault` : value의 함수를 실행 (h2 DB의 current_timestamp)

## 테스트

```java
@WebMvcTest(CatalogController.class)
class CatalogControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    @DisplayName("모든 상품 정보 가져오기")
    void getAllCatalogsTest() throws Exception {
        // Given
        // When & Then
        mockMvc.perform(get("/catalog-service/catalog"))
                .andExpect(status().isOk())
                .andDo(print());
    }
}
```

