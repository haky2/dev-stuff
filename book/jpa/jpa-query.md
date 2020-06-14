---
description: 자바 ORM 표준 JPA 프로그래밍
---

# 10장. 객체지향 쿼리 언어

## 객체지향 쿼리

### JPQL

* 테이블이 아닌 객체\(엔티티\)를 대상으로 검색하는 객체지향 쿼리
* SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않음
* JPA는 JPQL을 분석한 다음 적절한 SQL을 만들어 데이터베이스를 조회하며, 결과로 엔티티 객체를 생성해서 반환

```java
// Member is an entity and m.username is a field in entity (not a table)
String jpql = "select m from Member as m where m.username = 'kim'";
List<Member> resultList = 
    em.cretaeQuery(jpql, Member.class).getResultList();
```

### Criteria

* JPQL을 생성하는 빌더 클래스
* 프로그래밍 코드로 JPQL을 작성할 수 있음
* 복잡하고 장황하여 사용하기 불편하다는 단점

```java
Root<Member> m = query.from(Member.class);
CriteriaQuery<Member> cq = 
    query.select(m).where(cb.equal(m.get("username"), "kim");
List<Member> resultList = em.createQuery(cq).getResultList();
```

### QueryDSL

* JPQL을 생성하는 빌더
* 코드 기반이면서 단순하고 사용하기 쉬움
* 쿼리 전용 클래스를 만들어야 함

```java
// QueryDSL class
QMemeber member = QMember.member;

List<Member> members = query.from(member)
                            .where(member.username.eq("Kim"))
                            .list(member);
```

### Native SQL

* SQL을 직접 사용할 수 있는 기능
* 특정 데이터베이스에 의존하는 기능을 사용할 때 이용 \(ex. oracle의 connect by 등\)
* SQL은 지원하지만 JPQL은 지원하지 않는 기능 등에 사용

### JDBC 직접 사용

* JDBC 커넥션을 획득하여 사용
* JDBC나 MyBatis를 JPA와 함께 사용하면 영속성 컨텍스트를 적절한 시점에 강제로 플러시해야 함
* JPA를 우회해서 SQL을 실행하기 직전에 영속성 컨텍스트를 수동으로 프럴시해서 데이터베이스와 영속성 컨텍스트를 동기화하여 사용

## JPQL

### SELECT

```java
SELECT m FROM Member As m where m.username = "Hello"
```

### TypeQuery, Query

```java
// exact return type
TypedQuery<Member> query =
    em.createQuery("SELECT m FROM Member m", Member.class);
List<Member> resultList = query.getResultList();

// inexact return type
Query query = 
    em.createQuery("SELCET m.username, m.age from Member m);
List resultList = query.getResultList();

for (Object o : resultList) {
    Object[] result = (Object[]) o;
    ....
}
```

### 결과 조회

```java
// return collection
query.getResultList();
// return 1 result
query.getSingleResult();
```

### 파라미터 바인딩

```java
// named parameters
TypedQuery<Member> query = 
    em.createQuery("SELECT m FROM Member m where m.username = :username", Member.class);
query.setParameter("username", "User1");

// positional parameters
List<Member> members =
    em.createQuery("SELECT m FROM Member m where m.username = ?1", Member.class)
    .setParameter(1, "User1").getResultList();
```

### 프로젝션

SELECT 절에 조회할 대상을 지정하는 것을 프로젝션이라 한다.

* 엔티티 프로젝션은 영속성 컨텍스트에서 관리된다.
* 임베디드 프로젝션은 조회의 시작점이 될 수 없고, 엔티티 타입이 아니기 때문에 영속성 컨텍스트에서 관리되지 않는다. \(임베디드 타입은 값 타입\)

```java
// use DTO class
// cautions
// 1. input whole class name(including package name)
// 2. need a constructor
TypedQuery<UserDTO> query = 
    em.createQuery("SELECT new test.jpql.UserDTO(m.username, m.age)
    FROM Member m", UserDTO.class);
```

### 페이징

| method | parameters | desc |
| :--- | :--- | :--- |
| setFirstResult | int startPosition | 조회 시작 위치 \(0부터 시작\) |
| setMaxResults | int maxResult | 조회할 데이터 수 |

### JOIN

