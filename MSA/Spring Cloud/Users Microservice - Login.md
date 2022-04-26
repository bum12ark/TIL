## AuthenticationFilter 추가

### WebSecurityConfigurerAdapter

- 시프링 시큐리티의 웹 보안 기능 초기화 및 설정
- HttpSecurity 라는 세부적인 보안 기능을 설정할 수 있는 API를 제공하는 클래스를 생성
- 해당 객체를 상속받고 @EnableWebSecurity 어노테이션을 작성할 경우 SpringSecurityFilterChain이 자동으로 포함된다.
- `configure` 메소드를 오버라이딩 하여 접근 권한을 작성 할 수 있다.
- 예제 소스
    
    ```java
    @Configuration
    @EnableWebSecurity
    @RequiredArgsConstructor
    public class WebSecurity extends WebSecurityConfigurerAdapter {
    		@Override
        protected void configure(HttpSecurity http) throws Exception {
            http.csrf().disable();
    
            http.authorizeRequests().antMatchers("/actuator/**").permitAll();
    
            http.headers().frameOptions().disable();
        }
    }
    ```
    
    - `antMatchers()` : url 패턴을 지정

### UsernamePassordAuthenticationFilter

![1](https://user-images.githubusercontent.com/72686708/165282278-de727b25-7cee-47c6-88e6-4356b495dead.png)

- 아이디, 패스워드 기반의 인증을 담당하는 필터
- AuthenticationManager를 통한 인증 실행
- 인증 성공 시, 얻은 Authentication 객체를 SecurityContext에 저장 후 AuthenticationSuccessHandler 실행
- 인증 실패 시, AuthenticationFailureHandler 실행
- URL 기본 값: `/login`
- **attemptAuthentication(req, res)**
    
    ```java
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request,
                                                HttpServletResponse response) throws AuthenticationException {
    
        Authentication authenticate = null;
        try {
            RequestLogin credential = new ObjectMapper().readValue(request.getInputStream(), RequestLogin.class);
    
            authenticate = getAuthenticationManager().authenticate(
                    new UsernamePasswordAuthenticationToken(
                            credential.getEmail(), credential.getPassword(), new ArrayList<>()
                    )
            );
    
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    
        return authenticate;
    }
    ```
    
    - `/login` url로 요청이 들어올 경우 실행
    - request 객체의
- `successfulAuthentication`
    
    ```java
    @Override
    protected void successfulAuthentication(HttpServletRequest request,
                                            HttpServletResponse response,
                                            FilterChain chain,
                                            Authentication authResult) throws IOException, ServletException {
        // 로그인 성공 후 로직 처리
    }
    ```
    

## loadUserByUsername() 구현

```java
@Override
public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
    User findUser = userRepository.findByEmail(email)
            .orElseThrow(() -> new UsernameNotFoundException(email));

    return new org.springframework.security.core.userdetails.User(
            findUser.getEmail(), findUser.getEncryptedPwd(),
            true, true, true, true,
            new ArrayList<>()
    );
}
```

- 로그인을 시도할 시 해당 하는 유저 정보를 가져오는 메소드
- UserDetailService를 implement하였을 때 Override 해야하는 메소드

## Routes 정보 변경

```yaml
routes:
  - id: user-service
    uri: lb://USER-SERVICE
    predicates:
      - Path=/user-service/login
      - Method = POST
    filters:
      - RemoveRequestHeader=Cookie
      - RewritePath=/user-service/(?<segment>.*), /$\{segment}
```

- Method : Http Method 값 설정
- RemoveRequestHeader : name 을 파라미터로 받으며 파라미터로 받은 name 을 헤더에서 제거한다.
- RewritePath : regexp 와 replacement 파리미터를 받는다. 유연한 rewrite path 를 위하여 자바 정규표현식을 사용한다.

### 테스트

- 유저 마이크로 서비스: GET /users
- API Gateway: GET /user-service/users
- RewritePath를 통해 불필요한 uri 였던 user-service를 제거 가능

## 로그인 처리 과정

`login` 파라미터 매핑이 없을 경우 스프링 시큐리티에서 자동으로 매핑하여 작업을 진행

1. WebSecurity 클래스
    - Configuration 관련
2. AuthenticationFilter 클래스
    - login Form 인증과 허가와 관련된 필터
3. loadUserByUsername 메소드
    - 레파지토리에서 패스워드 가져오기

## 로그인 성공 처리

- 로그인 성공 후 토큰을 만드는 처리를 진행

> loadUserByUsername에서 user_id를 저장할 수 있도록 User 클래스를 확장하시고, successfulAuthentication 메소드에서 사용하실 수도 있습니다. 현재 예제에서는 H2 메모리DB를 사용하고 있고, 속도 및 비용이 그렇게 크지 않아서, Spring Security의 기본 UserDeails를 구현한 User 클래스를 최대한 사용하려고 했다.
>
