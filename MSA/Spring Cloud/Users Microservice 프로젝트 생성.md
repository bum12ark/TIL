## 개요

![Untitled](https://user-images.githubusercontent.com/72686708/142723757-8e9cb10d-9f56-45d4-b8f9-625b7907139a.png)


- Front-end: X
- Business Logic
    - Spring Boot
    - Spring Framework
- Database
    - JPA
    - H2
- Features
    - 신규 회원 등록
    - 회원 로그인
    - 상세 정보 확인
    - 회원 정보 수정/삭제
    - 상품 주문
    - 주문 내역 확인
- APIs
    
    ![Untitled 1](https://user-images.githubusercontent.com/72686708/142723759-9e260b38-e5f0-40d8-8582-6430cade5254.png)
    

### 프로젝트 생성

- **의존성**
    - DevTools, Lombok, Web, **Eureka Discovery Client**
- **Application Class**
    - `@EnableDiscoveryClient` 추가
        - 마이크로 서비스를 유레카 서버에 등록
- **application 파일**
    
    ```yaml
    eureka:
      client:
        register-with-eureka: true
        fetch-registry: true
        service-url:
          defaultZone: http://127.0.0.1:8786/eureka
      instance:
        instance-id: ${spring.application.name}:${spring.application.instance_id:${random.value}}
    ```
    
- **RestController Class**
    
    ```java
    @RestController
    @RequestMapping("/")
    @Slf4j @RequiredArgsConstructor
    public class UserController {
    
        @GetMapping("/health_check")
        public String status() {
            return "It's Working in User Service";
        }
    }
    ```
    
    - 상태 체크 (health_check)
- **Configuration 정보 추가**
    - application 파일에 Welcome message 추가
        
        ```yaml
        greeting:
          message: Welcome to the Simple E-commerce.
        ```
        
    - UseresController Class
        - Environment 사용
            
            ```java
            @RestController
            @RequestMapping("/")
            @Slf4j @RequiredArgsConstructor
            public class UserController {
            
                private final Environment environment;
            
                @GetMapping("/welcome")
                public String welcome() {
                    return environment.getProperty("greeting.message");
                }
            }
            ```
            
        - Value 사용
            
            ```java
            @Component @Data
            public class Greeting {
                @Value("${greeting.message}")
                String message;
            }
            ```
            
            ```java
            @RestController
            @RequestMapping("/")
            @Slf4j @RequiredArgsConstructor
            public class UserController {
                private final Greeting greeting;
            
                @GetMapping("/welcome")
                public String welcome() {
                    return greeting.getMessage;
                }
            
            }
            ```
            
- **H2 Database**
    - 자바로 작성된 오픈소스 RDBMS
    - Embedded, Server-Client 가능
    - JPA 연동 가능, 의존성 추가 및 scope runtime으로 설정
    - application 파일 h2 설정 추가
        
        ```yaml
        spring:
          application:
            name: user-service
          h2:
            console:
              enabled: true
              settings:
                web-allow-others: true
              path: /h2-console
          datasource:
            driver-class-name: org.h2.Driver
            url: jdbc:h2:~/msa-user-service
        ```
