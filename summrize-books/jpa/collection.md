---
description: 자바 ORM 표준 JPA 프로그래밍
---

# 14장. 컬렉션과 부가 기능

## 컬렉션

JPA 명세에는 자바 컬렉션 인터페이스에 대한 특별한 언급이 없다. 하위 내용은 하이버네이트 구현체를 기준으로 설명.

### JPA와 컬렉션

하이버네이트는 컬렉션을 효율적으로 관리하기 위해 엔티티를 영속 상태로 만들 때 원본 컬렉션을 감싸고 있는 내장 컬렉션을 생성해서 이 내장 컬렉션을 사용하도록 참조를 변경한다. 하이버네이트는 이런 특징 때문에 컬렉션을 사용할 때 즉시 초기화해서 사용하는 것을 권장한다.

```java
Collection<Member> members = new ArrayList<Member>();
```

| 컬렉션 인터페이스 | 내장 컬렉션 | 중복 허용 | 순서 보관 |
| :--- | :--- | :--- | :--- |
| Collection, List | PersistenceBag | O | X |
| Set | PersistenceSet | X | X |
| List + @OrderColumn | PersistentList | O | O |

### Collection, List

Collection, List는 엔티티를 추가할 때 중복 비교를 하지 않고 단순 저장만 하므로, 엔티티를 추가해도 지연 로딩된 컬렉션을 초기화하지 않는다.

```java
@Entity
public class Parent {

    @Id @GeneratedValue
    private Long id;
    
    @OneToMany
    @JoinColumn
    private Collection<CollectionChild> collection =
        new ArrayList<CollectionChild>();
        
    @OneToMany
    @JoinColumn
    private List<ListChild> list = new ArrayList<ListChild>();
    ...
}
```

### Set

Set은 엔티티를 추가할 때 중복된 엔티티가 있는지 비교해야 하므로, 지연 로딩된 컬렉션을 초기화한다.

```java
@Entity
public class Parent {

    @OneToMany
    @JoinColumn
    private Set<SetChild> set = new HashSet<SetChild>();
    ...
}
```

### List + @OrderColumn

데이터베이스에 순서 값을 저장해서 조회할 때 사용한다. 실무에서 사용하기에는 단점\(SQL 추가 실행, null 관리 등\)이 많아서 직접 순서 값을 관리하거나 @OrderBy 사용을 권장한다.

### @OrderBy

```java
@Entity
public class Team {

    @Id @GeneratedValue
    private Long id;
    private String name;
    
    @OneToMany(mappedBy = "team")
    @OrderBy("username desc, id asc") // 엔티티의 필드를 대상으로 한다
    private Set<Member> members = new HashSet<Member>();
    ...
}
```

## @Converter

컨버터를 사용하면 엔티티의 데이터를 변환해서 데이터베이스에 저장할 수 있다.

```java
@Entity
public class Member {

    @Id
    private String id;
    private String username;
    
    @Convert(converter=BooleanToYNConverter.class)
    private boolean vip;
    
    ...
}

@Converter
public class BooleanToYNConverter implements AttributeConverter<Boolean, String> {

    @Override
    public String convertToDatabaseColumn(Boolean attribute) {
        return (attribute != null && attribute) ? "Y" : "N";
    }
    
    @Override
    public Boolean convertToEntityAttribute(String dbData) {
        return "Y".equals(dbData);
    }
}
```

## 리스너

### 이벤트 종류

| event | desc |
| :--- | :--- |
| PostLoad | 엔티티가 영속성 컨텍스트에 조회된 직후 또는 refresh를 호출한 후 \(2차 캐시에 저장되어 있어도 호출된다\) |
| PrePersist | persist\(\)를 호출해서 엔티티를 영속성 컨텍스트에 관리하기 직전에 호출된다. |
| PreUpdate | flush나 commit을 호출해서 엔티티를 데이터베이스에 수정하기 직전에 호출된다. |
| PreRemove | remove\(\)를 호출해서 엔티티를 영속성 컨텍스트에서 삭제하기 직전에 호출된다. |
| PostPersist | flush나 commit을 호출해서 엔티티를 데이터베이스에 저장한 직후에 호출된다. |
| PostUpdate | flush나 commit을 호출해서 엔티티를 데이터베이스에 수정한 직후에 호출된다. |
| PostRemove | flush나 commit을 호출해서 엔티티를 데이터베이스에 삭제한 직후에 호출된다. |

