---
description: 자바 ORM 표준 JPA 프로그래밍
---

# 7장. 고급 매핑

## 상속 관계 매핑

관계형 데이터베이스에는 객체지향 언어에서 다루는 상속이라는 개념이 없다. ORM에서 이야기하는 상속 관계 매핑은 객체의 상속 구조와 데이터베이스의 슈퍼타입 서브타입 관계를 매핑하는 것이다.

### Joined Strategy

조인 전략\(Joined Strategy\)은 엔티티 각각을 모두 테이블로 만들고 자식 테이블이 부모 테이블의 기본 키를 받아서 `기본 키 + 외래 키로 사용하는 전략`이다.  
이 전략의 사용 시 주의점은 객체는 타입으로 구분할 수 있지만 테이블은 타입의 개념이 없으므로, 타입을 구분하는 컬럼을 추가해야 한다.

```java
// 상속 매핑은 부모 클래스에 @Inheritance를 사용해야 한다
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;
    
    private String name;
    private int price;
    ...
}

// 자식 테이블은 구분 컬럼을 지정한다
@Entity
@DiscriminatorValue("A")
public class Album extends Item {
    
    private String artist;
    ...
}


// 기본값으로 자식 테이블은 부모 테이블의 ID 컬럼명을 그대로 사용. 재정의 가능.
@Entity
@DiscriminatorValue("M")
@PrimaryKeyJoinColumn(name = "BOOK_ID")
public class Movie extends Item {
    
    private String director;
    private String actor;
    ...
}
```

{% tabs %}
{% tab title="장점" %}
* 테이블이 정규화된다.
* 외래 키 참조 무결성 제약조건을 활용할 수 있다.
* 저장공간을 효율적으로 사용한다.
{% endtab %}

{% tab title="단점" %}
* 조회할 때 조인이 많이 사용되므로 성능이 저하될 수 있다.
* 조회 쿼리가 복잡하다.
* 데이터를 등록할 INSERT SQL을 두 번 실행한다.
{% endtab %}

{% tab title="특징" %}
* JPA 표준 명세는 구분 컬럼을 사용하도록 하지만 하이버네이트를 포함한 몇몇 구현체는 구분 컬럼 없이도 동작한다.
{% endtab %}

{% tab title="어노테이션" %}
* @PrimaryKeyJoinColumn, @DiscriminatorColumn, @DiscriminatorValue
{% endtab %}
{% endtabs %}

### Single-Table Strategy

단일 테이블 전략은 `테이블을 하나만 사용하는 전략`이다. 그리고 구분 컬럼으로 어떤 자식 데이터가 저장되었는지 구분한다. 조회할 때 조인을 사용하지 않으므로 일반적으로 가장 빠르다.  
주의점은 자식 엔티티가 매핑한 컬럼은 모두 `null을 허용`해야 한다는 점이다.

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;
    
    private String name;
    private int price;
    ...
}

@Entity
@DiscriminatorValue("A")
public class Album extends Item { ... }

@Entity
@DiscriminatorValue("M")
public class Movie extends Item { ... }

@Entity
@DiscriminatorValue("B")
public class Book extends Item { ... }
```

{% tabs %}
{% tab title="장점" %}
* 조인이 필요 없으므로 일반적으로 조회 성능이 빠르다.
* 조회 쿼리가 단순하다.
{% endtab %}

{% tab title="단점" %}
* 자식 엔티티가 매핑한 컬럼은 모두 null을 허용해야 한다.
* 단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있다. 그러므로 상황에 따라서는 조회 성능이 오히려 느려질 수 있다. 
{% endtab %}

{% tab title="특징" %}
* 구분 컬럼을 꼭 사용해야 한다.
* @DiscriminatorValue를 지정하지 않으면 기본으로 엔티티 이름을 사용한다.
{% endtab %}
{% endtabs %}

### Table-per-Concrete-Class Strategy

구현 클래스마다 테이블 전략은 `자식 엔티티마다 테이블을 만들고 각각에 필요한 컬럼을 만들어 사용하는 전략`이다.  
일반적으로 추천하지 않는 전략이다.

```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;
    
    private String name;
    private int price;
    ...
}

