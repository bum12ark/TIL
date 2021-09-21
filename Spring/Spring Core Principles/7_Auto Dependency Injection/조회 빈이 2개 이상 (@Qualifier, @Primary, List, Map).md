## 조회 빈이 2개 이상 - 문제

- @Autowired는 타입으로 조회한다.
- 타입으로 조회하기 때문에 인터페이스의 구현체 컴포넌트가 2개 이상일 때 문제가 발생한다.
- `DiscountPolicy` 인터페이스를 구현한 `FixDiscountPolicy`, `RateDiscountPolicy`를 둘다 스프링빈으로 선언할 시 오류가 발생한다.

```java
@Component
public class FixDiscountPolicy implements DiscountPolicy {}
```

```java
@Component
public class RateDiscountPolicy implements DiscountPolicy {}
```

- 의존관계 설정 시 인터페이스인 `DiscountPolicy`로 주입 받을 시

```java
@Autowired
private DiscountPolicy discountPolicy;
```

- `NoUniqueBeanDefinitionException` 오류가 발생한다.

```java
org.springframework.beans.factory.NoUniqueBeanDefinitionException: 
No qualifying bean of type 'hello.core.discount.DiscountPolicy' available: 
expected single matching bean but found 2: fixDiscountPolicy,rateDiscountPolicy
```

- 하위타입으로 지정하여 해결할 수 있겠지만 DIP를 위배하고 유연성이 떨어진다.
- 또한 이름만 다르고, 완전히 똑같은 타입의 스프링 빈이 2개 있을 경우에도 해결 되지 않는다.

## @Autowired 필드 명, @Qualifier, @Primary

- 조회 대상 빈이 2개 이상일 때의 해결 방법
    - @Autowired 필드 명 매칭
    - @Qualifier → @Qualifier 끼리 매칭 → 빈 이름 매칭
    - @Primary 사용

### @Autowired 필드명 매칭

- @Autowired는 타입 매핑을 시도하고, 여러 빈이 있을 경우, 파라미터 이름으로 빈 이름을 추가 매칭한다.

**기존코드**

```java
@Autowired
private DiscountPolicy discountPolicy;
```

**변경코드**

```java
// 1. 필드 명으로 빈 이름 매칭
@Autowired
private DiscountPolicy rateDiscountPolicy;

@Component
public class OrderServiceImpl implements OrderService {

    private final DiscountPolicy discountPolicy;
		
		// 파라미터 명으로 빈 이름 매칭
    public OrderServiceImpl(DiscountPolicy rateDiscountPolicy) {
        this.discountPolicy = rateDiscountPolicy;
    }
```

- `**@Autowired` 매칭 순서**
    1. 타입 매칭
    2. 1의 결과가 2개 이상일 경우 필드 명, 파라미터 명으로 빈 이름 매칭

### @Qualifier

- @Qualifier는 추가 구분자를 붙여주는 방법이다
- 주의) 빈 이름을 변경하는 것이 아니다

**빈 등록 시 @Qualifier를 붙여준다.**

```java
@Component
@Qualifier("fixDiscountPolicy")
public class FixDiscountPolicy implements DiscountPolicy {}
```

```java
@Component
@Qualifier("mainDiscountPolicy")
public class RateDiscountPolicy implements DiscountPolicy {
```

**주입 시에 @Qualifier를 붙여주고 등록한 이름을 적어준다.**

```java
@Component
public class OrderServiceImpl implements OrderService {

    private final DiscountPolicy discountPolicy;

    public OrderServiceImpl(@Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
    }
}
```

- `mainDiscountPolicy`로 등록한 `rateDiscountPolicy`가 주입되는 것을 확인할 수 있다.
- 생성자 자동주입, 필드 자동주입, setter 자동주입 등 모든 자동주입 방식에서 사용 가능하다.
- `@Qualifier`를 주입할때 `@Qualifier("mainDiscountPolicy")`를 찾지 못한다면 mainDiscountPolicy라는 이름의 스프링 빈을 추가로 찾는다.
    - 하지만 @Qualifier는 @Qualifier를 찾는 용도로만 사용하는 것이 명확하고 좋다.

**직접 Bean으로 등록 시에도 @Qualifier를 동일하게 사용 가능**

```java
@Bean
@Qualifier("mainDiscountPolicy")
public DiscountPolicy discountPolicy() {
		return new ...
}
```

- `@Qualifier` 매칭 순서
    1. @Qualifier끼리 매칭
    2. 빈 이름 매칭
    3. `NoSuchBeanDefinitionException` 예외 발생
- **단점**
    - 빈 등록 시와 주입 받을 시 모든 코드에 @Qualifier를 붙여줘야한다.

### @Primary

- 자동주입의 우선순위를 정하는 방법
- 여러 빈이 매칭될 경우 `@Primary`가 붙어있는 빈이 우선권을 가진다.

**우선순위를 가질 빈에 `@Primary`를 붙여준다.**

```java
@Component
@Primary
public class RateDiscountPolicy implements DiscountPolicy {}
```

**사용 코드**

