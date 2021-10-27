## 페치 조인 (fetch join) - 기본

- SQL 조인의 종류가 아니다.
- JPQL에서 **성능 최적화**를 위해 제공하는 기능
- 연관된 엔티티나 컬렉션을 **SQL 한 번에 함께 조회**하는 기능
- `join fetch` 명령어 사용
- `[ LEFT [OUTER] | INNER] JOIN FETCH 조인경로`

### 엔티티 페치 조인

- 회원을 조회하면서 연관된 팀도 함께 조회
- SQL을 보면 회원 뿐만 아니라 팀도 함께 조회 된다.
- **JPQL**
    
    ```
    SELECT m FROM Member m JOIN FETCH m.team
    ```
    
- **SQL**
    
    ```sql
    SELECT m.*, **t.*** FROM member m
    INNER JOIN TEAM team t ON m.team_id = t_id
    ```
    

### 페치 조인 사용 코드

**예제 시나리오**

![Untitled](https://user-images.githubusercontent.com/72686708/139003480-5c59c9bc-d185-4207-9717-84fb70f1ad9c.png)

```java
void insertFetchJoinData(EntityManager entityManager) {
      Team teamA = new Team("팀A");
      entityManager.persist(teamA);
      Team teamB = new Team("팀B");
      entityManager.persist(teamB);
      Team teamC = new Team("팀C");
      entityManager.persist(teamC);

      Member member1 = new Member("회원1", 10);
      member1.changeTeam(teamA);
      entityManager.persist(member1);
      Member member2 = new Member("회원2", 20);
      member2.changeTeam(teamA);
      entityManager.persist(member2);
      Member member3 = new Member("회원3", 30);
      member3.changeTeam(teamC);
      entityManager.persist(member3);
      Member member4 = new Member("회원4", 40);
      entityManager.persist(member4);

      entityManager.flush();
      entityManager.clear();
  }
```

**테스트 코드**

```java
@Test
@DisplayName("페치 조인 예제 시나리오")
void fetchJoinScenarioTest() {
    EntityManager entityManager = emf.createEntityManager();
    EntityTransaction transaction = entityManager.getTransaction();
    transaction.begin();

    try {

        this.insertFetchJoinData(entityManager);

        String query = "SELECT m FROM Member m JOIN FETCH m.team";
        List<Member> results = entityManager.createQuery(query, Member.class)
                .getResultList();

        for (Member member : results) {
            System.out.println("member.getName() = " + member.getName());
            if (member.getTeam() != null) {
								System.out.println("entityManager.contains(member.getTeam()) = " + 
                            entityManager.contains(member.getTeam()));
                System.out.println("member.getTeam().getName() = " + member.getTeam().getName());
            } else {
                System.out.println("member.getTeam() = null");
            }
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
Hibernate: 
    /* SELECT
        m 
    FROM
        Member m 
    JOIN
        FETCH m.team */ select
            member0_.id as id1_0_0_,
            team1_.id as id1_3_1_,
            member0_.age as age2_0_0_,
            member0_.name as name3_0_0_,
            member0_.TEAM_ID as team_id4_0_0_,
            team1_.name as name2_3_1_ 
        from
            Member member0_ 
        inner join
            Team team1_ 
                on member0_.TEAM_ID=team1_.id

member.getName() = 회원1
entityManager.contains(member.getTeam()) = true
member.getTeam().getName() = 팀A
member.getName() = 회원2
entityManager.contains(member.getTeam()) = true
member.getTeam().getName() = 팀A
member.getName() = 회원3
entityManager.contains(member.getTeam()) = true
member.getTeam().getName() = 팀C
```

- 페치 조인으로 회원과 팀을 함께 조회 한다.
- 한 번에 Member와 Team을 가져오기 때문에 지연로딩이 발생하지 않는다!
- 일반적인 Join과 다르게 연관 Entity도 영속성 컨텍스트에 의해 관리되는 것을 확인할 수 있다.
- N + 1 문제를 해결할 수 있다.

### 컬렉션 페치 조인

- 일대다 관계, 컬렉션 페치 조인

**테스트 코드**

```java
@Test
@DisplayName("일대다 관계, 컬렉션 페치 조인 테스트")
void collectionFetchJoinTest() {
    EntityManager entityManager = emf.createEntityManager();
    EntityTransaction transaction = entityManager.getTransaction();
    transaction.begin();

    try {

        this.insertFetchJoinData(entityManager);

        String query = "SELECT t FROM Team t JOIN FETCH t.members";
        List<Team> results = entityManager.createQuery(query, Team.class)
                .getResultList();

        System.out.println("results.size() = " + results.size());
        for (Team team : results) {
            for (Member member : team.getMembers()) {
                System.out.println("[" + team + "] " + team.getName() + " = " + member);
            }
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
Hibernate: 
    /* SELECT
        t 
    FROM
        Team t 
    JOIN
        FETCH t.members */ select
            team0_.id as id1_3_0_,
            members1_.id as id1_0_1_,
            team0_.name as name2_3_0_,
            members1_.age as age2_0_1_,
            members1_.name as name3_0_1_,
            members1_.TEAM_ID as team_id4_0_1_,
            members1_.TEAM_ID as team_id4_0_0__,
            members1_.id as id1_0_0__ 
        from
            Team team0_ 
        inner join
            Member members1_ 
                on team0_.id=members1_.TEAM_ID
results.size() = 3
[jpql.domain.Team@75023c53] 팀A = Member{id=4, name='회원1', age=10}
[jpql.domain.Team@75023c53] 팀A = Member{id=5, name='회원2', age=20}
[jpql.domain.Team@75023c53] 팀A = Member{id=4, name='회원1', age=10}
[jpql.domain.Team@75023c53] 팀A = Member{id=5, name='회원2', age=20}
[jpql.domain.Team@3596b249] 팀C = Member{id=6, name='회원3', age=30}
```

![Untitled 1](https://user-images.githubusercontent.com/72686708/139003503-6aa1a95f-092b-43fb-ae96-c52d0982414f.png)

![Untitled 2](https://user-images.githubusercontent.com/72686708/139003510-5ac5941d-ce3f-44b8-b4b9-3bffc3d2c5e3.png)

- 관계형 DB는 조인을 할경우 로우의 수가 증가되도록 설계
- JPA는 DB의 결과 수 만큼 컬렉션 을 만들어 돌려주도록 설계
    - DB의 출력 로우 수 = result 리스트의 사이즈

### 페치 조인과 DISTINCT

- SQL의 DISTINCT는 중복된 결과를 제거하는 명력
- JPQL의 DISTINCT는 2가지 기능 제공
    1. SQL에 DISTINCT를 추가
    2. 애플리케이션에서 엔티티 중복 제거

**테스트**

```java
@Test
@DisplayName("DISTINCT 페치 조인 테스트")
void distinctFetchJoinTest() {
    EntityManager entityManager = emf.createEntityManager();
    EntityTransaction transaction = entityManager.getTransaction();
    transaction.begin();

    try {

        this.insertFetchJoinData(entityManager);

        String query = "SELECT DISTINCT t FROM Team t JOIN FETCH t.members";
        List<Team> results = entityManager.createQuery(query, Team.class)
                .getResultList();

        System.out.println("results.size() = " + results.size());
        for (Team team : results) {
            for (Member member : team.getMembers()) {
                System.out.println("[" + team + "] " + team.getName() + " = " + member);
            }
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
Hibernate: 
    /* SELECT
        DISTINCT t 
    FROM
        Team t 
    JOIN
        FETCH t.members */ select
            distinct team0_.id as id1_3_0_,
            members1_.id as id1_0_1_,
            team0_.name as name2_3_0_,
            members1_.age as age2_0_1_,
            members1_.name as name3_0_1_,
            members1_.TEAM_ID as team_id4_0_1_,
            members1_.TEAM_ID as team_id4_0_0__,
            members1_.id as id1_0_0__ 
        from
            Team team0_ 
        inner join
            Member members1_ 
                on team0_.id=members1_.TEAM_ID

results.size() = 2
[jpql.domain.Team@dab1f89] 팀A = Member{id=4, name='회원1', age=10}
[jpql.domain.Team@dab1f89] 팀A = Member{id=5, name='회원2', age=20}
[jpql.domain.Team@781711b7] 팀C = Member{id=6, name='회원3', age=30}
```

![Untitled 3](https://user-images.githubusercontent.com/72686708/139003519-bcd4af67-a1aa-4e19-8ae0-8e03c107637a.png)

- SQL에 DISTINCT를 추가하지만 데이터가다르므로 SQL 결과에서 중복제거 실패
- DISTINCT가 추가로 애플리케이션에서 중복 제거 시도
- 같은 식별자를 가진 **Team 엔티티 제거**

### 페치 조인과 일반 조인의 차이

- 일반 조인 실행 시 연관된 엔티티를 함께 조회하지 않음
- JPQL은 결과를 반환할 때 연관관계 고려 X
- 단지 SELECT 절에 지정한 엔티티만 조회할 뿐
- 페치 조인을 사용할 때만 연관된 엔티티도 **함께 조회 (즉시 로딩)**
- **페치 조인은 객체 그래프를 SQL로 한번에 조회하는 개념**

## 페치 조인의 특징과 한계

- **페치 조인 대상에는 별칭을 줄 수 없다.**
    - 객체그래프는 데이터를 모두 조회해야 하기 때문
    - 특정 컬럼만을 조회할 수 없다.
- **둘 이상의 컬렉션은 페치 조인할 수 없다.**
- **컬렉션을 페치 조인하면 페이징 API(setFirstResult, setMaxResults)를 사용할 수 없다.**
    - 일대일, 다대일 같은 단일 값 연관 필드들은 페치 조인해도 페이징 가능
    - 하이버네이트는 경고 로그를 남기고 메모리에서 페이징 (매우 위험)
- 연관된 엔티티들은 SQL 한번으로 조회 - 성증 최적화
- 엔티티에 직접 적용하는 글로벌 로딩 전략보다 우선함
    - `@OneToMany(fetch = FetchType.LAZY)`
- 실무에서 글로벌 로딩 전략은 모두 지연 로딩
- 최적화가 필요한 곳은 페치 조인 적용

## 페치 조인 - 정리

- 모든 것을 페치 조인으로 해결할 수는 없음
- 페치 조인은 객체 그래프를 유지할 때 사용하면 효과적
- 여러 테이블을 조인해서 엔티티가 가진 모양이 아닌 전혀 다른 결과를 내야하면, 페치 조인 보다는 일반 조인을 사용하고 필요한 데이터들만 조회해서 DTO로 반환하는 것이 효과적
