## 객체지향 쿼리 언어 소개

### 다양한 JPA 쿼리 방법

- **JPQL**
- JPA Criteria
- **QueryDSL**
- 네이티브 SQL
- JDBC API 직접 사용, MyBatis, SpringJdbcTemplate 함께 사용

### JPQL

- JPA를 사용하면 엔티티 객체를 중심으로 개발
- 문제는 복잡한 검색 쿼리
- 검색을 할 때도 **테이블이 아닌 엔티티 객체를 대상으로 검색**
- 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능
- 애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요
- JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어 제공
- SQL과 문법 유사 SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 지원
- JPQL은 엔티티 객체를 대상으로 쿼리
- SQL은 데이터베이스 테이블을 대상으로 쿼리

**예제**

```java
String jpql = "select m from Member m where m.name like '%길동%'";

List<Member> results = em.createQuery(jpql, Member.class)
				.getResultList();
```

- 테이블이 아닌 객체(엔티티)를 대상으로 검색하는 객체 지향 쿼리
- SQL을 추상화해서 특정 데이터베이스 SQL에 의존 X
- JPQL을 한마디로 정의하면 객체 지향 SQL
- **단점: 쿼리는 문자열이기 때문에 컴파일 시점에 오류를 잡을 수 없다.**

### Criteria

- 문자가 아닌 자바코드로 JPQL을 작성할 수 있음
- JPQL 빌더 역할
- JPA 공식 가능
- **단점: 너무 복잡하며 실용성이 없다.**
- Criteria 대신에 **QueryDSL 사용 권장**

### QueryDSL

```java
//JPQL
//select m from Member m where m.age > 18
JPAFactoryQuery query = new JPAQueryFactory(em);
QMember m = QMember.member;

List<Member> list = query.selectFrom(m)
				.where(m.age.gt(18))
				.orderBy(m.name.desc())
				.fetch();
```

- 문자가 아닌 자바코드로 JPQL을 작성할 수 있음
- JPQL 빌더 역할
- **컴파일 시점에 문법 오류를 찾을 수 있음**
- 동작쿼리 작성 편리함
- **단순하고 쉬움**
- **실무 사용 권장**

### 네이티브 SQL

- JPA가 제공하는 SQL을 직접 사용하는 기능
- JPQL로 해결할 수 없는 특정 데이터베이스에 의존적인 기능
- 예) 오라클 CONNECT BY, 특정 DB만 사용하는 SQL 힌트

```java
String sql = 
		"SELECT ID, AGE, TEAM_ID, NAME FROM MEMBER WHERE NAME = 'kim'";

List<Member> results = 
				entityManager.createNativeQuery(sql, Member.class).getResultList();
```

### JDBC 직접 사용, SprintJdbcTemplate 등

- JPA를 사용하면서 JDBC 커넥션을 직접 사용하거나, 스프링 JdbcTemplate, 마이바티스 등을 함께 사용 가능
- 단 영속성 컨텍스트를 적절한 시점에 강제로 플러시 필요
- 예) JPA를 우회해서 SQL을 실행하기 직전에 영속성 컨텍스트 수동 플러시

## JPQL (Java Persistence Query Language) - 기본 문법과 기능

### JPQL 소개

- JPQL은 객체지향 쿼리 언어다. 따라서 테이블을 대상으로 쿼리를 하는 것이 아니라 **엔티티 객체를 대상으로 쿼리**한다.
- JPQL의 SQL을 추상화해서 특정데이터베이스 SQL에 의존하지 않는다.
- JPQL은 결국 SQL로 변환된다.

### 예제 모델

![Untitled](https://user-images.githubusercontent.com/72686708/139003173-3191d978-fd7c-413f-b05f-4290c8480a28.png)

### JPQL 문법

```
SELECT_문 :: = 
	SELECT_절
	FROM_절
	[WHERE_절]
	[GROUP BY_절]
	[HAVING_절]
	[ORDER BY_절]

UPDATE_문 :: = UPDATE_절 [WHERE_절]
DELETE_문 :: = DELETE_절 [WHERE_절]
```

- select m from Member as m where m.age > 18
- 엔티티와 속성은 대소문자를 구분 (Member, age)
- JPQL 키워드는 대소문자를 구분하지 않음 (SELECT, FROM , where)
- 엔티티 이름 사용, 테이블 이름이 아님 (Member)
- **별칭은 필수 (m)** (as는 생략가능)

### 집합과 정렬

```sql
select
		COUNT(m),
		SUM(m.age),
		AVG(m.age),
		MAX(m.age),
		MIN(m.age)
from Member m
```

### TypeQuery, Query

- TypeQuery: 반환 타입이 명확할 때 사용
    
    ```java
    TypedQuery<Member> query = 
    		entityManager.createQuery("SELECT m FROM Member m", Member.class);
    ```
    
- Query: 반환 타입이 명확하지 않을 때 사용
    
    ```java
    Query query =
    		entityManager.createQuery("SELECT m.name, m.age FROM Member m");
    ```
    

### 결과 조회 API

- `query.getResultList()`: **결과가 하나 이상일 때**, 리스트 반환
    - 결과가 없으면 빈 리스트 반환
- `query.getSingleResult()`: **결과가 정확히 하나**, 단일 객체 반환
    - 결과가 없으면: javax.persistence.NoResultException
    - 둘 이상이면: javax.persistence.NonUniqueResultException

### 파라미터 바인딩 - 이름 기준, 위치 기준

- 이름 기준
    
    ```java
    String sql = "SELECT m FROM Member m where m.name = :name";
    query.setParameter("name", nameParam);
    ```
    
- 위치 기준
    
    ```java
    String sql = "SELECT m FROM Member m where m.name =?1";
    query.setParameter(1, nameParam);
    ```
