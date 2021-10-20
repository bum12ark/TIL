## 영속성 전이: CASCADE

- 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶을 때
- 예: 부모 엔티티를 저장할 때 자식 엔티티도 함께 저장
- 해당 엔티티가 persist 될 경우, cascade 가 붙은 객체에 대하여 영속성 전이가 이루어진다.

### **Cascade 미사용** **테스트**

```java
@Test
@DisplayName("Cascade를 사용하지 않을 경우 persist를 각각 호출 해야함")
void noCascadeTest() {
    EntityManager entityManager = emf.createEntityManager();
    EntityTransaction transaction = entityManager.getTransaction();
    transaction.begin();

    try {

        Child childA = new Child();
        Child childB = new Child();

        Parent parent = new Parent();
        parent.addChild(childA);
        parent.addChild(childB);

        entityManager.persist(parent);
        entityManager.persist(childA);
        entityManager.persist(childB);

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

### Cascade 사용 테스트

**Parent 객체**

```java
@Entity
public class Parent {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL)
    private List<Child> children = new ArrayList<>();

    public void addChild(Child child) {
        children.add(child);
        child.setParent(this);
    }
}
```

**Child 객체**

```java
@Entity
public class Child {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "parent_id")
    private Parent parent;
}
```

### 영속성 전이: CASCADE 주의

- 영속성 전이는 연관관계를 매핑하는 것과 아무 관련이 없음
- 엔티티를 영속화할 때 연관된 엔티티도 함께 영속화하는 편리함을 제공할 뿐
- **하나의 부모가 자식들을 관리할 때 사용하는 것이 좋다. (ex. 게시판 : 첨부파일 경로)**
    - 소유자가 하나 일 때, 단일 소유자일 경우
    - 단일 엔티티에 종속적일 경우
    - 라이프사이클(CRUD)이 동일할 경우
- 하지만 자식을 여러 부모가 관리할 경우 사용 하지 않는 것이 좋다.
    - 해당 자식을 다른 곳에서 조인하거나 매핑할 경우 데이터가 사라질 수 있기 때문

### CASCADE 종류

- **ALL: 모두 적용**
- **PERSIS: 영속**
- **REMOVE: 삭제**
- MERGE: 병합
- ERFRESH: REFRESH
- DETACH: DETACH

## 고아 객체

- 고아 객체 제거: 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제
- **orphanRemoval = true**

**Parent 객체**

```java
@Entity
public class Parent {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Child> children = new ArrayList<>();

    public void addChild(Child child) {
        children.add(child);
        child.setParent(this);
    }
}
```

**테스트**

```java
@Test
@DisplayName("orphanRemoval 테스트")
void orphanRemovalTest() {
    EntityManager entityManager = emf.createEntityManager();
    EntityTransaction transaction = entityManager.getTransaction();
    transaction.begin();

    try {

        Child childA = new Child();
        Child childB = new Child();

        Parent parent = new Parent();
        parent.addChild(childA);
        parent.addChild(childB);

        entityManager.persist(parent);

        entityManager.flush();
        entityManager.clear();

        Parent findParent = entityManager.find(Parent.class, parent.getId());
        // orphanRemoval = true 로 인하여 DELETE 쿼리
        findParent.getChildren().remove(0);

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
    select
        parent0_.id as id1_3_0_,
        parent0_.name as name2_3_0_ 
    from
        Parent parent0_ 
    where
        parent0_.id=?
Hibernate: 
    select
        children0_.parent_id as parent_i3_0_0_,
        children0_.id as id1_0_0_,
        children0_.id as id1_0_1_,
        children0_.name as name2_0_1_,
        children0_.parent_id as parent_i3_0_1_ 
    from
        Child children0_ 
    where
        children0_.parent_id=?
Hibernate: 
    /* delete relationship.mapping.proxy.domain.Child */ delete 
        from
            Child 
        where
            id=?
```

### 고아 객체 - 주의

- 참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제하는 기능
- **참조하는 곳이 하나일 때 사용해야함!**
- **특정 엔티티가 개인 소유할 때 사용**
- @OneToOne, @OneToMany만 가능
- 참고: 개념적으로 부모를 제거하면 자식은 고아가 된다. 따라서 고아 객체 제거 기능을 활성화하면, 부모를 제거할 때 자식도 함께 제거된다. 이것은 CasecadeType.REMOVE 처럼 동작한다.

## 영속성 전이 + 고아 객체, 생명 주기

- CascadeType.ALL + orphanRemoval = true
- 스스로 생명주기를 관리하는 엔티티는 entityManager.persist()로 영속화, entityManager.remove()로 제거
- 두 옵션을 모두 활성화하면 부모 엔티티를 통해서 자식의 생명주기를 관리할 수 있음
- 도메인 주도 설계(DDD)의 Aggregate Root 개념을 구현할 때 유용