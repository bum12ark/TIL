## Spring Gateway 연동

- 기존의 해당 인스턴스를 바로 부르는 방식에서 API Gateway를 연동함으로써 API Gateway를 통하여 접속한다.
- 인스턴스의 포트의 기본값이 랜덤 방식이었기 때문에 해당 인스턴스를 테스트할 때마다 포트 번호를 매번 확인 해야하는 불편함이 존재
- 그러나 API Gateway와 연동함으로써 API Gateway를 통하여 접근하므로써 포트 번호를 확인할 필요가 없다.
    - API Gateway에서 유레카 서버를 통하여 마이크로 서비스의 위치 정보를 가져오기 때문

### API Gateway 라우트 정보 등록

```yaml
spring:
  application:
    name: apigateway-service
  cloud:
    routes:
        - id: user-service
          uri: lb://USER-SERVICE
          predicates:
            - Path=/user-service/**
```

- 테스트 해본 결과 유저 마이크로 서비스로 직접 health_check를 호출하였을 때는 이상 없음
- 그러나 API Gateway를 통하여 접속할 경우 404 에러 발생
- 이유: User Service의 URI와 API Gateway의 URI가 다름

### 이유: User Service의 URI와 API Gateway의 URI가 다름

- 호출되는 uri는 `/user-service/health_check`
- 유저 서비스의 실제 uri 매핑 정보는 `/health_check`
- 두 uri 정보가 다르기 때문에 RequestMapping이 되지 않는 문제

**해결법**

- Root Request Mapping 정보 변경
    
    ```java
    @RestController
    @RequestMapping("/user-service")
    @Slf4j @RequiredArgsConstructor
    public class UserController {}
    ```
    
- Api Gateway의 Path 패턴 변경 (이후 포스팅에서 알아보자)

## 사용자 조회 구현

### UserServiceImpl class

```java
@Override
public User getUserByUserId(String userId) {
    return userRepository.findByUserId(userId).orElseThrow(NullPointerException::new);
}

@Override
public Iterable<User> getUserByAll() {
    return userRepository.findAll();
}
```

- `orElseThrow` : DB의 값이 없을 경우 NPE 발생
    - 추후에 RestControllerAdvice + ExceptionHandler 방식으로 변경

### UserController class

```java
@GetMapping("/users")
public ResponseEntity getUsers() {
    Iterable<User> findUsers = userService.getUserByAll();

    List<ResponseUser> results = new ArrayList<>();
    findUsers.forEach(user -> results.add(new ResponseUser(user.getEmail(), user.getName(), user.getUserId(), new ArrayList<>())));

    return ResponseEntity.status(HttpStatus.OK).body(results);
}

@GetMapping("/users/{userId}")
public ResponseEntity getUser(@PathVariable("userId") String userId) {

    User findUser = userService.getUserByUserId(userId);
    ResponseUser responseUser = new ResponseUser(findUser.getEmail(), findUser.getName(), findUser.getUserId());

    return ResponseEntity.status(HttpStatus.OK).body(responseUser);
}
```

### 유닛 테스트

```java
@Test
@DisplayName("모든 유저 조회")
void getUsers() throws Exception {
    // Given
    // When & Then
    mockMvc.perform(get("/user-service/users"))
            .andExpect(status().isOk())
            .andDo(print());
}

@Test
@DisplayName("특정 유저 조회")
void getUser() throws Exception {
    // Given
    String userId = "testId";

    given(userService.getUserByUserId(userId)).willReturn(User.builder().userId("testId").build());

    // When & Then
    mockMvc.perform(get("/user-service/users/" + userId))
            .andExpect(status().isOk())
            .andExpect(jsonPath("userId").value("testId"))
            .andDo(print());
}
```

- `jsonPath()` : response body의 json 값을 검증하는데 사용

### Postman 통합 테스트

![Untitled](https://user-images.githubusercontent.com/72686708/142895413-c14d3b21-5ca3-4778-b5cb-8dba0e427d36.png)
