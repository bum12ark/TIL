## 프로젝션

- SELECT 절에 조회할 대상을 지정하는 것
- **프로젝션 대상**
    - 엔티티
        - SELECT **m** FROM Member m → 엔티티 프로젝션
        - SELECT **m.team** FROM Member m → 엔티티 프로젝션
    - 임베디드 타입
        - SELECT **m.address** FROM Member m → 임베디드 타입 프로젝션
    - 스칼라 타입 (숫자, 문자 기본 데이터 타입)
        - SELECT **m.name, m.age** FROM Member m → 스칼라 타입 프로젝션
        - DISTINCT로 중복 제거

### 엔티티 프로젝션

- 엔티티 프로젝션은 영속성 컨텍스트에 의해 관리된다.

```java
@Test
@DisplayName("엔티티 프로젝션은 영속성 컨텍스트에 의해 관리된다.")
void entityProjectionTest() {
    EntityManager entityManager = emf.createEntityManager();
    EntityTransaction transaction = entityManager.getTransaction();
    transaction.begin();

    try {
        Member member = new Member();
        member.setName("홍길동");
        member.setAge(10);
        entityManager.persist(member);

        entityManager.flush();
        entityManager.clear();

        List<Member> results = entityManager.createQuery("SELECT m FROM Member m", Member.class)
                .getResultList();

        Member findMember = results.get(0);
				// update 쿼리 실행 ??
        findMember.setAge(20);

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

```java
Hibernate: 
    /* SELECT
        m 
    FROM
        Member m */ select
            member0_.id as id1_0_,
            member0_.age as age2_0_,
            member0_.name as name3_0_,
            member0_.TEAM_ID as team_id4_0_ 
        from
            Member member0_
Hibernate: 
    /* update
        jpql.domain.Member */ update
            Member 
        set
            age=?,
            name=?,
            TEAM_ID=? 
        where
            id=?