@Entity
public class Album extends Item { ... }

@Entity
public class Movie extends Item { ... }

@Entity
public class Book extends Item { ... }
```

{% tabs %}
{% tab title="장점" %}
* 서브 타입을 구분해서 처리할 때 효과적이다.
* not null 제약조건을 사용할 수 있다. 
{% endtab %}

{% tab title="단점" %}
* 여러 자식 테이블을 함께 조회할 때 성능이 느리다. \(UNION 사용\)
* 자식 테이블을 통합해서 쿼리하기 어렵다. 
{% endtab %}

{% tab title="특징" %}
* 구분 컬럼을 사용하지 않는다.
{% endtab %}
{% endtabs %}

## @MappedSuperclass

부모 클래스는 테이블과 매핑하지 않고 자식 클래스에게 매핑 정보만 제공하고 싶으면 @MappedSuperclass를 사용하면 된다. 이것은 단순히 매핑 정보를 상속할 목적으로만 사용된다.

```java
@MappedSuperclass
public abstract class BaseEntity {

    @Id @GeneratedVAlue
    private Long id;
    private String name;
    ...
}

@Entity
public class Seller extends BaseEntity {

    private String shopName;
    ...
}

// 상속받은 id의 컬럼명을 재정의
@Entity
@AttributeOverride(name = "id", column = @Column(name = "USER_ID"))
public class User extends BaseEntity {

    private String email;
    ...
}

// 상속받은 복수의 필드 재정의
@Entity
@AttributeOverrides({
    @AttributeOverride(name = "id", column = @Column(name = "MEMBER_ID"))
    @AttributeOverride(name = "name", column = @Column(name = "MEMBER_NAME"))
})
public class Member extends BaseEntity {

    private String contact;
    ...
}
```

#### 특징

* 테이블과 매핑되지 않고 자식 클래스에 엔티티의 매핑 정보를 상속하기 위해 사용한다.
* @MappedSuperclass로 지정한 클래스는 엔티티가 아니므로 em.find\(\)나 JPQL에서 사용할 수 없다.
* 이 클래스를 직접 생성해서 사용할 일은 거의 없으므로 추상 클래스로 만드는 것을 권장한다.
* 등록일, 수정일, 등록자, 수정자 같은 공통 엔티티를 효과적으로 관리할 수 있다.

{% hint style="info" %}
엔티티는 @Entity이거나 @MappedSuperclass로 지정한 클래스만 상속받을 수 있다.
{% endhint %}

## 복합 키와 식별 관계 매핑

### 식별 관계 vs 비식별 관계

외래 키가 기본 키에 포함되는지 여부에 따라 식별 관계와 비식별 관계로 구분한다.

#### 식별 관계

부모 테이블의 기본 키를 내려받아서 자식 테이블의 기본 키 + 외래 키로 사용하는 관계

#### 비식별 관계

부모 테이블의 기본 키를 받아서 자식 테이블의 외래 키로만 사용하는 관계

* 필수적 비식별 관계 : 외래 키에 null을 허용하지 않는다. 연관관계는 필수이다.
* 선택적 비식별 관계 : 외래 키에 null을 허용한다. 연관관계는 선택이다.

### 비식별 관계 매핑

JPA에서 식별자를 둘 이상 사용하려면 별도의 식별자 클래스를 만들어야 한다. `JPA는 식별자를 구분하기 위해 equals와 hashCode를 사용해서 동등성 비교를 한다.` 식별자 필드가 하나일 때는 문제가 없지만, 2개 이상이면 별도의 식별자 클래스를 만들고 그곳에서 equals와 hashCode를 구현해야 한다. 식별자 클래스는 보통 equals와 hashCode를 구현할 때 모든 필드를 사용한다.   
JPA는 복합 키를 지원하기 위해 @IdClass\(관계형 데이터베이스\)와 @EmbeddedId\(객체지향\)를 제공한다.

#### @IdClass

```java
public class ParentId implements Serializable {
    
