## Named 쿼리 - 정적 쿼리

- 미리 정의해서 이름을 부여해두고 사용하는 JPQL
- 정적 쿼리
- 어노테이션, XML에 정의
- 애플리케이션 로딩 시점에 초기화 후 재사용
- **애플리케이션 로딩 시점에 쿼리를 검증**
- 참고) Spring Data JPA 에서는 `@Query` 어노테이션을 통하여 인터페이스를 사용하여 사용하기 쉽게 제공한다.

### Named 쿼리 - 어노테이션

```java
@Entity
@NamedQuery(
        name = "Member.findByName",
        query = "SELECT m FROM Member m WHERE m.name = :name"
)
public class Member {
	...
}
```

```java
@Test
void namedQueryTest() {
    EntityManager entityManager = emf.createEntityManager();
    EntityTransaction transaction = entityManager.getTransaction();
    transaction.begin();

    try {
        List<Member> results = entityManager.createNamedQuery("Member.findByName", Member.class)
                .setParameter("name", "회원1")
                .getResultList();

        // 트랜잭션 커밋
        transaction.commit();
    } catch (Exception e) {
        e.printStackTrace();
        transaction.rollback();
    } finally {
        entityManager.close();
    }
}
```

## JPQL - 벌크 연산

### 벌크 연산

- 재고가 10개 미만인 모든 상품의 가격을 10% 상승하려면?
- JPA 변경 감지 기능으로 실행하려면 너무 많은 SQL 실행
    1. 재고가 10개 미만인 상품을 리스트로 조회한다.
    2. 상품 엔티티의 가격을 10% 증가한다.
    3. 트랜잭션 커밋 시점에 변경감지가 동작한다.
- 변경된 데이터가 100건이라면 100번의 UPDATE SQL 실행

### 벌크 연산 예제

- 쿼리 한 번으로 여러 테이블 로우 변경 (엔티티)
- **executeUpdate()의 결과는 영향받은 엔티티 수 반환**
- **UPDATE, DELETE 지원**
- **INSERT(INSERT INTO .. SELECT, 하이버네이트 지원)**

**테스트**

```java
@Test
void bulkOperationTest() {
    EntityManager entityManager = emf.createEntityManager();
    EntityTransaction transaction = entityManager.getTransaction();
    transaction.begin();

    try {

        int resultCount = entityManager.createQuery("update Member m set m.age = 20")
                .executeUpdate();

        // 트랜잭션 커밋
        transaction.commit();
    } catch (Exception e) {
        e.printStackTrace();
        transaction.rollback();
    } finally {
        entityManager.close();
    }
}
```

**출력 콘솔**

```java
Hibernate: 
    /* update
        Member m 
    set
        m.age = 20 */ update
            Member 
        set
            age=20
```

### 벌크 연산 주의점

- 벌크 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리
- **해결법**
    - 벌크 연산을 먼저 실행
    - **벌크 연산 수행 후 영속성 컨텍스트 초기화**