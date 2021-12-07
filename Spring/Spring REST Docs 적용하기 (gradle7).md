### 환경

- SpringFramework 2.6.0
- Gradle 7.3
- Java 11

## build.gradle

### plugins

```java
plugins {
    id 'org.springframework.boot' version '2.6.0'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
    id 'org.asciidoctor.jvm.convert' version '3.3.2'
}
```

- `id 'org.asciidoctor.jvm.convert' version '3.3.2'` 추가
- REST Docs 파일인 asciidoc을 변환해주는 플러그인

### snippetsDir 설정

- ext는 전역 변수를 설정해주는 곳이다.
- gradle은 `build/generated-snippets`에 생성되며 maven은 `target/generated-snippets`에 생성된다.

```java
ext {
    set('springCloudVersion', "2021.0.0-RC1")
    snippetsDir = file('build/generated-snippets')
}
```

### 의존성 추가

- REST Docs는 테스트 코드를 통해 작성된다.
- 현재 테스트 코드를 mockMVC를 사용하여 작성하므로 mockmvc 의존성을 추가 하였다.
- asciidoctor의 operation block 을 사용하기 위해 `spring-restdocs-asciidoctor` 의존성을 직접 주입 시켜주었다.

```java
dependencies {
		// ...
    testImplementation 'org.springframework.restdocs:spring-restdocs-mockmvc'
    asciidoctorExtensions 'org.springframework.restdocs:spring-restdocs-asciidoctor'
}
```

### asciidoctor 추가

- org.asciidoctor.jvm.convert 을 사용할 경우 asciidoctor 구성이 기본적으로 만들어지지 않기 때문에 configurations를 사용하여 직접 구성하였다.

```java
asciidoctor {
    configurations 'asciidoctorExtensions'
    attributes 'snippets': snippetsDir
    dependsOn test
}
```

### bootJar 추가

- 스니펫을 이용해 문서 작성 후, build/docs/asciidoc 하위에 생기는 html 파일을 BOOT-INF/classes/static/docs로 복사해준다.

```java
bootJar {
    dependsOn asciidoctor
    copy {
        from "${asciidoctor.outputDir}"
        into 'src/main/resources/static/docs'
    }
}
```

## 테스트 코드 작성

- 테스트 코드에 따라 스니펫이 생성된다.
- @AutoConfigureRestDocs 어노테이션 사용

```java
@WebMvcTest(UserController.class)
@Import(TestConfig.class)
@AutoConfigureRestDocs(uriHost = "127.0.0.1", uriPort = 8000)
class UserControllerTest {
    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @MockBean
    private ModelMapper mockModelMapper;

    @MockBean
    private UserService userService;

		@Test
    @DisplayName("회원 삭제 - 존재하지 않는 회원")
    public void deleteUserByEmailNotExistUser() throws Exception {
        // Given
        String email = "notExistUser@gmail.com";
        willThrow(new NotExistUserException()).given(userService).deleteUserByEmail(email);

        // When
        ResultActions actions = mockMvc.perform(delete("/users/{email}", email));

        // Then
        actions.andExpect(status().isConflict())
                .andDo(print())
                .andDo(document("user-delete-no-such-user", pathParameters(
                        parameterWithName("email").description("유저 이메일")
                )));
    }
}
```

### API 문서 양식 `adoc` 만들기

```
== 회원
=== 회원 삭제 (존재하지 않는 회원)

operation::user-delete-no-such-user[snippets='curl-request,http-response']
```

- `./gradlew build` 명령어를 통해 스니핏 및 html 파일 생성

### 웹 페이지에서 확인

- `localhost:port/docs/api-docs.html`  으로 스니핏으로 만들어진 HTTP Docs를 확인할 수 있다.

![Untitled](https://user-images.githubusercontent.com/72686708/145045405-f907e327-c0fb-44d1-a041-a18a9b0af6e8.png)

## Trouble Shooting

### `'org.asciidoctor.convert' is deprecated.` 오류

- 스니핏 파일을 변환해주고 build 폴더 위치에 복사하기 위한 플러그인이, gradle 7.x 부터는 jvm 플러그인을 사용하지 않으면 위와 같은 오류가 발생한다.
- 버전 호환성 문제로써 `org.asciidoctor.jvm.convert` 를 사용해야한다.
- gradle 7 버전 이상 부터는 기존의 `org.asciidoctor.convert` 가 Deprecated 되어 사용 불가능 하다.

### pathVariable 이슈

- MockMvcRequestBuilder.* 의 get(), post() 를 사용하였을 때 urlTemplate 오류가 발생 하였다.
- pathParameters를 문서화 해주기 위해서는 RestDocumentationRequestBuilders.* 를 사용해야 한다.

## Reference

- [https://docs.spring.io/spring-restdocs/docs/2.0.5.RELEASE/reference/html5/#working-with-markdown](https://docs.spring.io/spring-restdocs/docs/2.0.5.RELEASE/reference/html5/#working-with-markdown)
- [https://github.com/team-simplelog/simplelog/issues/17](https://github.com/team-simplelog/simplelog/issues/17)
- [https://github.com/spring-projects/spring-restdocs/issues/680](https://github.com/spring-projects/spring-restdocs/issues/680)
- [https://velog.io/@max9106/Spring-Spring-rest-docs를-이용한-문서화](https://velog.io/@max9106/Spring-Spring-rest-docs%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EB%AC%B8%EC%84%9C%ED%99%94)
