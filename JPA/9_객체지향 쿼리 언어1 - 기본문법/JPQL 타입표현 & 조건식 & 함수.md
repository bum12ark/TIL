## JPQL 타입 표현

- 문자: 'HELLO', 'She''s'
- 숫자: 10L(Long), 10D(Double), 10F(Float)
- Boolean: TRUE, FALSE
- ENUM: jpabook.MemberType.Admin (패키지명 포함)
- 엔티티 타입: TYPE(m) = Member (상속 관계에서 사용)

### JPQL 기타

- SQL과 문법이 같은 식
- EXISTS, IN
- AND, OR, NOT
- =, >, >=, <, <=, <>
- BETWEEN, LIKE, **IS NULL**

## 조건식 - CASE 식

- COALESCE: 하나씩 조회해서 null이 아니면 바환
    - 사용자 이름이 없으면 이름 없는 회원을 반환
        
        ```java
        select coalesce(m.name, '이름 없는 회원') from Member m
        ```
        
- NULLIF: 두 값이 같으면 null 반환, 다르면 첫번째 값 반환
    - 사용자 이름이 '관리자'면 null을 반환하고 나머지는 본인의 이름을 반환
        
        ```java
        select NULLIF(m.name, '관리자') from Member m
        ```
        

## JPQL 기본 함수

- CONCAT
- SUBSTRING
- TRIM
- LOWER, UPPER
- LENGTH
- LOCATE
- ABS, SQRT, MOD
- SIZE, INDEX (JPA 용도)

### 사용자 정의 함수 호출

- 하이버네이트는 사용전 방언에 추가해야 한다.
    - 사용하는 DB 방언을 상속받고, 사용자 정의 함수를 등록한다.