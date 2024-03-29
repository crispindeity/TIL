# JPQL 조인

## 조인 종류

### 내부 조인
- `SELETE m FROM Member m [INNER] JOIN m.team t`
```java
Team team = new Team();
team.setName("teamA");
em.persist(team);

Member member = new Member();
member.setUsername("member");
member.setAge(10);
em.persist(member);


em.flush();
em.clear();

String query = "select m from Member m inner join m.team t";
List<Member> result = em.createQuery(query, Member.class)
                .getResultList();
```
- `inner` 는 생략이 가능하다.
```sql
Hibernate: 
    /* select
        m 
    from
        Member m 
    inner join
        m.team t */ 
    select
        member0_.id as id1_0_,
        member0_.age as age2_0_,
        member0_.team_id as team_id4_0_,
        member0_.username as username3_0_ 
    from
        Member member0_ 
    inner join
        Team team1_ 
            on member0_.team_id=team1_.id
```
- 발생한 `Query`, `inner join` 이 잘 되는걸 확인할 수 있다.

### 외부 조인
- `SELETE m FROM Member m LEFT [OUTER] JOIN m.team t`
```java
Team team = new Team();
team.setName("teamA");
em.persist(team);

Member member = new Member();
member.setUsername("member");
member.setAge(10);
em.persist(member);


em.flush();
em.clear();

String query = "select m from Member m left outer join m.team t";
List<Member> result = em.createQuery(query, Member.class)
                .getResultList();
```
- `outer` 는 생략이 가능하다.
```sql
Hibernate: 
    /* select
        m 
    from
        Member m 
    left outer join
        m.team t */ 
    select
        member0_.id as id1_0_,
        member0_.age as age2_0_,
        member0_.team_id as team_id4_0_,
        member0_.username as username3_0_ 
    from
        Member member0_ 
    left outer join
        Team team1_ 
            on member0_.team_id=team1_.id
```
- 발생한 `Query`

### 세타 조인
- `SELETE COUNT(m) FROM Member m, Team t WHERE m.username = t.name`
```java
Team team = new Team();
team.setName("teamA");
em.persist(team);

Member member = new Member();
member.setUsername("member");
member.setAge(10);
em.persist(member);


em.flush();
em.clear();

String query = "select m from Member m, Team t where m.username = t.name";
List<Member> result = em.createQuery(query, Member.class)
                .getResultList();
```
- 연관관계가 없는 데이터를 조인
- 카테시안 곱이 발생
```sql
Hibernate: 
    /* select
        m 
    from
        Member m,
        Team t 
    where
        m.username = t.name */ 
    select
        member0_.id as id1_0_,
        member0_.age as age2_0_,
        member0_.team_id as team_id4_0_,
        member0_.username as username3_0_ 
    from
        Member member0_ cross 
    join
        Team team1_ 
    where
        member0_.username=team1_.name
```
- 발생한 `Query`, `cross join` 이 발생한다.

## ON 절
- ON 절을 활용한 조인(JPA 2.1부터 지원)

### 조인 대상 필터링
- 회원과 팀을 조인하면서, 팀 이름이 `A` 인 팀만 조인
    - JPQL: `SELECTE m, t FROM Member m LEFT JOIN m.team t on t.name = 'A'`
    - SQL: `SELECT m.*, t.* FROM Member m LEFT JOIN Team t ON m.TEAM_ID = t.id and t.name = 'A'`
- 회원의 이름과 팀의 이름이 같은 대상 외부 조인(연관관계가 없는 엔티티 외부 조인)
    - JPQL: `SELECT m, t FROM Member m LEFT JOIN Team t on m.username = t.name`
    - SQL: `SELECT m.*, t.* FROM Member m LEFT JOIN Team t ON m.username = t.name`

## 서브 쿼리

### 메인 쿼리와 연관이 없는 서브쿼리
- 나이가 평균보다 많은 회원
```sql
SELECT m FROM Member m
WHERE m.age > (select avg(m2.age) from Member m2)
```
- 메인 쿼리와 서브 쿼리가 연관이 없어 따로 동작하므로 성능적인 측면에서 더 좋다.

### 메인 쿼리와 연관이 있는 서브쿼리
- 한 건이라도 주문한 고객
```sql
SELECT m FROM Member m
WHERE (select count(o) from Order o where m = o.member) > 0
```
- 메인 쿼리와 서브 쿼리가 연관이 있는 상태

### 서브 쿼리 지원 함수
- [NOT] EXISTS (subquery): 서브 쿼리에 결과가 존재하면 참
    - {ALL | ANY | SOME} (subquery)
    - ALL: 모두 만족하면 참
    - ANY, SOME: 같은 의미, 조건을 하나라도 만족하면 참
- [NOT] IN (subquery): 서브 쿼리의 결과 중 하나라도 같은 것이 있으면 참 

### 지원 함수 예제
- 팀 A 소속인 회원
```sql
SELECT m FROM Member m
WHERE EXISTS (select t from m.team t where t.name = '팀A')
```

- 전체 상품 각각의 재고보다 주문량이 많은 주문들
```sql
SELECT o FROM `Order` o
WHERE o.orderAmount > ALL (select p.stockAmount from Product p)
```

- 어떤 팀이든 팀에 소속되 회원
```sql
SELECT m FROM Member m
WHERE m.team = ANY (select t from Team t)
```

### JPA 서브 쿼리의 한계
- JPA는 `WHERE`, `HAVING` 절에서만 서브 쿼리를 사용할 수 있다.
- Hibernate 에서는 `SELECT` 절도 지원
- `FROM` 절의 서브 쿼리는 현재 JPQL 에서는 불가능하다.
    - 조인으로 풀 수 있으면 풀어서 해결 할 수 있다.