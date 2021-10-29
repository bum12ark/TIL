### 소개

- SpringBoot는 테스트 목적에 따라 다양한 어노테이션을 제공한다.
- `spring-boot-starter-test` 를 사용하여 의존성을 전이적으로 가져올 수 있따.

### 테스트 종류

- 통합 테스트
    - @SpringBootTest
- 단위 테스트
    - @WebMvcTest
    - @DataJpaTest
    - @RestClientTest
    - @JsonTest

> Tip!
JUnit4를 사용하는 경우 @RunWith(SpringRunner.class)를 테스트 위에 추가해야한다. 
JUnit5를 사용하는 경우 해당 어노테이션은 명시할 필요없다. @Extendwith(SpringExtension.class)와 같은 동일한 역할을 하는 어노테이션이 @SpringBootTest와 같은 어노테이션에 포함되어 있기 때문이다.
> 

## @SpringBootTest

- 통합테스트를 제공하는 기본적인 스프링 부트 테스트 어노테이션
    - 실제 운영 환경에서 사용될 클래스들을 통합하여 테스트한다.
    - 단위 테스트와 같이 기능 검증을 위한 것이 아니라 spring framework에서 전체적으로 플로우가 제대로 동작하는 지 확인하기 위해 사용
- @SpringBootTest는 기본적으로 서버를 시작하지 않는다.
- 장점
    - 애플리케이션의 설정, 모든 Bean을 모두 등록하기 때문에 운영환경과 가장 유사한 테스트 가능
- 단점
    - 애플리케이션의 설정, 모든 Bean을 모두 등록하기 때문에 시간이 오래 걸림

### @SpringBootTest 속성 (webEnvironment)

- webEnvironment 속성을 사용하여 테스트 실행 방법을 선택할 수 있다.
    - **MOCK (기본값)**
        - `ApplicationContext`를 로드하고 가상 (mock) 웹 환경을 제공한다.
        - 이 속성을 사용할 경우 내장 서버는 시작되지 않는다.
        - 웹 응용 프로그램의 Mock 기반 테스트를 위해 `@AutoConfigureMockMvc` 또는 `@AutoConfigureWebTestClient`와 함께 사용 가능하다.
    - **RANDOM_PORT**
        - EmbeddedWebApplicationContext를 로드하여 실제 서블릿 환경을 구성한다.
    - **DEFINED_PORT**
        - RANDOM_PORT와 동일하게 실제 서블릿 환경을 구성한다.
        - 차이점으로는 application.properties에서 지정한 포트를 수신 대기 한다.
    - **NONE**
        - ApplicationContext를 로딩한다. 그러나 다른 웹 환경은 제공되지 않는다.
- properties
    1. 프로퍼티를 {key=value} 형식으로 직접 추가할 수 있다.
        
        ```java
        @SpringBootTest(
                properties = {
                        "testId=hello",
                        "testPrice=1"
                }
        )
        public class MyTest {
        
            @Value("${testId}")
            private String testId;
        
            @Value("${testPrice}")
            private String testPrice;
        
            @Test
            void propertiesTest() {
                assertThat(testId).isEqualTo("hello");
                assertThat(testPrice).isEqualTo("1");
            }
        }
        ```
        
    2. 기본적으로 클래스 경로 상의 application.properties를 통해 애플리케이션 설정을 수행한다.

### @AutoConfigureMockMvc과 MockMvc

- @SpringBootTest는 기본적으로 서버를 시작하지 않는다.
- Mock 테스트 시 필요한 의존성을 제공해준다.
- MockMvc 객체를 통해 실제 컨테이너가 실행되는 것은 아니지만 로직상으로 테스트를 진행할 수 있다.
- Mock 환경 내에서 테스트하는 것은 일반적으로 전체 서블릿 컨테이너로 시작하는것보다는 빠르다.
- Spring MVC 계층에서 Mocking이 발생하기 때문에 하위 수준의 Servlet 컨테이너에 의존하는 코드는 MockMvc로 직접 테스트할 수 없다.

```java
@SpringBootTest
@AutoConfigureMockMvc
class ExampleTest {
    
    @Autowired
    MockMvc mockMvc;
}
```

### TestRestTemplate

- webEnvironment 설정이 NONE이 아닌 경우 사용 가능
- webEnviornment에 맞추어 자동으로 설정되어 빈이 생성되며, RestTemplate의 테스트를 처기 가능하다.

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class TestRestTemplateTest {

    @Autowired
    TestRestTemplate template;
    
    @Test
    void testRequest() {
        HttpHeaders headers = this.template.getForEntity("/example", String.class)
                .getHeaders();
        assertThat(headers.getLocation()).hasHost("other.example.com");
    }
    
    @TestConfiguration(proxyBeanMethods = false)
    static class RestTemplateBuilderConfiguration {
        
        @Bean
        RestTemplateBuilder restTemplateBuilder() {
            return new RestTemplateBuilder().setConnectTimeout(Duration.ofSeconds(1))
                    .setReadTimeout(Duration.ofSeconds(1));
        }
    }
}
```

### @Transactional

- 스프링 부트 테스트에서 `@Transactional` 사용 시 기본적으로 각 테스트 메서드가 끝날 때 트랜잭션을 **롤백**
- 그러나 webEnvironment가 `RANDOM_PORT,` `DEFINED_PORT` 일 경우 롤백되지 않는다.
    - 위 두 속성은 실제 서블릿 환경을 제공하므로 별도의 스레드에서 실행된다.
    - 그러므로 별도의 트랜잭션에서 실행되기 때문

### @MockBean

- Mock 객체를 빈으로써 등록할 수 있다.
- ApplicationContext는 Mock 객체를 빈으로 등록하며, 같은 이름과 타입으로 빈이 등록되어 있다면 마지막에 선언한 빈으로 대체된다.

```java
@Service
class RemoteService {
	
	public String getHello() {
			return "hello";
	}
}
```

```java
@SpringBootTest
class MyTest {
		
		@MockBean
    private RemoteService service;

    @Test
    void exampleTest() {
        BDDMockito.given(service.getHello()).willReturn("hello");
        String hello = this.service.getHello();
        assertThat(hello).isEqualTo("hello");
    }
}
```

## Reference

- [스프링 공식 문서](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing.spring-boot-applications.mocking-beans)
- [갓대희님 블로그](https://goddaehee.tistory.com/211)