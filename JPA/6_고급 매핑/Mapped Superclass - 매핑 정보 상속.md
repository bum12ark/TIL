## @MaappedSuperclass

- 공통 매핑 정보가 필요할 때 사용 (id, name)

![Untitled](https://user-images.githubusercontent.com/72686708/139000619-5411d174-a23b-48b6-a65b-9d29f0e428d8.png)

- **상속 관계 매핑 X**
- 엔티티 X, 테이블과 매핑 X
- 부모 클래스를 상속 받는 **자식 클래스에 매핑 정보만 제공**
- 조회, 검색 불가(`entityManager.find(BaseEntity)` 불가)
- 직접 생성해서 사용할 일이 없으므로 **추상 클래스 권장**

**클래스**

```java
@MappedSuperclass
public abstract class BaseEntity {
    private String createdBy;
    private LocalDateTime createdDate;
    private String lastModifiedBy;
    private LocalDateTime lastModifiedDate;
}

@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn
public abstract class Item extends BaseEntity {
		// ...
}
```

실행 쿼리

```sql
create table Item (
       DTYPE varchar(31) not null,
        id bigint not null,
        createdBy varchar(255),
        createdDate timestamp,
        lastModifiedBy varchar(255),
        lastModifiedDate timestamp,
        name varchar(255),
        price integer,
        primary key (id)
    )
```

- 테이블과 관계없고, 단순히 엔티티가 공통으로 사용하는 매핑 정보를 모으는 역할
- 주로 등록일, 수정일, 등록자, 수정자 같은 전체 엔티티에서 공통으로 적용하는 정보를 모을 때 사용
- 참고: @Entity 클래스는 엔티티나 @MappedSuperclass로 지정한 클래스만 상속 가능