```

### 프로젝션 - 여러 값 조회

- SELECT **m.name, m.age** FROM Member m
1. Query 타입으로 조회
2. Object[] 타입으로 조회
3. new 명령어로 조회
    - 단순 값을 DTO로 바로 조회
        
        ```java
        SELECT **new** jpql.MemberDto(m.name, m.age) FROM Member m
        ```
        
    - 패키지 명을 포함한 전체 클래스 명 입력
    - 순서와 타입이 일치하는 생성자 필요

**테스트**

```java
@Test
@DisplayName("프로젝션 여러 타입으로 조회")
void projectionVariousTypeSelection() {
    EntityManager entityManager = emf.createEntityManager();
    EntityTransaction transaction = entityManager.getTransaction();
    transaction.begin();

    try {
        Member member = new Member();
        member.setName("홍길동");
        member.setAge(10);
        entityManager.persist(member);

        entityManager.flush();
        entityManager.clear();

        String sql = "SELECT m.name, m.age FROM Member m";

        // Query 타입으로 조회
        System.out.println("1. Query 타입으로 조회");
        List queryTypeResults = entityManager.createQuery(sql)
                .getResultList();

        for (Object result : queryTypeResults) {
            Object[] resultArray = (Object[]) result;
            System.out.println("name = " + resultArray[0]);
            System.out.println("age = " + resultArray[1]);
        }

        // 2. Object[] 타입으로 조회 (Type 쿼리 사용)
        System.out.println("2. Object[] 타입으로 조회");
        List<Object[]> typeQueryResults = entityManager.createQuery(sql)
                .getResultList();

        for (Object result : typeQueryResults) {
            Object[] resultArray = (Object[]) result;
            System.out.println("name = " + resultArray[0]);
            System.out.println("age = " + resultArray[1]);
        }

        // 3. new 명령어로 조회
        System.out.println("3. new 명령어로 조회");
        List<MemberDto> newOperationResults = 
                entityManager.createQuery("select new jpql.MemberDto(m.name, m.age) from Member m"
                        , MemberDto.class)
                .getResultList();

        for (MemberDto newOperationResult : newOperationResults) {
            System.out.println("newOperationResult.getName() = " + newOperationResult.getName());
            System.out.println("newOperationResult.getAge() = " + newOperationResult.getAge());
        }

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
1. Query 타입으로 조회
Hibernate: 
    /* SELECT
        m.name,
        m.age 
    FROM
        Member m */ select
            member0_.name as col_0_0_,
            member0_.age as col_1_0_ 
        from
            Member member0_
name = 홍길동
age = 10
2. Object[] 타입으로 조회
Hibernate: 
    /* SELECT
        m.name,
        m.age 
    FROM
        Member m */ select
            member0_.name as col_0_0_,
            member0_.age as col_1_0_ 
        from
            Member member0_
name = 홍길동
age = 10
3. new 명령어로 조회
Hibernate: 
    /* select
        new jpql.MemberDto(m.name,
        m.age) 
    from
        Member m */ select
            member0_.name as col_0_0_,
            member0_.age as col_1_0_ 
        from
            Member member0_
newOperationResult.getName() = 홍길동
newOperationResult.getAge() = 10
```

## 페이징

- JPA는 페이징을 다음 두 API로 추상화
- **setFirstResult**(int startPosition): 조회 시작 위치 (0부터 시작)
- **setMaxResult**(int maxResult): 조회할 데이터 수

### 페이징 예시

테스트

```java
@Test
@DisplayName("페이징 사용 예제")
void pagingTest() {
    EntityManager entityManager = emf.createEntityManager();
    EntityTransaction transaction = entityManager.getTransaction();
    transaction.begin();

    try {
        // 임의의 멤버 생성
        for (int i = 0; i < 50; i++) {
            Member member = new Member();
            member.setName("회원" + i);
            member.setAge(i + 10);
            // 회원 등록
            entityManager.persist(member);
        }

        entityManager.flush();
        entityManager.clear();

        List<Member> results = entityManager.createQuery("SELECT m FROM Member m ORDER BY m.age DESC", Member.class)
                .setFirstResult(10)
                .setMaxResults(20)
                .getResultList();

        System.out.println("results = " + results);
        
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
    /* SELECT
        m 
    FROM
        Member m 
    ORDER BY
        m.age DESC */ select
            member0_.id as id1_0_,
            member0_.age as age2_0_,
            member0_.name as name3_0_,
            member0_.TEAM_ID as team_id4_0_ 
        from
            Member member0_ 
        order by
            member0_.age DESC limit ? offset ?
results = [Member{id=40, name='회원39', age=49}, Member{id=39, name='회원38', age=48}, Member{id=38, name='회원37', age=47}, Member{id=37, name='회원36', age=46}, Member{id=36, name='회원35', age=45}, Member{id=35, name='회원34', age=44}, Member{id=34, name='회원33', age=43}, Member{id=33, name='회원32', age=42}, Member{id=32, name='회원31', age=41}, Member{id=31, name='회원30', age=40}, Member{id=30, name='회원29', age=39}, Member{id=29, name='회원28', age=38}, Member{id=28, name='회원27', age=37}, Member{id=27, name='회원26', age=36}, Member{id=26, name='회원25', age=35}, Member{id=25, name='회원24', age=34}, Member{id=24, name='회원23', age=33}, Member{id=23, name='회원22', age=32}, Member{id=22, name='회원21', age=31}, Member{id=21, name='회원20', age=30}]
```

### 페이징 방언

- 데이터베이스 방언(Dialect)에 따라 다른 쿼리가 발생한다.
- MySQL: `limit ~ offset`
- ORACLE: `ROWNUM` 3 depth

## 조인

- 내부 조인
    - `SELECT m FROM Member m [INNER] JOIN m.team t`
- 외부 조인
    - `SELECT m FROM Member m (LEFT | RIGHT | FULL) [OUTER] JOIN m.tam t`
- 세타 조인
    - `SELECT m FROM Member m, Team t WHERE m.name = t.name`

### 조인 - 특징

- 연관 Entity에 Join을 걸어도 실제 쿼리에서 SELECT 하는 Entity는 오직 JPQL에서 **조회하는 주체가 되는 Entity만 조회하여 영속화**
- 조회의 주체가 되는 Entity만 SELECT 해서 영속화하기 때문에 연관 Entity 객체에 접근할 경우 지연로딩 발생
- 연관 엔티티의 데이터는 필요하지 않지만 연관 엔티티가 검색조건에는 필요할 경우에 사용됨

### 조인 - ON 절

**ON 절을 활용한 조인**

1. 조인 대상 필터링
    - 예) 회원과 팀을 조인하면서, 팀 이름이 A인 팀만 조인
        - **JPQL**
            
            ```java
            SELECT m, t FROM Member m LEFT JOIN m.team t ON t.name = 'A'
            ```
            
        - **SQL**
            
            ```sql
            SELECT m.*, t.*
              FROM Member m
                   LEFT JOIN Team t
                   ON m.team_id = t.id AND t.name = 'A'
            ```
            
2. 연관관계 없는 엔티티 외부 조인
    - 예) 회원의 이름과 팀의 이름이 같은 대상 외부 조인
        - **JPQL**
            
            ```java
            SELECT m, t FROM
            Member m LEFT JOIN Team t ON m.name = t.name
            ```
            
        - **SQL**
            
            ```sql
            SELECT m.* t.*
              FROM Member m
                   LEFT JOIN Team t
                   ON m.name = t.name
            ```
            

## 서브 쿼리

- 나이가 평균보다 많은 회원
    
    ```java
    SELECT m FROM Member m
    WHERE m.age > **(SELECT AVG(m2.age) FROM Member m2)**
    ```
    
- 한 건이라도 주문한 고객
    
    ```java
    SELECT m FROM Member m
    WHERE **(SELECT COUNT(o) FROM Order o WHERE m = o.member)** > 0
    ```
    

### 서브 쿼리 지원 함수

- [NOT] EXIST (subquery) : 서브 쿼리에 결과가 존재하면 참
    - {ALL |  ANY | SOME} (subquery)
    - ALL: 모두 만족하면 참
    - ANY, SOME: 같은 의미, 조건을 하나라도 만족하면 참
- [NOT] IN (subquery): 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참

### 서브 쿼리 예제

- 팀 A 소속인 회원
    
    ```java
    SELECT m FROM Member m
    WHERE **EXISTS** (SELECT t FROM Team t WHERE t.name = '팀A')
    ```
    
- 전체 상품 각각의 재고보다 주문량이 많은 주문들
    
    ```java
    SELECT o FROM Order o
    WHERE o.orderAmount > **ALL** (SELECT p.stockAmount FROM Product p)
    ```
    
- 어떤 팀이든 팀에 소속된 회원
    
    ```java
    SELECT m FROM Member m
    WHERE m.team = **ANY** (SELECT t FROM Team t)
    ```
    

### JPA 서브 쿼리 한계

- JPA는 WHERE, HAVING 절에서만 서브 쿼리 사용 가능
- SELECT 절도 가능 (하이버 네이트 지원)
- **FROM 절의 서브 쿼리는 현재 JPQL에서 불가능**
    - **조인으로 풀 수 있으면 풀어서 해결**