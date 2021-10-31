> Spring Web MVC에서 제공하며,
**글로벌** **예외** 처리를 위해 사용되는 어노테이션
> 
- 일반적으로 `@ExceptionHandler`, `@InitBinder` 및 `@ModelAttribute` 메서드는 선언된 `@Controller` 클래스(또는 클래스 계층 구조) 내에 적용된다.
- `@ControllerAdbvice`에는 `@Component` 어노테이션이 달려 있으며, 이는 이러한 클래스를 Component Scan을 통해 Spring Beans로 등록할 수 있음을 의미한다.
- `@RestControllerAdvice`는 `@ControllerAdvice`와 `@ResponseBody` 모두에 주석이 달린 합성 주석으로, 기본적으로 `@ExceptionHandler` 메소드가 메시지 변환을 통해 응답 본문에 랜더링 된다.

### 범위 설정법

- `@ControllerAdvice`는 모든 요청(즉, 모든 컨트롤러)에 적용되지만 아래 예제와 같이 어노테이션 속성을 사용하여 컨트롤러 **하위 집합으로 범위를 좁힐 수 있다.**

```java
// Target all Controllers annotated with @RestController
@ControllerAdvice(annotations = RestController.class)
public class ExampleAdvice1_1 {}
@RestControllerAdvice
public class ExampleAdvice1_2 {}

// Target all Controllers within specific packages
@ControllerAdvice("org.example.controllers")
public class ExampleAdvice2 {}

// Target all Controllers assignable to specific classes
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})
public class ExampleAdvice3 {}
```

# Reference

---

- [https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-controller-advice](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-controller-advice)