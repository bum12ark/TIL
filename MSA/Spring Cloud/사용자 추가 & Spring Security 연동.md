## 사용자 추가 (회원 가입)

### Validation 사용

- `spring-boot-starter-validation` 추가
    
    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    ```
    

### RequestUser Class

```java
@Data
public class RequestUser {
    @NotNull(message = "Email can't be null")
    @Size(min = 2, message = "Email not be less than 2 characters")
    @Email
    private String email;
    @NotNull(message = "Name can't be null")
    @Size(min = 2, message = "Name not be less than 2 characters")
    private String name;
    @NotNull(message = "Password can't be null")
    @Size(min = 8, message = "Password not be less than 8 characters")
    private String pwd;
}
```

- `javax.validation.constraints` 의 어노테이션을 사용하여 Validation 추가

### UserController class

```java
@PostMapping("/users")
public ResponseEntity createUser(@RequestBody @Valid RequestUser requestUser) {
    log.info("UserController.createUser");

    User user = userService.createUser(requestUser);
    ResponseUser responseUser = new ResponseUser(user.getEmail(), user.getName(), user.getUserId());

    return ResponseEntity.status(HttpStatus.CREATED).body(responseUser);
}
```

- @Valid 어노테이션을 통해 파라미터 검증처리를 진행
- 컨트롤러 → 서비스 : DTO
- 서비스 → 컨트롤러 : Entity

### User class

- 엔티티의 역할을 하는 클래스
- 기본적으로 Setter는 사용하지 않는다.
- 객체의 일관성 보장을 위하여 생성자에 Builder 패턴을 사용
- 위의 이유로 인하여 ModelMapper 또한 사용하지 않음

```java
@Entity
@Table(name = "users")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class User {

    @Id @GeneratedValue
    private Long id;
    @Column(nullable = false, length = 50)
    private String email;
    @Column(nullable = false, length = 50)
    private String name;
    @Column(nullable = false, unique = true)
    private String userId;
    @Column(nullable = false, unique = true)
    private String encryptedPwd;

    @Builder
    public User(String email, String name, String userId, String encryptedPwd) {
        this.email = email;
        this.name = name;
        this.userId = userId;
        this.encryptedPwd = encryptedPwd;
    }
}
```

### UserServiceImpl

```java
@Service
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository;

    @Override
    public User createUser(RequestUser requestUser) {
        String userId = UUID.randomUUID().toString();

        User user = User.builder()
                .userId(userId)
                .email(requestUser.getEmail())
                .name(requestUser.getName())
                .encryptedPwd(requestUser.getPwd())
                .build();

        return userRepository.save(user);
    }
}
```

- Builder 패턴을 통해 객체 생성

### UserRepository

```java
public interface UserRepository extends JpaRepository<User, Long> {
}
```

- JpaRepository를 상속 받아 사용

### 테스트

![Untitled](https://user-images.githubusercontent.com/72686708/142760939-7737a4c2-e3de-436e-86a5-4124bf215c03.png)

## Spring Security

- **Authentication + Authorization** (인증 + 인가)

### **적용절차**

1. 애플리케이션에 spring security 의존성 추가
2. `WebSecurityConfigurerAdapter`를 상속받는 `Security Configuration` 클래스 생성
3. Security Configuration 클래스에 `@EnableWebSecurity` 추가
4. Authentication → `configure(AuthenticationManagerBuilder auth)` 메소드 재정의
5. Password encode를 위한 `BCryptPasswordEncoder` 빈 정의
6. Authorization → `configure(HttpSecurity http)` 메소드 재정의

### 의존성 추가

- `spring-boot-starter-security` 의존성 추가

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### WebSecurity class

```java
@Configuration
@EnableWebSecurity
public class WebSecurity extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable();
        http.authorizeRequests().antMatchers("/users/**").permitAll();

        http.headers().frameOptions().disable();
    }
}
```

- `@EnableWebSecurity` : web security 용도로 사용하겠다는 어노테이션, 스프링 시큐리티를 사용하기 위해 추가
- `http.authorizeRequests()` : 시큐리티 처리에 HttpServletRequest를 사용하겠다는 의미
    - `antMatchers()` : 특정한 경로 지정
    - `permitAll()` : 모든 사용자가 접근 할 수 있다.
- `http.header().frameOptions().disable` : h2-console 접근을 위해 설정

### BCryptPasswordEncoder

- password를 해싱하기 위해 BCrypt 알고리즘 사용
- 랜덤 Salt를 부여하여 여러번 Hash를 적용한 암호화 방식

### UserServiceImpl class

- `passwordEncoder.encode(requestUser.getPwd())` 를 사용하여 비밀번호 해쉬

### BCryptConfig class

```java
@Configuration
public class BCryptConfig {

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

- `BCryptPasswordEncoder` 를 빈으로 등록하여 사용

### 테스트 클래스 작성

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @MockBean
    private UserService userService;

    @Test
    @DisplayName("회원가입 성공 테스트")
    void createUserSuccess() throws Exception {
        // Given
        RequestUser requestUser = new RequestUser();
        requestUser.setEmail("testId@gmail.com");
        requestUser.setName("홍길동");
        requestUser.setPwd("testPassword");

        User saveUser = User.builder()
                .userId(UUID.randomUUID().toString())
                .name(requestUser.getName())
                .encryptedPwd(UUID.randomUUID().toString())
                .build();

        given(userService.createUser(requestUser)).willReturn(saveUser);

        String requestJson = objectMapper.writeValueAsString(requestUser);

        // When & Then
        mockMvc.perform(
                post("/users")
                        .content(requestJson)
                        .contentType(MediaType.APPLICATION_JSON)
                        .accept(MediaType.APPLICATION_JSON)
        )
                .andExpect(status().isCreated())
                .andDo(print());
    }

    @Test
    @DisplayName("회원가입 잘못된 파라미터")
    void createUserBadParameter() throws Exception {
        // Given
        RequestUser requestUser = new RequestUser();
        requestUser.setEmail("not_email");
        requestUser.setName("두자");
        requestUser.setPwd("errorPw");

        String requestJson = objectMapper.writeValueAsString(requestUser);

        // When & Then
        mockMvc.perform(
                post("/users")
                        .content(requestJson)
                        .contentType(MediaType.APPLICATION_JSON)
                        .accept(MediaType.APPLICATION_JSON)
        )
                .andExpect(status().isBadRequest())
                .andDo(print());
    }
}
```
