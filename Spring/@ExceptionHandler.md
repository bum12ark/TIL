> 특정 핸들러 클래스 및 메소드에서 예외를 처리하기 위한 어노테이션
> 
- `@Controller` 및 `@ControllerAdvice` 클래스는 아래 예제와 같이 `@ExceptionHandler`를 사용하여 컨트롤러 메소드의 예외를 처리하는 메소드를 가질 수 있다.
    
    ```java
    @Controller
    public class SimpleController {
        // ...
        @ExceptionHandler
        public ResponseEntity<String> handle(IOException ex) {
            // ...
        }
    }
    ```
    
- 일치하는 예외 유형의 경우 아래 예제와 같이 대상 예외를 메소드 인수로 선언하는 것이 좋다.
    - 여러 예외 메소드가 일치하는 경우 일반적으로 원인 예외 일치보다는 루트 예외 일치가 선호된다.
    
    ```java
    // FileSystemException.class, RemoteException.class: 원인 예외
    @ExceptionHandler({FileSystemException.class, RemoteException.class})
    public ResponseEntity<String> handle(IOException ex) {
    		// IOException: Root 예외
        // ...
    }
    ```
    
- 스프링 MVC에서 `@ExceptionHandler`에 대한 지원은 DispatcherServlet 수준, HandlerExceptionResolver 메커니즘을 기반으로 빌드된다.

### 메소드 인자

- Exception type
- HandlerMethod
- WebRequest, NativeWebRequest
- javax.servlet.ServletRequest, javax.servlet.ServletResponse
- javax.servlet.http.HttpSession
- ... (더 많은 정보는 Reference의 첫번 째 document를 참조)

### 반환값

- @ResponseBody
- HttpEntity<B>, ResponseEntity<B>
- String
- View
- java.util.Map, org.springframework.ui.Model
- @ModelAttribute
- ModelAndView object
- void

# Reference

---

- [https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-exceptionhandler](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-exceptionhandler)
- [https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/ExceptionHandler.html](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/ExceptionHandler.html)