```java
@Component
public class OrderServiceImpl implements OrderService {

    private final DiscountPolicy discountPolicy;

    public OrderServiceImpl(DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
    }
}
```

### @Primary, @Qualifier 활용

- 자주 사용하는 메인 데이터베이스 커넥션을 획득하는 빈이 있고 장애가 발생 시 사용하는 서브 데이터베이스 커넥션을 획등하는 빈이 있다고 가정해보자.
- 메인 커넥션을 담당하는 빈은 `@Primary`를 적용해서 조회하는 곳에 `@Qualifier` 없이 편리하게 조회 하고
- 서브 커넥션을 담당하는 빈을 획들할 때에는 `@Qualifier`를 지정하여 명시적으로 획득하는 방식으로 사용하면 코드를 깔끔하게 유지 할 수 있다.

### 우선순위

- 스프링은 자동보다는 수동, 넓은 범위의 선택권 보다는 좁은 범위의 선택권이 우선순위가 높다
- `@Primary`와 `@Qualifier` 중 `@Qualifier`가 우선권이 높다.

## 애노테이션 활용 (@Qualifier)

- @Qualifier는 문자열이기 때문에 컴파일 환경에서 오류를 잡아낼 수 없다.(오타!)
- 커스텀 애노테이션을 만들어 문제를 해결해보자

```java
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@Qualifier("mainDiscountPolicy")
public @interface MainDiscountPolicy {}
```

- @Qualifier의 애노테이션들을 다 가져온 뒤 @Qualifier("이름")을 작성한다.

```java
@Component
@MainDiscountPolicy
public class RateDiscountPolicy implements DiscountPolicy {}
```

```java
@Component
public class OrderServiceImpl implements OrderService {

    private final DiscountPolicy discountPolicy;

    public OrderServiceImpl(@MainDiscountPolicy DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
    }
}
```

- 코드를 추적하는데 장점이 있다.
- 뚜렷한 목적 없이 무분별하게 재정의 하는 것은 유지보수에 더 혼란만 가중할 수 있다.

## 조회한 빈이 모두 필요할때 (List, Map)

- 해당 타입의 빈이 모두 필요한 경우가 있다.
- 예를 들어 할인 서비스를 제공하는 클라이언트가 할인의 종류(rate, fix, ...)를 선택할 수 있는 경우가 있을 수 있다.

```java
@Test
void findAllBean() {
    ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class, DiscountService.class);

    DiscountService discountService = ac.getBean(DiscountService.class);

    Member member = new Member(1L, "userA", Grade.VIP);
    int fixDiscountPrice = discountService.discount(member, 10000, "fixDiscountPolicy");
    int rateDiscountPrice = discountService.discount(member, 10000, "rateDiscountPolicy");

    assertThat(discountService).isInstanceOf(DiscountService.class);
    assertThat(fixDiscountPrice).isEqualTo(1000); // 고정 정책의 경우 1000원 할인
    assertThat(rateDiscountPrice).isEqualTo(1000);
}

static class DiscountService {
    private final Map<String, DiscountPolicy> discountPolicyMap; // key: 빈이름, 객체
    private final List<DiscountPolicy> discountPolicies;

    public DiscountService(Map<String, DiscountPolicy> discountPolicyMap, List<DiscountPolicy> discountPolicies) {
        this.discountPolicyMap = discountPolicyMap;
        this.discountPolicies = discountPolicies;

        System.out.println("discountPolicyMap = " + discountPolicyMap);
        System.out.println("discountPolicies = " + discountPolicies);
    }

    // 할인 코드를 빈 이름과 매칭 (다형성 코드 완성)
    public int discount(Member member, int price, String discountCode) {
        DiscountPolicy discountPolicy = discountPolicyMap.get(discountCode);
        return discountPolicy.discount(member, price);
    }
}
```

- `Map<String, DiscountPolicy>`: map의 키에 스프링 빈의 이름을 넣어주고, 그 값으로 DiscountPolicy 타입으로 조회한 모든 스프링 빈을 담아준다.
- `List<DiscountPolicy>`: DiscountPolicy 타입으로 조회한 모든 스프링 빈을 담아준다.

## List, Map 다형성 활용 주입 방법

- DiscountService의 Map<String, DiscountPolicy>에 어느 빈들이 주입될 지, 빈들의 이름은 무엇일지 코드만 보고 한번에 파악하기 힘들다.
- 자동 등록을 사용하고 있기 때문에 파악하려면 여러 코드를 찾아봐야한다.
- 이런 경우 수동 빈으로 등록하거나 또는 자동으로 하면 특정 패키지에 같이 묶어두는게 좋다!

```java
@Configuration
public class DiscountPolicyConfig {
		
		@Bean
		public DiscountPolicy rateDiscountPolicy() {
				return new RateDiscountPolicy();
		}
		@Bean
		public DiscountPolicy fixDiscountPolicy() {
				return new FixDiscountPolicy();
		}
}
```

- 설정 정보만 봐도 빈 이름은 물론이고, 어떤 빈들이 주입될지 파악 가능하다!

# 📄 Reference

- [[인프런] 스프링 핵심 원리 - 기본편 (김영한)](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8/dashboard)