    private String id1;
    private String id2;
    
    public ParentId() {}
    
    public ParentId(String id1, String id2) {
        this.id1 = id1;
        this.id2 = id2;
    }
    
    @Override
    public boolean equals(Object o) {...}
    
    @Override
    public boolean hashCode() {...}
}

@Entity
@IdClass(ParendId.class)
public class Parent {

    @Id
    @Column(name = "PAREN_ID1")
    private String id1;
    
    @Id
    @Column(name = "PAREN_ID2")
    private String id2;
    
    private String name;
    ...
}

@Entity
public class Child {
    
    @Id
    private String id;


    /*
     * joinColumn의 name속성과 referencedColumnName 속성의 값이 같으면,
     * referencedColumnName은 생략해도 된다.
     */
    @ManyToOne
    @JoinColumns({
        @JoinColumn(name = "PARENT_ID1", referencedColumnName = "PARENT_ID1"),
        @JoinColumn(name = "PARENT_ID2", referencedColumnName = "PARENT_ID2")
    })
    private Parent parent;
}
```

@IdClass를 사용할 때 식별자 클래스는 다음 조건을 만족해야 한다.

* 식별자 클래스의 속성명과 엔티티에서 사용하는 식별자의 속성명이 같아야 한다.
* Serializable 을 구현해야 한다.
* equals, hashCode를 구현해야 한다.
* 기본 생성자가 있어야 한다.
* 식별자 클래스는 public이어야 한다.

#### @EmbeddedId

```java
@Embeddable
public class ParentId implements Serializable {

    @Column(name = "PARENT_ID1")
    private String id1;
    
    @Column(name = "PARENT_ID2")
    private String id2;
    
    // equals and hashCode 구현
    ...
}

@Entity
public class Parent {

    @EmbeddedId
    private ParentId id;
    
    private String name;
    ...
}
```

@EmbeddedId를 사용할 때 식별자 클래스는 다음 조건을 만족해야 한다.

* @Embeddable 어노테이션을 붙여주어야 한다.
* Serializable 을 구현해야 한다.
* equals, hashCode를 구현해야 한다.
* 기본 생성자가 있어야 한다.
* 식별자 클래스는 public이어야 한다.

{% hint style="info" %}
복합 키에는 @GenerateValue를 사용할 수 없다. 복합 키를 구성하는 여러 컬럼 중 하나에도 사용할 수 없다.
{% endhint %}

### 식별 관계 매핑

#### @IdClass와 식별 관계

```java
public class ChildId implements Serializable {

    private String parent;
    private String childId;
    
    
    // equals, hashCode
    ...
}


public class GrandChildId implements Serializable {

    private ChildId child;
    private String id;
    
    
    // equals, hashCode
    ...
}

@Entity
public class Parent {

    @Id @Column(name = "PARENT_ID")
    private String id;
    private String name;
    ...
}

@Entity
@IdClass(ChildId.class)
public class Child {

    @Id
    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    public Parent parent;
    
    @Id @Column(name = "CHILD_ID")
    private String childId;
    
    private String name;
    ...
}

@Entity
@IdClass(GrandChildId.class)
public class GrandChild {

    @Id
    @ManyToOne
    @JoinColumns({
        @JoinColumn(name = "PARENT_ID"),
        @JoinColumn(name = "CHILD_ID")
    })
    private Child child;
    
    @Id @Column(name = "GRANDCHILD_ID")
    private String id;
    
    private String name;
    ...
}

```

#### @EmbeddedId와 식별 관계

```java
@Embeddable
public class ChildId implements Serializable {