```java
// inner join
String innerJoinQuery = "SELECT m FROM Member m [INNER] JOIN m.testm t"
    + "WHERE t.name = :teamName"

// outer join    
String outerJoinQuery = "SELECT m FROM Member m LEFT [OUTER] JOIN m.test t"    
    + "on t.name = 'A'"

// theta join
String thetaJoinQuery = "SELECT count(m) FROM Member m, Team t"
    + "WHERE m.username = t.name"
```

### FETCH JOIN

* fetch join은 sql의 조인 종류는 아니고 JPQL에서 성능 최적화를 위해 제공하는 기능이다.
* 연관된 엔티티나 컬렉션을 한 번에 같이 조회하는 기능. join fetch 명령어로 사용할 수 있다.
* DISTINCT를 추가하면 sql 및 애플리케이션에서 중복을 제거한다

```java
select m from Member m join fetch m.team
```

#### 페치 조인의 특징

* SQL 한 번으로 연관된 엔티티들을 함꼐 조회할 수 있어서 성능을 최적화할 수 있다
* 페치 조인은 글로벌 로딩 전략\(ex. fetch = FetchType.LAZY\) 보다 우선한다. 따라서 글로벌 로딩 전략은 될 수 있으면 지연 로딩을 사용하고 최적화가 필요하면 페치 조인을 적용하는 것이 효과적이다
* 쿼리 시점에 조회하므로 준영속 상태에서도 객체 그래프를 탐색할 수 있다.

#### 페치 조인의 한계

* 페치 조인 대상에는 별칭을 줄 수 없다. \(하이버네이트는 허용\) 따라서 select, where, 서브 쿼리에 페치 조인 대상을 사용할 수 없다.
* 둘 이상의 컬렉션을 페치할 수 없다.
* 컬렉션을 페치 조인하면 페이징 api를 사용할 수 없다

### 서브 쿼리

서브쿼리를 where, having 절에서만 사용할 수 있고, select, from 절에서는 사용할 수 없다.

<table>
  <thead>
    <tr>
      <th style="text-align:left">method</th>
      <th style="text-align:left">syntax</th>
      <th style="text-align:left">desc</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">EXISTS</td>
      <td style="text-align:left">[NOT] EXISTS (subquery)</td>
      <td style="text-align:left">&#xC11C;&#xBE0C;&#xCFFC;&#xB9AC;&#xC5D0; &#xACB0;&#xACFC;&#xAC00; &#xC874;&#xC7AC;&#xD558;&#xBA74;
        &#xCC38;</td>
    </tr>
    <tr>
      <td style="text-align:left">ALL, ANY, SOME</td>
      <td style="text-align:left">[ALL | ANY | SOME] (subquery)</td>
      <td style="text-align:left">
        <p>ALL : &#xBAA8;&#xB450; &#xB9CC;&#xC871;&#xD558;&#xBA74; &#xCC38;</p>
        <p>ANY, SOME : &#xC870;&#xAC74;&#xC744; &#xD558;&#xB098;&#xB77C;&#xB3C4;
          &#xB9CC;&#xC871;&#xD558;&#xBA74; &#xCC38;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">IN</td>
      <td style="text-align:left">[NOT] IN (subquery)</td>
      <td style="text-align:left">&#xC11C;&#xBE0C;&#xCFFC;&#xB9AC;&#xC758; &#xACB0;&#xACFC; &#xC911; &#xD558;&#xB098;&#xB77C;&#xB3C4;
        &#xAC19;&#xC740; &#xAC83;&#xC774; &#xC788;&#xC73C;&#xBA74; &#xCC38;</td>
    </tr>
  </tbody>
</table>

### 정적 쿼리

미리 정의한 쿼리에 이름을 부여해서 필요할 때 사용할 수 있는 쿼리. Named 쿼리라 하며 한 번 정의하면 변경할 수 없다.

```java
@Entity
@NamedQuery(
	name = "Member.findByUsername",
	query = "select m from Member m where m.username = :username")
public class Member {
	...
}
```

## Criteria

```java
// JPQL
// select m from Memeber m
// where m.username=''
// order by m.age desc

CriteriaBuilder cb = em.getGriteriaBuilder();
CriteriaQeury<Member> cq = cb.createQuery(Member.class);
Root<Member> m = cq.from(Member.class);

Predicate usernameEqual = cb.equal(m.get("username"), "");

javax.persistence.criteria.Order ageDesc = cb.desc(m.get("age"));

cq.select(m)
  .where(usernameEqual)
  .orderBy(ageDesc);
  
List<Member> resultList = em.createQuery(cq).getResultList();
```

