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
* JPA를 우회해서 SQL을 실행하기 직전에 영속성 컨텍스트를 수동으로 플러시해서 데이터베이스와 영속성 컨텍스트를 동기화하여 사용

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
// where m.username='회원1'
// order by m.age desc

CriteriaBuilder cb = em.getGriteriaBuilder();
CriteriaQeury<Member> cq = cb.createQuery(Member.class);
// from 생성
Root<Member> m = cq.from(Member.class);
// 검색 조건 정의
Predicate usernameEqual = cb.equal(m.get("username"), "회원1");
// 정렬 조건 정의
javax.persistence.criteria.Order ageDesc = cb.desc(m.get("age"));
// 쿼리 생성
cq.select(m)
  .where(usernameEqual)
  .orderBy(ageDesc);
  
List<Member> resultList = em.createQuery(cq).getResultList();
```

## QueryDSL

```java
public void queryDSL() {

    EntityManager em = emf.createEntityManager();
    
    JPAQuery query = new JPAQuery(em);
    QMember qMmeber = new QMember("m");
    List<Member> members = query.from(qMember)
                                .where(qMember.name.eq("회원1"))
                                .orderBy(qMember.name.desc())
                                .list(qMember);
}
```

## Native SQL

native SQL을 사용하면 엔티티를 조회할 수 있고 JPA가 지원하는 영속성 컨텍스트의 기능을 그대로 사용할 수 있다.   
반면에 JDBC API를 직접 사용하면 단순히 데이터의 나열을 조회할 뿐이다.

```java
// 결과 타입 정의
public Query createNativeQuery(String sqlString, Class resultClass);

// 결과 타입을 정의할 수 없을 때
public QUery createNativeQuery(String sqlString);

// 결과 매핑 사용
public Query createNativeQuery(String sqlString, String resultSetMapping);
```

## 객체지향 쿼리 심화

### 벌크 연산

벌크 연산을 사용할 때는 벌크 연산이 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리한다는 점에 주의해야 한다.  
따라서, 벌크 연산을 먼저 수행하던지, 벌크 연산 수행 후 영속성 컨텍스트 초기화 혹은 em.refresh\(\)를 사용하는 것이 필요하다.

```java
String qlString = 
    "update Product p " +
    "set p.price = p.price * 1.1 " +
    "where p.stockAmount < :stockAmount";

String qlString2 = 
    "delete from Product p " +
    "where p.price < :price";

int resultCount = em.createQeury(qlString)
                    .setParameter("stockAmount", 10)
                    .executeUpdate();
//                    .setParameter("price", 100)
//                    .executeUpdate();
```

### 영속성 컨텍스트와 JPQL

#### 쿼리 후 영속 상태인 것과 아닌 것

조회한 엔티티만 영속성 컨텍스트가 관리한다. \(ex. 임베디드 타입은 조회 후 값을 변경해도 수정이 발생하지 않는다.\)

#### JPQL로 조회한 엔티티와 영속성 컨텍스트

* JPQL로 조회한 엔티티는 영속 상태다
* 영속성 컨텍스트에 이미 존재하는 엔티티가 있으면 기존 엔티티를 반환한다

영속성 컨텍스트는 영속 상태인 엔티티의 동일성을 보장하기 때문에 기존 엔티티는 그대로 두고 새로 검색한 엔티티를 버린다.

find\(\)는 영속성 컨텍스트에서 먼저 찾고 없으면 데이터베이스에서 찾는다 \(1차 캐시\)  
JPQL은 항상 데이터베이스에 SQL을 실행해서 결과를 조회한다. 그후 영속성 컨텍스트에 저장하고, 다시 조회하게되면 데이터베이스에 먼저 조회 후 영속성 컨텍스트에 같은 엔티티가 있다면 새로 검색한 엔티티는 버리고 기존 엔티티를 반환한다.

### JPQL과 플러시 모드

* em.setFlushMode\(FlushModeType.AUTO\); // 커밋 또는 쿼리 실행 시 플러스 \(기본값\)
* em.setFlushMode\(FlushModeType.COMMIT\); // 커밋시에만 플러시

JPQL은 영속성 컨텍스트에 있는 데이터를 고려하지 않고 데이터를 조회하므로, JPQL 실행 전 영속성 컨텍스트의 내용을 데이터베이스에 반영해야 한다.

