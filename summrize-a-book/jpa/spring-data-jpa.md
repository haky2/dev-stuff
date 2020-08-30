---
description: 자바 ORM 표준 JPA 프로그래밍
---

# 12장. 스프링 데이터 JPA

## Spring Data JPA

데이터 접근 계층을 개발할 때 구현 클래스 없이 인터페이스만 작성해도 개발을 완료할 수 있다.  
리포지토리를 개발할 때 인터페이스만 작성하면 실행 시점에 스프링 데이터 JPA가 구현 객체를 동적으로 생성해서 주입해준다.

```java
// JpaRepository를 상속받고 제네릭에 엔티티와 엔티티가 사용하는 식별자 지정
public interface MemberRepository extends JpaRepository<Member, Long> {
    Member findByUsername(String username);
}

public interface ItemRepository extends JpaRepository<Item, Long> {
}
```

## 쿼리 메소드 기능

인터페이스에 메소드만 선언하면 해당 메소드의 이름으로 적절한 JPQL 쿼리를 생성해서 실행한다.

### 메소드 이름으로 쿼리 생성

```java
// select m from MEmber m where m.email = ?1 and m.name = ?2
public interface MemberRepository extends Repository<Member, Long> {
    List<Member> findByEmailAndName(String email, String name);
}
```

### JPA NamedQuery

```java
@Entity
@NamedQuery(
    name="Member.findByUsername",
    query="select m from Member m where m.username = :username")
public class Memeber {
    ...
}


// jpa를 직접 사용해서 named 쿼리 호출
public class MemberRepository {

    public List<Member> findByUsername(String username) {
        ...
        List<Member> resultList =
            em.createNamedQuery("Member.findByUsername", Member.class)
              .setParameter("username", "member1")
              .getResultList();
    }
}

// 스프링 데이터 jpa로 named 쿼리 호출
public interface MemberRepository extends JpaRepository<Member, Long> {

    List<Member> findByUsername(@Param("username") String username);
}
```

### @Query

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query("select m from Member m where m.username =?1")
    Member findByUsername(String username);
    
    @Query("select m from Member m where m.username =:name")
    Member findByUsername2(@Parma("name") String username);
    
    @Query(value = "SELECT * FROM MEMBER WHERE USERNAME = ?0",
           nativeQeury = true)
    Member findByUsername3(String username);
}
```

### 벌크성 수정 쿼리

```java
// 벌크성 쿼리를 실행하고 나서 영속성 컨텍스트를 초기화 하는 옵션
// @Modifying(clearAutomatically = true)
@Modifying
@Query("update Product p set p.price = p.price * 1.1 where
    p.stockAmount < :stockAmount")
int bulkPriceUp(@Param("stockAmount") String stockAmount);
```

### 반환 타입

```java
// multiple result (or return empty collection)
List<Member> findByName(String name);
// single result (or return null)
Member findByEmail(String email);
```

### 페이징과 정렬

```java
// count query 사
Page<Member> findByName(String name, Pageable pageable);

// no count query
List<Member> findByName(String name, Pageable pageable);

List<Member> findByName(String name, Sort sort)
```

```java
public interface MemberRepository extends Repository<Member, Long> {

    Page<Member> findByNameStartingWith(String name, Pageable pageable);
}

// 현재 페이지, 조회할 데이터 수, 정렬정보
PageRequest pageRequest = 
  new PageRequest(0, 10, new Sort(Direction.DESC, "name"));
  
Page<Member> result =
  memberRepository.findByNameStartingWith("Kim", pageRequest);

// 조회된 데이터  
List<Member> members = result.getContent();
// 전체 페이지 수
int totalPages = result.getTotalPages();
// 다음 페이지 존재 여부
boolean hasNextPage = result.hasNextPage();
```

## Custom Repository

```java
public interface MemberRepositoryCustom {
    public List<Member> findMemberCustom();
}

// repository interface name + Impl (repository-impl-postfix option)
public class MemberRepositoryImpl implements MemberRepositoryCustom {

    @Override
    public List<Member> findMEmberCustom() {
        ...
    }
}

public interface MemberRepository extends JpaRepository<Member, Long>,
    MemberRepositoryCustom {
}
```

## Spring Data JPA와 QueryDSL

스프링 데이터 JPA는 2가지 방법으로 QueryDSL을 지원한다

### QueryDslPredicateExecutor

스프링 데이터 JPA가 제공하는 페이징과 정렬 기능도 함께 사용할 수 있다.  
하지만 join, fetch를 사용할 수 없다. 따라서 JPAQuery를 직접 사용하거나 QueryDslRepositorySupport를 사용해야 한다.

```java
public interface ItemRepository extends JpaRepository<Item, Long>,
    QueryDslPredicateExecutor<Item> {
}
```

### QueryDslRepositorySupport

```java
public interface CustomOrderRepository {
    public List<Order> search(OrderSearch orderSearch);
}


public class OrderRepositoryImpl extends QueryDslRepositorySupport
    implements CustomOrderRepository {
    
    // 엔티티 클래스 정보를 넘겨주어야 한다
    public OrderRepositoryImpl() {
        super(Order.class);
    }
    
    @Override
    public List<Order> search(OrderSearch orderSearch) {
        QOrder order = QOrder.order;
        QMember member = QMember.member;
        
        JPQLQuery query = from(order);
        
        if (StringUtils.hasText(orderSearch.getMemberName())) {
            query.leftJoin(order.member, member)
                 .where(member.name.contains(orderSearch.getMemberName()));
        }
        if (orderSearch.getOrderStatus() != null) {
            query.where(order.status.eq(orderSearch.getOrderStatus()));
        }
        return query.list(order);
    }
}
```

## 참고

* \*\*\*\*[**JPA의 동일성 보장으로 인해 발생하는 데이터 동기화 문제**](https://devhyogeon.tistory.com/6?category=878035)\*\*\*\*
* \*\*\*\*[**Spring Batch의 멱등성 유지하기**](https://jojoldu.tistory.com/451)\*\*\*\*
* \*\*\*\*[**Spring Boot, Spring Data JPA, Querydsl로 타입 세이프 쿼리 작성하기**](https://jsonobject.tistory.com/462)\*\*\*\*

