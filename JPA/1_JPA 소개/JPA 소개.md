## JPA?

- Java Persistence API
- 자바 진영의 **ORM** 기술 표준
- JPA는 인터페이스의 모음
- JPA 표준을 구현한 3가지 구현체 (하이버네이트, EclipseLink, DataNucleus)

## ORM?

- Object-relational mapping(객체 관계 매핑)
- 객체는 객체대로 설계
- 관계형 데이터베이스는 관계형 데이터베이스대로 설계
- **ORM 프레임워크가 중간에 매핑**

## JPA는 애플리케이션과 JDBC 사이에서 동작

![Untitled](https://user-images.githubusercontent.com/72686708/138983415-995f5f0d-235f-489d-aa2e-964c2a3ce9c1.png)

## JPA 장점 (사용해야하는 이유)

- SQL 중심적인 개발에서 객체 중심으로 개발
- 생산성, 유지보수
- 패러다임의 불일치 해결
- 성능, 표준
- 데이터 접군 추상화와 벤더 독립성
- JPA를 JAVA Collections 처럼 사용 가능

### 생산성

- 저장: **jpa.persist**(member)
- 조회: Member member = **jpa.find**(memberId)
- 수정: **member.setName**("변경할 이름")
- 삭제: **jpa.remove**(member)

### 유지보수

- 기존: 연관된 SQL문을 찾아 수정
- JPA 사용: 필드만 추가하면 됨, SQL은 JPA가 처리

### JPA와 패러다임의 불일치 해결

1. JPA와 상속
   - 상속관계(일대다, 일대일, ..)에 맞추어 JPA가 쿼리를 생성해준다.
   - INSERT의 경우 테이블에 맞추어 쿼리를 분할
   - SELECT의 경우 테이블에 맞추어 조인 쿼리를 만들어 줌
2. JPA와 연관관계
   - 마치 자바 컬렌션에 넣은 것처럼 호출 가능하다.
3. JPA와 객체 그래프 탐색
   - 지연로딩을 사용하여 사용 시점에 SQL이 호출된다.
4. JPA와 비교하기
   - 동일 트랜잭션에서 조회한 엔티티는 같음을 보장

## JPA의 성능 최적화 기능

1. 1차 캐시와 동일성(identity) 보장
   - 같은 트랜잭션 안에서는 같은 엔티티를 반환 - 조회 성능 향상
   - DB Isolation Level이 Read Commit이어도 애플리케이션에서 Repeatable Read 보장
2. 트랜잭션을 지원하는 쓰기 지연 (transcational write-behind)
   - 트랜잭션을 커밋할 때까지 INSERT SQL을 모음
   - JDBC BATCH SQL 기능을 사용해서 한번에 SQL 전송
3. 지연 로딩 (Lazy Loading)
   - 지연 로딩: 객체가 실제 사용될 때 로딩
   - 즉시 로딩: JOIN SQL로 한번에 연관된 객체까지 미리 조회

## 데이터베이스 방언

![Untitled 1](https://user-images.githubusercontent.com/72686708/138983432-169bbba9-60c0-4288-ae1b-71c3d986f697.png)

- JPA는 특정 데이터베이스에 종속 X
- 각각 데이터베이스가 제공하는 SQL 문법과 함수는 조금씩 다르다.
  - 가변문자: MySQL은 VARCHAR, Oracle은 VARCHAR2
  - 페이징: MYSQL은 LIMIT, Oracle은 ROWNUM
- 방언: SQL 표준을 지키지 않는 특정 데이터베이스만의 고유한 기능

### DB를 JAVA Collections 처럼 사용 가능

- 자바 컬렉션에서의 리스트와 같이 객체를 가져온 다음 값을 변경한 뒤 컬렉션에 다시 넣지 않는 것처럼 데이터베이스를 조작할 수 있다.
- JPA의 컬럼 수정 시 영속성 컨텍스트를 통하여 엔티티(객체)를 가져온 후 엔티티의 값을 변경하는 것만으로도 영속성 컨텍스트가 플러쉬 될 때 적용된다.

## JPA 구동 방식

![Untitled 2](https://user-images.githubusercontent.com/72686708/138983504-3b882bfb-1e57-46c5-9199-eff1d8b2f3de.png)

## H2 데이터베이스 파일 생성 방법

- h2/bin/build 파일로 실행

![Untitled 3](https://user-images.githubusercontent.com/72686708/138983514-7c99511c-20c7-407a-9733-db0f64c0de18.png)

1. [localhost:8082](http://localhost:8082) 로 접속
2. JDBC URL: 사용할 데이터 베이스 파일을 생성하기 위한 url 작성
3. 연결
4. 이후 부터는 jdbc:h2:tcp://localhost/~/test로 접속 가능

## 주의

- **엔티티 매니저 팩토리**는 하나만 생성해서 애플리케이션 전체에서 공유
- **언티티 매니저**는 쓰레드간에 공유X (사용하고 버려야 한다.) → 트랜잭션 처럼
- **JPA의 모든 데이터 변경은 트랜잭션 안에서 실행**
