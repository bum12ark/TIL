### Zuul vs. Spring Cloud Gateway

1. Blocking vs non-Blocking
    - Zuul 1.0 은 Bolcking 방식 SCG는 **non-Blocking** 방식
    - 또한 Zuul은 Spring과의 호환성에서 문제가 있음
    - SCG는 비동기 방식과 함수형 프로그래밍 지원
2. Tomcat vs Netty
    - Zuul은 Tomcat을 사용하고 SCG는 **Netty**를 사용

### Spring Cloud Gateway 구성도

![Untitled](https://user-images.githubusercontent.com/72686708/142436840-a27ed7bd-98bd-450e-b847-8bae92cdc37c.png)


## Spring Cloud Gateway 프로젝트 생성

1. 의존성
    - DevTools, Eureka Discovery Client, **Spring Cloud Routing** **Gateway**
2. application.yml
    
    ```yaml
    server:
      port: 8000
    
    eureka:
      client:
        register-with-eureka: false
        fetch-registry: false
        service-url:
          defaultZone: http://localhost:8761/eureka
    spring:
      application:
        name: apigateway-service
      cloud:
        gateway:
          routes:
            - id: first-service
              uri: http://localhost:8081
              predicates:
                - Path=/first-service/**
            - id: second-service
              uri: http://localhost:8082
              predicates:
                - Path=/second-service/**
    ```
    
    - `routes` : List 형식으로 로드밸런싱 정보를 기입
        - `predicates` : 사전 조건 기입

## Spring Cloud Gateway - Filter

![Untitled 1](https://user-images.githubusercontent.com/72686708/142436870-43ae848d-d486-49fd-bcb6-d18be9a628aa.png)

1. 요청정보가 들어옴
2. Predicate 에서 요청에 대한 사전 조건을 분기, 판단
3. 사전필터와 사후필터로 요청 처리 가능
    - 작업 방법은 property, java code 둘다 가능

### Java Code를 사용한 필터링

- API Gateway를 통해 Request Header와 Response Header 에 정보를 추가해보자
- 또한 API Gateway에서 추가한 임의 header 값들이 라우팅된 서버에서 정상적으로 확인되는지 검증해보자
- 실습을 위해 기존 application.yml 파일의 routes 정보는 주석 처리

**FilterConfig.java**

```java
@Configuration
public class FilterConfig {

    @Bean
    public RouteLocator gatewayRoutes(RouteLocatorBuilder builder) {
        return builder.routes()
                .route(
                        predicateSpec -> predicateSpec.path("/first-service/**")
                                .filters(filterSpec -> filterSpec.addRequestHeader("first-request", "first-request-header")
                                                                 .addResponseHeader("first-response", "first-response-header"))
                                .uri("http://localhost:8081")
                )
                .route(
                        predicateSpec -> predicateSpec.path("/second-service/**")
                                .filters(filterSpec -> filterSpec.addRequestHeader("second-request", "second-request-header")
                                                                 .addResponseHeader("second-response", "second-response-header"))
                                .uri("http://localhost:8082")
                )
                .build();
    }
}
```

- `filter` : 해당 메소드를 통해 임의 header 값을 추가

**FisrtServiceController.java**

```java
@GetMapping("/message")
public String message(@RequestHeader("first-request") String firstRequestHeader) {
    log.info("header = {}", firstRequestHeader);
    return "Hello world in First Service";
}
```

- 헤더를 출력하는 메소드 추가

**테스트 결과**

```
c.e.firstservice.FirstServiceController  : header = first-request-header
```

![Untitled 2](https://user-images.githubusercontent.com/72686708/142436900-617c100c-173d-4bad-ab51-7f68591b6cb7.png)

### application 파일을 통한 필터링

```yaml
spring:
  application:
    name: apigateway-service
  cloud:
    gateway:
      routes:
        - id: first-service
          uri: http://localhost:8081
          predicates:
            - Path=/first-service/**
          filters:
            - AddRequestHeader=first-request, first-request-header
            - AddResponseHeader=first-response, first-response-header
        - id: second-service
          uri: http://localhost:8082
          predicates:
            - Path=/second-service/**
          filters:
            - AddRequestHeader=second-request, second-request-header
            - AddResponseHeader=second-response, second-response-header
```

## Custom Filter 적용

```java
@Component @Slf4j
public class CustomFilter extends AbstractGatewayFilterFactory<CustomFilter.Config> {

    public CustomFilter() {
        super(Config.class);
    }

    public static class Config {
        // Put the configuration properties
    }

    @Override
    public GatewayFilter apply(Config config) {
        // Custom Pre Filter
        return (exchange, chain) -> {
            ServerHttpRequest request = exchange.getRequest();
            ServerHttpResponse response = exchange.getResponse();
            
            log.info("Custom PRE filter: request id -> {}", request.getId());
            
            // Custom Post Filter
            return chain.filter(exchange)
                    .then(Mono.fromRunnable(() -> {
                        log.info("Custom POST filter: response code -> {}",response.getStatusCode());
                    }));
        };
    }
}
```

- Netty 비동기 서버에서는 HttpServletRequest 대신 ServerHttpRequest 를 사용한다.
- Gateway를 구현하기 위해서는 GatewayFilterFactory를 구현해야 하며, 상속할 수 있는 추상 클래스가 `AbstractGatewayFilterFactory` 이다
- exchange
    - 서비스 요청/응답값을 담고있는 변수로, 요청/응답값을 출력하거나 반환할 때 사용
    - 리턴받는 응답값은 Mono.fromRunnable (Spring Web Flux에서 사용)
- config
    - application 파일에서 선언한 각 filter의 args(인자값) 사용을 위한 클래스
- application.yml
    
    ```yaml
    spring:
      cloud:
        gateway:
          routes:
            - id: first-service
              uri: http://localhost:8081
              predicates:
                - Path=/first-service/**
              filters:
                - CustomFilter
    ```
    
- Controller
    
    ```java
    @GetMapping("/check")
    public String check() {
        return "Hi, there. This is a message from Second Service.";
    }
    ```
    

**테스트**

- 호출
    - [http://localhost:8000/first-service/check](http://localhost:8000/first-service/check)
- 로그
    
    ```java
    2021-11-16 21:06:31.820  INFO 5964 --- [ctor-http-nio-3] c.e.a.filter.CustomFilter                : Custom PRE filter: request id -> 5c19bc5b-3
    2021-11-16 21:06:31.823  INFO 5964 --- [ctor-http-nio-3] c.e.a.filter.CustomFilter                : Custom POST filter: response code -> 200 OK
    ```
    

## Global Filter

- 공통적으로 사용될 수 있는 필터를 만들어 보자
- Custom Filter는 application 파일에 필요한 라우팅 서버에 매번 적용해야한다.
- 필요한 환경 설정 정보를 가지고 있는 Config 클래스와 application 파일을 정의하여 사용해보자.
    - Config 클래스의 필드들에 해당하는 값을 application 파일에서 등록한다.
- 글로벌 필터는 모든 필터의 **가장 처음 실행되고 가장 마지막에 종료**된다.

**GlobalFilter**

```java
@Component @Slf4j
public class GlobalFilter extends AbstractGatewayFilterFactory<GlobalFilter.Config> {

    public GlobalFilter() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        // Custom Pre Filter
        return (exchange, chain) -> {
            ServerHttpRequest request = exchange.getRequest();
            ServerHttpResponse response = exchange.getResponse();

            log.info("Global Filter baseMessage = {}", config.getBaseMessage());

            if (config.isPreLogger()) {
                log.info("Global Filter Start: request id = {}", request.getId());
            }
            // Custom Post Filter
            return chain.filter(exchange)
                    .then(Mono.fromRunnable(() -> {
                        if (config.isPostLogger()) {
                            log.info("Global Filter End: response code = {}", response.getStatusCode());
                        }
                    }));
        };
    }

    @Data
    public static class Config {
        private String baseMessage;
        private boolean preLogger;
        private boolean postLogger;
    }

}
```

**application.yml**

```yaml
spring:
  cloud:
    gateway:
      default-filters:
        - name: GlobalFilter
          args:
            baseMessage: Spring Cloud Gateway Global Filter
            preLogger: true
            postLogger: true
```

- `defulat-filters`
    - `name`: 필터 클래스 이름
    - `args`: Config의 변수 명 작성

**테스트**

```yaml
2021-11-16 21:21:08.777  INFO 8920 --- [ctor-http-nio-3] c.e.a.filter.GlobalFilter                : Global Filter baseMessage = Spring Cloud Gateway Global Filter
2021-11-16 21:21:08.778  INFO 8920 --- [ctor-http-nio-3] c.e.a.filter.GlobalFilter                : Global Filter Start: request id = f68f5350-1
2021-11-16 21:21:08.778  INFO 8920 --- [ctor-http-nio-3] c.e.a.filter.CustomFilter                : Custom PRE filter: request id -> f68f5350-1
2021-11-16 21:21:08.901  INFO 8920 --- [ctor-http-nio-3] c.e.a.filter.CustomFilter                : Custom POST filter: response code -> 200 OK
2021-11-16 21:21:08.901  INFO 8920 --- [ctor-http-nio-3] c.e.a.filter.GlobalFilter                : Global Filter End: response code = 200 OK
```

- 실행순서
    1. Global Pre Filter
    2. Custom Pre Filter
    3. Custom Post Filter
    4. Global Post Filter

## Load Balancer

- 라운드 로빈  방식으로 로드 밸런싱 된다.

### Spring Cloud Gateway - Eureka 연동

- Eureka 서버에 API Gateway를 등록하는 예제

![Untitled 3](https://user-images.githubusercontent.com/72686708/142436964-64b4ebaf-dacf-4fed-a618-141c3b7ed1b6.png)

1. 사용자의 요청 업보를 API Gateway가 받음
2. 유레카로 부터 마이크로 서비스의 위치 정보를 받음
3. 해당 정보로 포트 포워딩

**기동 순서**

1. 유레카 서버 기동
2. API Gateway, 마이크로 서비스 기동

**마이크로 서비스**

- 유레카 클라이언트 의존성 추가
- apllication 파일의 유레카 클라이언트 속성 추가
    
    ```yaml
    eureka:
      client:
        fetch-registry: true
        register-with-eureka: true
        service-url:
          defaultZone: http://127.0.0.1:8786/eureka
    ```
    

**게이트 웨이 서버**

- 유레카 클라이언트 의존성 추가
- apllication 파일의 유레카 클라이언트 속성 추가
    
    ```yaml
    eureka:
      client:
        fetch-registry: true
        register-with-eureka: true
        service-url:
          defaultZone: http://127.0.0.1:8786/eureka
    ```
    
- application route 정보 변경
    
    ```yaml
    spring:
      cloud:
        routes:
            - id: first-service
              uri: lb://MY-FIRST-SERVICE
              predicates:
                - Path=/first-service/**
    ```
    
    - 유레카 서버에 등록된 application 이름으로 지정
    - uri: 유레카에 등록되어있는 MY-FIRST-SERVICE로 포워딩 처리
    - `lb` : load balancing의 약자
    - `predicates.Path` : Path에 작성된 정보로 요청이 올경우 uri로 포워딩
    - 기존의 http 정보를 이용하여 로드 밸런싱이 되는것이 아니라 유레카 서버에서 제공받은 정보로 로드 밸런싱

**테스트**

- url: [http://localhost:8000/second-service/welcome](http://localhost:8000/second-service/welcome)

### Load Balancer

![Untitled 4](https://user-images.githubusercontent.com/72686708/142437006-7cecda5a-bfe1-4bdc-bcb0-55f5be95a24b.png)

- 서로 다른 포트를 가진 마이크로 서비스를 두개 생성한다.
- 랜덤 방식으로 포트를 생성해도 된다.
- 포트 번호가 어떻게 생성될지 모를 때도 유레카 서버를 통해 인스턴스의 정보를 알 수 있다는 장점이 있다.

**컨트롤러**

```java
@Slf4j @RequiredArgsConstructor
public class FirstServiceController {

    private final Environment environment;

    @GetMapping("/check")
    public String check(HttpServletRequest request) {
        log.info("Server port = {}", request.getServerPort());

        return String.format("Hi, there. This is a message from First Service on PORT %s",
                environment.getProperty("local.server.port"));
    }
}
```

- HttpServeltRequest 객체 또는 Environment 객체를 사용하여 포트 번호를 출력하는 로직 추가
- 해당 로깅으로 어느 서버로 로드 밸런싱 되는지 확인

**테스트**

- url: [http://localhost:8000/first-service/check](http://localhost:8000/first-service/check)
- 콘솔
    
    ![Untitled 5](https://user-images.githubusercontent.com/72686708/142437044-6969497a-cf6c-4f68-a366-53ee4be57140.png)
    
    ![Untitled 6](https://user-images.githubusercontent.com/72686708/142437055-afc52163-3241-41df-a30d-9b1687055124.png)
    
- 요청이 들어올 때 마다 등록된 인스턴스로 **라운드 로빈** 방식으로 로드 밸런싱 되는 것을 확인 가능하다.
