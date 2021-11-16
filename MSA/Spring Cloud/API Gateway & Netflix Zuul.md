## API Gateway Service

![Untitled](https://user-images.githubusercontent.com/72686708/141927202-bbf54b3d-ff7d-439c-9497-1945fc37e33d.png)

- 클라이언트에 노출되는 인터페이스를 간소화, 안정화하는 프록시 역할을 수행
- **장점**
    - 시스템의 내부 구조는 숨기고 외부 요청에 대하여 적절한 형태로 가공하여 응답할 수 있다
- **특징**
    - 인증 및 권한 부여에 대한 단일 작업 가능
    - 마이크로 서비스 검색 통합
    - 응답 캐싱
    - 정책, 회로 차단기 및 QoS 다시 시도
    - 속도 제한
    - 부하 분산 (로드 밸런싱)
    - 로깅, 추적, 상관 관계
    - 헤더, 쿼리 문자열 및 청구 변환
    - IP 허용 목록에 추가

## Netflix Ribbon

- Spring Cloud에서의 MSA간 통신
    1. RestTemplate
        
        ```java
        RestTemplate restTemplate = new RestTemplate();
        restTemplate.getForObject("http://localhost:8080/", User.class, 200);
        ```
        
    2. FeignClient
        
        ```java
        @FeignClient("stores")
        public interface StoreClient {
            @RequestMapping(method = RequestMethod.GET, value="/stores")
            List<Store> getStores();
        }
        ```
        
- Ribbon : **Client side** Load Balancer
    - **서비스 이름으로 호출**
    - Health Check
    - 문제점: 함수형 프로그래밍과 호환이 되지 않음, 비동기 처리 지원 불가
    - **Spring Cloud Ribbon은 Spring Boot 2.4에서 Maintenance 상태**

## Netfilx Zuul - 프로젝트 생성

- Netflix 에서 구현한 API Gateway 서비스
- 구성
    - First Service (Netflix Zuul에 등록할 인스턴스)
    - Second Service (Netflix Zuul에 등록할 인스턴스)
    - Netflix Zuul (Gateway)
        - Routing
        - API gateway
- **Spring Cloud Zuul은 Spring Boot 2.4에서 Maintenance 상태**
- 현재는 Spring Cloud Gateway를 사용하는 것이 일반적

### Netflix Zuul 구현

1. FirstService, SecondService 생성
    - Spring Boot: 2.3.8
    - 의존성: Lombok, Spring Web, Eureka Discovery Client
2. FirstService, SecondService 구성
    - Controller
        
        ```java
        @RestController
        @RequestMapping("/")
        public class FirstServiceController {
        
            @GetMapping("/welcome")
            public String welcome() {
                return "Welcome to the First service.";
            }
        }
        ```
        
    - application.yml
        
        ```yaml
        server:
          port: 8081
        
        spring:
          application:
            name: my-first-service
        
        eureka:
          client:
            fetch-registry: false
            register-with-eureka: false
        ```
        
3. Test
    - 127.0.0.1:8081/first-service/welcome
4. Zuul Service 생성
    - Spring Boot: 2.3.8
    - 의존성, Lombok, Spring WEb, Zuul
5. Zuul Service 구성
    - Application.java
        
        ```java
        @SpringBootApplication
        @EnableZuulProxy
        public class ZuulServiceApplication {
        
            public static void main(String[] args) {
                SpringApplication.run(ZuulServiceApplication.class, args);
            }
        
        }
        ```
        
    - application.yml
        
        ```yaml
        server:
          port: 8000
        
        spring:
          application:
            name: my-zuul-service
        
        zuul:
          routes:
            first-service:
              path: /first-service/**
              url: http://localhost:8081
            second-service:
              path: /second-service/**
              url: http://localhost:8082
        ```
        
        - `zuul.routes`
            - `server.port`로 `path`로요청이 들어올 경우 `url`로 포트포워딩
6. ZuulFilter
    - API Gateway의 장점인 사전 처리 사후처리를 위한 필터 클래스
    - 기본적인 로그 정보를 기록

**테스트**

![Untitled 1](https://user-images.githubusercontent.com/72686708/141927222-cf8b5291-b340-4af2-84e4-c718ca760513.png)

- Zuul의 API Gateway 서비스를 통해 정상적으로 First Service가 호출된 것을 확인 가능하다.

## Netflix Zuul - Filter 적용

```java
@Component @Slf4j
public class ZuulLoggingFilter extends ZuulFilter {

    @Override
    public Object run() throws ZuulException {
        log.info("****************** printing log");

        RequestContext currentContext = RequestContext.getCurrentContext();
        HttpServletRequest request = currentContext.getRequest();

        log.info("****************** {}", request.getRequestURI());
        return null;
    }
    @Override
    public String filterType() {
        // 사전 필터로 설정
        return "pre";
    }

    @Override
    public int filterOrder() {
        return 1;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

}
```

```
c.e.z.filter.ZuulLoggingFilter           : ****************** printing log
c.e.z.filter.ZuulLoggingFilter           : ****************** /first-service/welcome
```
