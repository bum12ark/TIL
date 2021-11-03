## 설계 이유

- Spring REST + Spring HATEOS 관련 프로젝트를 진행하던 도중 일어난 이슈로 구현하게 되었다.
- 기존의 에러를 핸들링할 수 있는 방법은 `try ~ catch` 문과 `org.springframework.validation.Errors` 등 여러 방법이 존재한다.
- 하지만 위 두 방법은 개발하는 사람에 따라 예외처리 방식이 달라져 유지보수 측면에서 좋지 않을 것으로 생각되었다.
- 그리하여 전역에서 예외를 핸들링할 수 있는 `@ControllerAdvice` + `@ExceptionHandler`를 사용하는 전역예외처리 담당 클래스를 설계 및 구현하게 되었다.

## 개념

- ControllerAdvice와 ExceptionHandler
    - [@`ControllerAdvice`](https://www.notion.so/ControllerAdvice-4a4b15a3b2354958a4d873be11c81d71)
    - [@ExceptionHandler](https://www.notion.so/ExceptionHandler-7492922e50ff462c893d6c85488be1e7)

# 클래스 설계

---

### 목표

- API 통신간에 일어날 수 있는 예외 처리를 공통적으로 동일하게 처리할 수 있도록 설계
- 사용자 정의 익셉션 클래스들이 증가하더라도 `ControllerAdvice`에서 추가적인 코드를 작성하지 않도록 설계
- 동일한 Exception에 관해서 개발자에 따라 다른 Response가 아닌 동일한 Response를 내보내도록 설계

### CustomException

- 사용자 개별 정의 Exception들의 상속을 받는  상위 클래스
- 서버 기동 시 일어날 수 있는 예외를 핸들링 하기 위해 `RuntimeException` 클래스를 상속받는다.
- 생성자를 protected로 설정하여 외부 패키지에서 생성할 수 없도록 설계
- **에러**에 관한 정보를 담는 `Enum` 클래스와 **Spring HATEOS**에 필요한 `Link` 클래스를 담고있다.

```java
@Getter
public class CustomException extends RuntimeException {
    private ErrorEnum errorEnum;
    private Link link;

    protected CustomException(ErrorEnum errorEnum, Link link) {
        super(errorEnum.getMessage());

        this.errorEnum = errorEnum;
        this.link = link;
    }
}
```

### xxxException

- 사용자가 개별적으로 정의하는 **`사용자 정의 Exception class`**
- CustomException을 상속받아야한다. (글로벌 예외처리의 코드 복잡성을 줄이기 위해)
- 생성자를 통해 ErrorEnum과 Link 정보를 CustomException으로 전달한다.
- ErrorEnum은 **생성자를 통해 전달받지 않으며** 
해당 익셉션에 해당하는 ErrorEnum을 **클래스 내부에서 전달**한다.

```java
public class NotFoundException extends CustomException {

    public NotFoundException(Link link) {
        super(ErrorEnum.NOT_FOUND, link);
    }
}
```

### ErrorEnum

- Http 상태코드와 메시지를 가지고 있는 Enum 클래스이다.
- 동일한 종류의 예외에서 동일한 상태코드와 메시지를 전달하기 위해서 설계되었다.
- `@JsonFormat(shape = JsonFormat.Shape.OBJECT)`
    - Enum을 JSON 형식으로 변환하기 위해 사용되는 어노테이션 속성

```java
@Getter @AllArgsConstructor
@JsonFormat(shape = JsonFormat.Shape.OBJECT) 
public enum ErrorEnum {
    INVALID_PARAMETER(HttpStatus.BAD_REQUEST, "Invalid Request Data"),
    NOT_FOUND(HttpStatus.NOT_FOUND, "Not Found Data"),
    UNAUTHORIZED(HttpStatus.UNAUTHORIZED, "Unauthorized User");

    private final HttpStatus httpStatus;
    private final String message;
}
```

### GlobalExceptionHandler

- API 관련 글로벌 예외를 핸들링하기 위해 `@RestControllerAdvice` 어노테이션 사용
- `@ExceptionHandler(CustomException.class)`
    - ExceptionHandler의 대상으로 CustomException 클래스를 설정한다.
    - 사용자 지정 Exception들은 모두 CustomException을 상속하기 때문에
    모든 사용자 지정 Exception들을 한곳에서 핸들링할 수 있다.
- `Spring HATEOS`의 `EntityModel`와 `ResponseEntity`를 사용하여 `REST`한 API를 생성한다.

```java
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(CustomException.class)
    public ResponseEntity customExceptionHandler(CustomException ce) {
        log.error("#################################################");
        log.error("## ErrorEnum: " + ce.getErrorEnum());
        log.error("## Link: " + ce.getLink());
        log.error("#################################################");
				
				// Spring HATEOS
        EntityModel<ErrorEnum> model = EntityModel.of(ce.getErrorEnum())
                .add(ce.getLink());

        return ResponseEntity
                .status(ce.getErrorEnum().getHttpStatus())
                .body(model);
    }
}
```

# 테스트 예제

---

- Service
    
    ```java
    @Service
    @RequiredArgsConstructor
    @Transactional(readOnly = true)
    public class ExService {
    
        private final ExRepository exRepository;
    		
    		public Example findById(Long exId) {
            Example example = exRepository.findById(exId)
                    .orElseThrow(() -> new NotFoundException(
                            linkTo(methodOn(ExRestController.class).getExampleOne(exId))
    																.withSelfRel())
                    );
            return example;
        }
    }
    ```
    
- 테스트 (JUnit5)
    
    ```java
    @Test
    public void 예제_단일건수조회_NotFound() throws Exception {
        // GIVEN
        String url = "/api/ex/100";
    
        // WHEN
        ResultActions actions = mockMvc.perform(get(url));
    
        // THEN
        actions.andDo(print())
                .andExpect(status().isNotFound())
                .andExpect(jsonPath("httpStatus").value(is(ErrorEnum.NOT_FOUND.toString())))
                .andExpect(jsonPath("message").exists())
                .andExpect(jsonPath("_links.self").exists())
        ;
    }
    ```
    
- MockHttpServletResponse
    
    ```java
    MockHttpServletResponse:
               Status = 404
        Error message = null
              Headers = [Content-Type:"application/hal+json;charset=UTF-8"]
         Content type = application/hal+json;charset=UTF-8
                 Body = {
    											"httpStatus":"NOT_FOUND",
    											"message":"Not Found Data",
    											"_links":{
    																"self":{"href":"http://localhost/api/ex/100"}
    																}
    										}
        Forwarded URL = null
       Redirected URL = null
              Cookies = []
    ```
    

# Reference

---

- [https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-exceptionhandler](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-exceptionhandler)
- [https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/ExceptionHandler.html](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/ExceptionHandler.html)
- [https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-exceptionhandler](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-exceptionhandler)
- [https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/ExceptionHandler.html](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/ExceptionHandler.html)