---
description: 자바 ORM 표준 JPA 프로그래밍
---

# 8장. 프록시와 연관관계 관리

## 프록시

엔티티를 조회할 때 연관된 엔티티들이 항상 사용되는 것은 아니다. 비즈니스 로직에 따라 사용될 때도 있지만 그렇지 않을 때도 있다. JPA는 엔티티가 실제 사용될 때까지 데이터베이스 조회를 지연하는 방법을 제공하는데 이것을 `지연 로딩`이라 한다. 지연 로딩 기능을 사용하려면 실제 엔티티 객체 대신에 데이터베이스 조회를 지연할 수 있는 가짜 객체가 필요한데 이것을 `프록시 객체`라 한다.

### 프록시 기초

```java
// 사용여부에 상관없이 데이터베이스를 조회
Member member = em.find(Member.class, "member1");

// 실제 사용하는 시점에 데이터베이스를 조회 (데이터베이스 접근을 위임한 프록시 객체 반환)
Member member = em.getReference(Member.class, "member1");
```

#### 프록시의 특징

* 프록시 객체는 처음 사용할 때 한 번만 초기화된다.
* 프록시 객체가 초기화되면 프록시 객체를 통해서 실제 엔티티에 접근할 수 있다. \(프록시 객체가 실제 엔티티로 바뀌는 것은 아님\)
* 프록시 객체는 원본 엔티티를 상속받은 객체이므로 타입 체크 주의
* 영속성 컨텍스트에 찾는 엔티티가 이미 있다면 em.getReference\(\)를 호출해도 프록시가 아닌 실제 엔티티를 반환한다.
* 초기화는 영속성 컨텍스트의 도움을 받아야 가능하다. 준영속 상태의 프록시를 초기화하면 예외가 발생

### 프록시와 식별자

엔티티를 프록시로 조회할 때 식별자\(PK\) 값을 파라미터로 전달하는데 프록시 객체는 이 식별자 값을 보관한다.

```java
/* 
 * @Acess(AccessType.PROPERTY)인 경우
 * 프록시는 식별자를 가지고 있으므로 getId()를 호출해도 프록시 초기화를 하지 않는다.
 */
Team team = em.getReference(Team.class, "team1");
team.getId();


/* 
 * 연관관계를 설정할 때는 식별자 값만 사용하므로,
 * 프록시를 사용하면 데이터베이스 접근 횟수를 줄일 수 있다.
 * 
 * 연관관계를 설정할 때는 엔티티 접근 방식을 필드로 설정해도 프록시를 초기화 하지 않는다.
 * @Access(AccessType.FIELD)
 */
Member member = em.find(Member.class, "member1");
Team team = em.getReference(Teamm.class, "team1"); // SQL을 실행하지 않음
member.setTeam(team);
```

### 프록시 확인

```java
// 프록시 인스턴스 초기화 여부 확인
boolean isLoad = em.getEntityManagerFactory()
                   .getPersistenceUnitUtil().isLoaded(entity);
// boolean isLoad = emf.getPersistenceUnitUtil().isLoaded(entity);


// 엔티티인지 프록시로 조회한 것인지 확인하려면 클래스명을 출력 ..javassist..
System.out.printlnf(memberProxy = " + member.getClass().getName());
```

{% hint style="info" %}
#### 프록시 강제 초기화

하이버네이트의 initialize\(\) 메소드를 사용하면 프록시를 강제로 초기화할 수 있다.

org.hibernate.Hibernate.initialize\(order.getMember\(\)\);

JPA 표준에는 프록시 강제 초기화 메소드가 없다. 따라서 강제로 초기화하려면 member.getName\(\)처럼 프록시의 메소드를 직접 호출하면 된다. JPA 표준은 단지 초기화 여부만 확인할 수 있다.
{% endhint %}

## 즉시 로딩과 지연 로딩

### 즉시 로딩

즉시 로딩을 사용하려면 fetch 속성을 `FetchType.EAGER`로 지정한다.

```java
@Entity
public class Member {
    
    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    
    ...
}


/* 
 * 즉시 로딩으로 설정하여 find()로 회원을 조회하는 순간 팀도 함께 조회한다.
 * 대부분의 JPA 구현체는 즉시 로딩을 최적화하기 위해 가능하면 조인 쿼리를 사용(쿼리 두번 x)
 */
Member member = em.find(Member.class, "member1");
Team team = member.getTeam();
```

{% hint style="info" %}
#### NULL 제약조건과 JPA 조인 전략

JPA는 선택적 관계\(null\)면 외부 조인을 사용하고, 필수 관계\(not null\)면 내부 조인을 사용한다.

```java
/*
 * @JoinColumn(nullable = true): NULL 허용(기본값), 외부 조인 (outer join)
 * @JoinColumn(nullable = false): NULL 허용하지 않음, 내부 조인 (inner join)
 */
@Entity
public class Member {

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "TEAM_ID", nullable = false)
    private Team team;
    ...
}


// @ManyToOne.optional = false로 설정해도 내부 조인을 사용
@Entity
public class Member {

    @ManyToOne(fetch = FetchType.EAGER, optional = false)
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    ...
}
```
{% endhint %}

### 지연 로딩