### 엔티티에 직접 적용

```java
@Entity
public class Duck {

    @Id @GeneratedValue
    public Long id;
    
    private String name;
    
    @PrePersist
    public void prePersist() {
        System.out.println("Duck.prePersist id = " + id);
    }
    
    @PostPersist
    public void postPersist() {
        System.out.println("DuckpostPersist id = " + id);
    }
    
    @PostLoad
    public void postLoad() {
        System.out.println("Duck.postLoad id = " + id);
    }
    
    @PreRemove
    public void preRemove() {
        System.out.println("Duck.preRemove id = " + id);
    }
    
    @PostRemove
    public void postRemove() {
        System.out.println("Duck.postRemove id = " + id);
    }
    
    ...
}
```

### 별도의 리스너 등록

```java
@Entity
@EntityListeners(DuckListener.class)
public class Duck {
    ...
}

public class DuckListener {

    @PrePersist
    // 특정 타입이 확실하면 특정 타입을 받을 수 있다.
    private void prePersist(Object obj) {
        System.out.println("prePersist obj = [" + obj + "]");
    }
    
    @PostPersist
    // 특정 타입이 확실하면 특정 타입을 받을 수 있다.
    private void postPersist(Object obj) {
        System.out.println("postPersist obj = [" + obj + "]");
    }
}
```

### 기본 리스너 사용

모든 엔티티의 이벤트를 처리하려면 META-INF/orm.xml에 기본\(default\) 리스너로 등록하면 된다.  
여러 리스너를 등록했을 때 이벤트 호출 순서는 다음과 같다.

1. 기본 리스너
2. 부모 클래스 리스너
3. 리스너
4. 엔티티

## 엔티티 그래프

엔티티 조회시점에 연관된 엔티티들을 함께 조회하는 기능이다.  
엔티티 그래프는 정적으로 정의하는 Named 엔티티 그래프와 동적으로 정의하는 엔티티 그래프가 있다.

### Named 엔티티 그래프

```java
// Named 엔티티 그래프와 subgraph
@NamedEntityGraph(name = "Order.withAll", attributeNodes = {
    @NamedAttributeNode("member"),
    @NamedAttributeNode(value = "orderItems", subgraph = "orderItems")
    },
    subgraphs = @NamedSubgraph(name = "orderItems", attributeNodes = {
    @NamedAttributeNode("item")
    })
)
@Entity
@Table(name = "ORDERS")
public class Order {

    @Id @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "MEMBER_ID")
    private Member member;
    
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> orderItems = new ArrayList<OrderItem>();
    
    ...
}
```

```java
// em.find()에서 엔티티 그래프 사용
EntityGraph graph = em.getEntityGraph("Order.withAll");

Map hints = new HashMap();
hints.put("javax.persistence.fetchgraph", graph);

Order order = em.find(Order.class, orderId, hints);

// JPQL에서 사용
List<Order> resultList = 
    em.createQuery("select o from ORder o where o.id = :orderId",
        Order.class)
        .setParameter("orderId", orderId)
        .setHint("javax.persistence.fetchgraph", em.getEntityGraph("Order.withAll"))
        .getResultList();
```

### 동적 엔티티 그래프

```java
EntityGraph<Order> graph = em.createEntityGraph(Order.class);
graph.addAttributeNodes("member");
Subgraph<OrderItem> orderItems = graph.addSubgraph("orderItems");
orderItems.addAttributeNodes("item");

Map hints = new HashMap();
hints.put("javax.persistence.fetchgraph", graph);

Order order = em.find(Order.class, orderId, hints);
```

