언어: **객체지향** (JAVA, Scala, ...)

데이터베이스: **관계형 DB** (Oracle, MySQL, ...)

⇒ **객체를 관계형 DB에 관리** 에서 오는 문제점이 있다.

## SQL 중심적인 개발의 문제점

1. 무한 반복, 지루한 코드
    - CRUD (등록, 조회, 수정)
    - 매핑 작업 (자바 객체 <=> SQL)
    - 컬럼이 추가될 때 마다 쿼리들을 수정해야 한다.
    - **SQL에 의존**적인 개발을 피하기 어렵다.
2. 패러다임의 불일치 (객체 vs 관계형 데이터베이스)
    - 객체를 SQL로 저장하기 위하여 SQL 매퍼를 작성해야 한다.

### 객체와 관계형 데이터베이스의 차이

1. 상속
   
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/50de3c3e-94d2-4714-92df-64eb25ec085c/Untitled.png)
    
    - 조회하는 단계에서 객체간 매핑 작업이 필요해진다.
        - 객체를 쪼개어 만들었기 때문에 각각의 테이블에 따른 조인 쿼리를 만든 후 해당하는 객체 생성 후 객체에 맞도록 변수에 넣어주는 작업
2. 연관관계
   
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ef1054c5-dac9-4a57-926b-a5bc5b56140d/Untitled.png)
    
    - 객체는 참조를 사용: `member.getTeam()`
    - 테이블은 외래 키를 사용: `JOIN ON M.team_id = T.team_id`
    - 객체는 단방향 참조만 가능, RDB는 양방향 참조가 가능
3. 데이터 타입
4. 데이터 식별 방법

### 엔티티 신뢰 문제

**객체 그래프 탐색:** 객체는 자유롭게 객체 그래프를 탐색할 수 있어야 한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/aa7f2042-5a59-4faf-b9b8-6ea289e5760a/Untitled.png)

- 처음 실행하는 SQL에 따라 탐색 범위가 결정되기 때문!
  
    ```java
    Member member = memberDAO.getTeam();
    member.getTeam(); // Not Null
    member.getOrder(); // Null
    member.getOrder().getDelivery(); // null? or not null?
    ```
    
- 개발자가 직접 코드를 까보지 않는 이상 해당 객체의 메소드를 신뢰할 수 없다.

### 모든 객체를 미리 로딩할 수 는 없다

- 그리하여 상황에 따라 동일한 회원 조회 메소드를 여러개 생성한다.
  
    ```java
    memberDAO.getMember(); // Member만 조회
    memberDAO.getMemberWithTeam(); // Member와 Team 조회
    ```
    
- 하지만 위의 방식은 조인 쿼리가 다를 때 마다 SQL문을 생성해야한다는 단점이 존재한다.

⭐ **계층형 아키텍처 진정한 의미의 계층 분할이 어렵다.**

### 비교하기

- ORM 솔루션에서 가져오는 방식과 컬렉션에서 가져오는 방식의 비교가 다르다.

```java
String memberId = "100";
Member member1 = memberDAO.getMember(memberId);
Member member2 = memberDAO.getMember(memberId);
if (member1 == member2) // false: 다르다

Member member1 = list.get(memberId);
Member member2 = list.get(memberId);
if (member1 == member2) // true: 같다
```

### 결론

- 객체답게 모델링 할수록 매핑 작업만 늘어난다.
- 미스매치되는 부분들이 생겨난다. (ex. 비교하기, ...)
- 객체를 자바 컬렉션에 저장 하듯이 DB에 저장할 수 있는 방법은 없을까? ⇒ **JPA!!!**