지연 로딩을 사용하려면 fetch 속성을 `FetchType.LAZY`로 지정한다.

```java
@Entity
public class Member {
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    
    ...
}

// 지연 로딩으로 설정하여 find()를 호출하면 회원만 조회하고 팀은 조회하지 않는다.
Member member = em.find(Member.class, "member1");
// 반횐된 객체는 프록시 객체 (만약 영속성 컨텍스트에 로딩되어 있으면 실제 엔티티 사용)
Team team = member.getTeam();
// 실제 팀 객체 사용 (초기화)
team.getName();
```

## 지연 로딩 활용

### 프록시와 컬렉션 래퍼

하이버네이트는 엔티티를 영속 상태로 만들 때 엔티티에 컬렉션이 있으면 컬렉션을 추적하고 관리할 목적으로 원본 컬렉션을 하이버네이트가 제공하는 내장 컬렉션으로 변경하는데 이것을 `컬렉션 래퍼`라 한다.

```java
Member member = em.find(Member.class, "member1");
// 초기화 되지 않음
// member.getOrders().get(0)처럼 실제 데이터를 조회할 때 초기화
List<Order> orders = member.getOrders();
System.out.println("orders = " + orders.getClass().getName());
// orders = org.hibernate.collection.internal.PersistentBag


// Q1. orders는 초기화가 될까?
Order order1 = new Order();
orders.add(order1);
```

### JPA 기본 fetch 전략

* @ManyToOne, @OneToMany : 즉시 로딩\(FetchType.EAGER\)
* @OneToMany, @ManyToMany : 지연 로딩\(FetchType.LAZY\)

### FetchType.EAGER 사용 시 주의점

* **컬렉션을 하나 이상 즉시 로딩하는 것은 권장하지 않는다.** A 테이블을 N, M 두 테이블과 일대다 조인하면 N \* M이 되면서 너무 많은 데이터를 반활할 수 있고, 결과적으로 애플리케이션 성능이 저하될 수 있다.
* **컬렉션 즉시 로딩은 항상 외부 조인을 사용한다.** 다대일 관계인 두 테이블\(회원, 팀\)을 조인할 때 회원 테이블의 외래키에 not null을 걸어두면 내부 조인을 사용해도 된다. 반대로 팀 테이블에서 회원 테이블로 조인할 때 회원이 한 명도 없는 팀을 내부 조인하면 팀까지 조회되지 않는 문제가 발생한다. 따라서 JPA는 일대다 관계를 즉시 로딩할 때 항상 외부 조인을 사용한다.
* **FetchType.EAGER 설정과 조인 전략**
  * @ManyToOne, @OneToOne
    * \(optional = false\): 내부 조인
    * \(optional = true\)  : 외부 조인
  * @OneToMany, @ManyToMany
    * \(optional = false\): 외부 조인
    * \(optional = true\)  : 내부 조인

## 영속성 전이

JPA에서 엔티티를 저장할 때 연관된 모든 엔티티는 영속 상태여야 한다. 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶으면 영속성 전이 기능을 사용하면 된다.  JPA는 `CASCADE` 옵션으로 영속성 전이를 제공한다.

```java
@Entity
public class Parent {

    @Id @GeneratedValue
    private Long id;
    
    @OneToMany(mappedBy = "parent", cascade = CascadeType.PERSIST)
    private List<Child> children = new ArrayList<Child>();
    ...
}

@Entity
public class Child {

    @Id @GeneratedValue
    private Long id;
    
    @ManyToOne
    private Parent parent;
    ...
}

private static void saveWithCascade(EntityManager em) {

    Child child1 = new Child();
    Child child2 = new Child();
    
    Parent parent = new Parent();
    child1.setParent(parent);
    child2.setParent(parent);
    parent.getChildren().add(child1);
    parent.getChildren().add(child2);
    
    // parent만 영속화하면 child까지 함께 영속화해서 저장한다
    em.persist(parent);
}

private static void removeWithCascade(EntityManager em, Parent parent) {

    // CascadeType.REMOVE로 설정하면 연관된 엔티티도 함께 삭제된다.
    // 설정하지 않는다면 외래 키 제약조건으로 인해 ㅁ외래키 무결성 예외가 발생한다.
    em.remove(parent);
}
```

### CASCADE의 종류

```java
public enum CascadeType {
    ALL,
    PERSIST,
    MERGE,
    REMOVE,
    REFRESH,
    DETACH
}
```

## 고아 객체

JPA는 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제하는 기능을 제공하는데 이것을 고아 객체\(ORPHAN\) 제거라 한다.

```java
@Entity
public class Parent {

    @Id @GeneratedValue
    private Long id;
    
    @OneToMany(mappedBy = "parent", orphanRemoval = true)
    private List<Child> children = new ArrayList<Child>();
    ...
}



Parent parent1 = em.find(Parent.class, id);
// 부모 엔티티의 컬렉션에서 자식 엔티티의 참조만 제거하면 자 엔티티가 자동으로 삭제
parent1.getChildren().remove()
```

## 참고

* [테이블 조인](https://12bme.tistory.com/165)
* [JPA 컬렉션](http://wonwoo.ml/index.php/post/992)