    private String parent;
    
    @Column(name = "CHILD_ID")
    priavte String id;
    
    
    // equals, hashCode
    ...
}

@Embeddable
public class GrandChildId implements Serializable {

    private ChildId childId;
    
    @Column(name = "GRANDCHILD_ID")
    private String id;
    
    
    // equals, hashCode
    ...
}

@Entity
public class Parent {

    @Id @Column(name = "PARENT_ID")
    private String id;
    private String name;
    ...
}

@Entity
public class Child {

    @EmbeddedId
    private ChildId id;

    /*
     * @MapsId는 외래키와 매핑한 연관관계를 기본 키에도 매핑하겠다는 어노테이션
     * 속성 값은 @EmbeddedId를 사용한 식별자 클래스의 기본 키 필드를 지정
     */
    @MapsId("parentId")
    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    public Parent parent;
    
    private String name;
    ...
}

@Entity
public class GrandChild {

    @EmbeddedId
    private GrandChildId id;

    @MapsId("childId")
    @ManyToOne
    @JoinColumns({
        @JoinColumn(name = "PARENT_ID"),
        @JoinColumn(name = "CHILD_ID")
    })
    private Child child;
    
    private String name;
    ...
}

```

### 식별, 비식별 관계의 장단점

#### 식별 관계의 단점 \(비식별 관계의 장점\)

{% tabs %}
{% tab title="데이터베이스 관점" %}
* 식별 관계는 부모 테이블의 기본 키를 자식 테이블로 전파하면서 자식 테이블의 기본 키 컬럼이 점점 늘어난다.  결국 조인할 때 SQL이 복잡해지고 기본 키 인덱스가 불필요하게 커질 수 있다.
* 식별 관계는 2개 이상의 컬럼을 합해서 복합 기본 키를 만들어야 하는 경우가 많다.
* 식별 관계를 사용할 때 기본 키로 비즈니스 의미가 있는 자연 키 컬럼을 조합하는 경우가 많다. 비즈니스 요구사항은 시간이 지남에 따라 언젠가는 변하는데, 자연 키 컬럼들이 자식, 손자까지 전파되면 변경하기 힘들다.  그에 반해, 비식별 관계의 기본 키는 비즈니스와 관계없는 대리 키를 주로 사용한다.
* 식별 관계는 부모 테이블의 기본 키를 자식 테이블의 기본 키로 사용하므로 테이블 구조가 유연하지 못하다.
{% endtab %}

{% tab title="객체지향 관점" %}
* 일대일 관계를 제외하고 식별 관계는 2개 이상의 컬럼을 묶은 복합 기본 키를 사용한다. 컬럼이 하나인 기본 키를 매핑하는 것보다 많은 노력이 필요하다.
* 비식별 관계의 기본 키는 주로 대리 키를 사용하는데 JPA는 @GenerateValue처럼 대리 키를 생성하기 위한 편리한 방법을 제공한다.
{% endtab %}
{% endtabs %}

#### 식별 관계의 장점

* 기본 키 인덱스를 활용하기 좋다.
* 상위 테이블들의 기본 키 컬럼을 자식, 손자 테이블들이 가지고 있으므로 특정 상황에 조인 없이 하위 테이블만으로 검색을 완료할 수 있다.

{% hint style="success" %}
비식별 관계를 사용하고 기본 키는 Long 타입의 대리 키를 사용하는 것을 추천한다.

* 대리키는 비즈니스와 아무 관련이 없고, JPA에서 @GenerateValue를 통해 간편한 생성을 제공한다.
* Long타입은 Integer보다 크므로 안전하다.
* 필수적 비식별 관계를 사용하는 것이 선택적 비식별 관계보다 좋다.
* 필수적 관계는 not null로 항상 관계가 있다는 것을 보장하므로 내부 조인만 사용해도 된다. 선택적 관계는 null을 허용하므로, 외부 조인\(outer join\)을 사용해야 한다.
{% endhint %}

## 조인 테이블

연관관계를 관리하는 조인 테이블을 추가하고 연관된 테이블의 외래 키를 가지고 연관관계를 관리한다.  
조인 테이블의 가장 큰 단점은 테이블을 하나 추가해야 한다는 점이다. 따라서 관리해야 하는 테이블이 늘어나고 추가로 조인해야 한다. 따라서 기본은 조인 컬럼을 사용하고 필요시에 조인 테이블을 사용하는 것이 좋다.

### 일대일 조인 테이블

```java
@Entity
public class Parent {

