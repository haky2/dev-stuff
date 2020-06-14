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


//
public class MemberRepository {

    public List<Member> findByUsername(String username) {
        ...
        List<Member> resultList =
            em.createNamedQuery("Member.findByUsername", Member.class)
              .setParameter("username", "member1")
              .getResultList();
    }
}

// 
public interface MemberRepository extends JpaRepository<Member, Long> {

    List<Member> findByUsername(@Param("username") String username);
}
```



