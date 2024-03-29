# JPA의 다양한 쿼리 사용 방법

## JPQL
- JPA 를 사용하게 되면 Entity 객체를 중심으로 개발하게 되는데, 검색 쿼리를 어떻게 처리해야할까?
    - 검색을 할 때도 테이블이 아닌 Entity 객체를 대상으로 검색해야 한다.
    - 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능에 가깝다.
    - 애플리케이션이 필요한 데이터만 DB 에서 불러오려면 결국 검색 조건이 포함된 SQL 이 필요하다.
- JPA 는 SQL 을 추상화한 JPQL 이라는 객체 지향 쿼리 언어를 제공한다.
- SQL과 문법이 유사하며, ANSI SQL 에서 지원하는 모든것을 사용할 수 있다.
    - SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 등등
- JPQL 은 Entity 객체를 대상으로 쿼리
- SQL 은 DB 테이블을 대상으로 쿼리
- JPQL 을 작성하면, SQL 로 변역되어 동작하게 된다.
```java
String jpql = "select m from Member m where m.name like '%hello%'";

List<Member> result = em.createQuery(jpql, Member.class).getResultList();
```
- JPQL 작성

```sql
Hibernate:
    /* 
    select
        m
    from
        Member m
    where
        m.name like '%hello%'
    */
        member0_.member_id,
        member0_.city,
        member0_.street,
        member0_.zipcode,
        member0_.name,
    from 
        Member member0_
    where
        member0_.name like '%hello%'
```
- JPQL 이 번역되어 발생한 SQL
- 위에 주석 처리를 통해 작성된 JPQL 을 보여주고, 아래에 실제 발생된 SQL 을 보여준다.
- 테이블이 아닌 객체를 대상으로 검색하는 객체지향 쿼리이다.
- SQL 을 추상화해서, 특정 데이터베이스 SQL 에 의존적이지 않다.

## Criteria
- 동적 쿼리를 활용하기 어려운 JPQL 의 단점을 보완할때 사용.
- 동적 쿼리를 활용할때 뿐만 아니라 다른 여러 기능을 활용할 수 있게 해준다.
```java
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> = cb.createQuery(Member.class);

Root<Member> m = query.from(Member.class);

CriteriaQuery<Member> cq = query.select(m).where(cb.equal(m.get("name"), "kim"));
List<Member> result = em.createQuery(cq).getResultList();
```
- 단순 사용 예제

```java
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> query = cb.createQuery(Member.class);

Root<Member> m = query.from(Member.class);
CriteriaQuery<Member> cq = query.select(m);

String name = "hello";

if (name != null) {
    cq = cq.where(cb.equal(m.get(name), "kim"));
}

List<Member> result = em.createQuery(cq).getResultList();
```
- 동적 쿼리 작성 예제

### Criteria 정리
- 문자(String)가 아닌 자바 코드로 JPQL 을 작성 할 수 있다.
- JPQL 빌더 역할
- JPA 공식 기능
- JPQL 에 비해 단순하지만, SQL 보다 직관적이지 않아 유지 보수 측면에서 매우 좋지 않다.(실무에서 활용하기에는 무리가 있다.)
- Criteria 대신 QueryDSL 사용을 권장한다.

## QueryDSL
- JPQL 을 좀 더 쉽게 활용하기 위해 사용하는 오픈소스 라이브러리
- JPQL 빌더 역할
- 사용하기 전에 추가적인 설정이 필요하지만, 사용하기에는 매우 편리하다.
- 컴파일 시점에 문법 오류를 찾을 수 있다.
- 단순하고 쉽다.(실무에서 활용하기 좋다.)
```java
JPAFactoryQuery query = new JPAQueryFactory(em);
QMember m = QMember.member;

List<Member> list = 
    query
    .selectFrom(m)
    .where(m.age.gt(18))
    .orderBy(m.name.desc())
    .fetch();
```
- 단순 사용 예제


## 네이티브 SQL
- JPA 가 제공하는 SQL 을 직접 사용하는 기능
- JPQL 로 해결할 수 없는 특정 데이터베이스에 의존적인 기능
    - 오라클 DB 의 CONNECT BY 등 특정 DB 만 사용하는 SQL 힌트
```java
em.createNativeQuery("select member_id, city, street, zipcode, username from Member")
    .getResultList();
```
- 단순 사용 예제

```sql
Hibernate:
    /* dynamic native SQL query */
    select
        member_id,
        city,
        street,
        zipcode,
        username
    from
        Member
```
- 작성한 NativeQuery 그대로 적용되는것을 볼 수 있다.

## JDBC 직접 사용
- 네이티브 SQL 을 사용하기 보다 JDBC 를 직접 사용하는게 좋다.
- JPA 를 사용하면서 JDBC 커넥션을 직접 사용하거나, 스프링 JdbcTemplate, 마이바티스 등을 함께 사용할 수 있다.
- 주의할 점은 영속성 컨텍스트를 적절한 시점에 강제로 flush 해줘야 한다.
    - JPA 를 우회해서 SQL 을 실행하기 직전에 영속성 컨텍스트를 수동 flush 해줘야 한다.
    - flush 가 동작하는 시기: commit 되기 직전 그리고 query 가 발생 했을때

```java
Member member = new Member();
member.setName("member1");
em.persist(member);

// Select Query가 발생하여, 자동으로 flush를 해준다.
List<Member> result = em.createNativeQuery(sql).getResultList(); 

for (Member m : result) {
    System.out.println("member = " + m);
}
```
- JPA 기술을 사용할때는 자동으로 flush 해줘서 문제가 없다.

```java
Member member = new Member();
member.setName("member1");
em.persist(member);

// em.flush(); 강제로 flush()
// dbconn.executeQuery("select * from member");
```
- JDBC 는 JPA 와 관련이 없기 때문에, 자동이로 flush 를 해주지 않는다.
- SQL 이 실행되기 전에 강제로 flush 를 꼭 해줘야 한다.

----
### REFERENCE

- [자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)

---
#JPA_다양한_쿼리_사용_방법