    @Id @GeneratedValue
    @Column(name = "PARENT_ID")
    private Long id;
    
    private String name;
    
    @OneToOne
    @JoinTable(name = "PARENT_CHILD",
        joinColumns = @JoinColumn(name = "PARENT_ID"),
        inverseJoinColumns = @JoinColumn(name = "CHILD_ID"))
    private Child child;
    ...
}

@Entity
public class Child {

    @Id @GeneratedValue
    @Column(name = "CHILD_ID")
    private Long id;
    
    private String name;
    
    /* 
     * 양방향 매핑시 추가
     */
    @OneToOne(mappedBy = "child")
    private Parent parent;
    ...
}
```

### 일대다 조인 테이블

```java
@Entity
public class Parent {

    @Id @GeneratedValue
    @Column(name = "PARENT_ID")
    private Long id;
    
    private String name;
    
    @OneToMany
    @JoinTable(name = "PARENT_CHILD",
        joinColumns = @JoinColumn(name = "PARENT_ID"),
        inverseJoinColumns = @JoinColumn(name = "CHILD_ID"))
    private List<Child> children = new ArrayList<Child>();
    ...
}

@Entity
public class Child {

    @Id @GeneratedValue
    @Column(name = "CHILD_ID")
    private Long id;
    
    private String name;
    ...
}
```

### 다대일 조인 테이블

```java
// 일대다에서 방향만 반대이므로 일대다와 구조는 같다.
@Entity
public class Parent {

    @Id @GeneratedValue
    @Column(name = "PARENT_ID")
    private Long id;
    
    private String name;
    
    // 양방향 매
    @OneToMany(mappedBy = "parent")
    private List<Child> children = new ArrayList<Child>();
    ...
}

@Entity
public class Child {

    @Id @GeneratedValue
    @Column(name = "CHILD_ID")
    private Long id;
    
    private String name;
    
    @ManyToOne(optional = false)
    @JoinTable(name = "PARENT_CHILD",
        joinColumns = @JoinColumn(name = "CHILD_ID"),
        inverseJoinColumns = @JoinColumn(name = "PARENT_ID"))
    private Parent parent;
    ...
}
```

### 다대다 조인 테이블

```java
@Entity
public class Parent {

    @Id @GeneratedValue
    @Column(name = "PARENT_ID")
    private Long id;
    
    private String name;
    
    @ManyToMany
    @JoinTable(name = "PARENT_CHILD",
        joinColumns = @JoinColumn(name = "PARENT_ID"),
        inverseJoinColumns = @JoinColumn(name = "CHILD_ID"))
    private List<Child> children = new ArrayList<Child>();
    ...
}

@Entity
public class Child {

    @Id @GeneratedValue
    @Column(name = "CHILD_ID")
    private Long id;
    
    private String name;    
    ...
}
```

## @SecondaryTable

@SecondaryTable을 사용하면 한 엔티티에 여러 테이블을 매핑할 수 있다. `하지만 테이블당 엔티티를 각각 만들어서 일대일 매핑하는 것을 권장한다.`  
이 방법은 항상 두 테이블을 조회하므로 최적화하기 어렵다. 반면에 일대일 매핑은 원하는 부분만 조회할 수 있고 필요하면 둘을 함께 조회하면 된다.

