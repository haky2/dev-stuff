---
description: 자바 ORM 표준 JPA 프로그래밍
---

# 6장. 다양한 연관관계 매핑

## 다대일 \[N:1\]

데이터베이스 테이블의 관계에서 외래 키는 항상 다쪽에 있다.  
따라서, 객체 양방향 관계에서 연관관계의 주인은 항상 다쪽이다.

### 다대일 단방향

```java
@Entity
public class Member {
    
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    
    private String username;
    
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    
    //Getter, Setter ...
    ...
}
```

```java
@Entity
public class Team {
    
    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
    
    private String name;
    
    //Getter, Setter ...
    ...
}
```

### 다대일 양방향

```java
@Entity
public class Member {
    
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    
    private String username;
    
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    // 연관관계 편의 메소드
    public void setTeam(Team team) {
        this.team = team;
        if (!team.getMembers().contains(this)) {
            team.getMembers().add(this);
        }
    }
}
```

```java
@Entity
public class Team {
    
    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
    
    private String name;
    
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<Members>();
    
    // 연관관계 편의 메소
    public void addMember(Member member) {
        this.members.add(member);
        if (member.getTeam() != this) {
            member.setTeam(this);
        }
    }
}
```

#### 양방향은 외래 키가 있는 쪽이 연관관계의 주인이다

일대다와 다대일 연관관계는 항상 다\(N\)에 외래 키가 있다. JPA는 외래 키를 관리할 때 연관관계의 주인만 사용한다. 주인이 아닌 필드는 조회를 위한 JPQL이나 객체 그래프를 탐색할 때 사용한다.

#### 양방향 연관관계는 항상 서로를 참조해야 한다

항상 서로 참조하게 하려면 연관관계 편의 메소드를 작성하는 것이 좋다. 연관관계 편의 메소드를 양쪽에 다 작성하면 무한루프에 빠지므로 주의해야 한다.

## 일대다 \[1:N\]

### 일대다 단방향

```java
@Entity
public class Team {
    
    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
    
    private String name;
    
    // JoinColumn을 명시하지 않으면 JoinTable이 기본으로 사용된다
    @OneToMany
    @JoinColumn(name = "TEAM_ID")
    private List<Member> members = new ArrayList<Members>();
    
    //Getter, Setter ...
    ...
}
```

```java
@Entity
public class Member {
    
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    
    private String username;
    
    //Getter, Setter ...
    ...
}
```

#### 일대다 단방향 매핑의 단점

일대다 단방향 매핑의 단점은 매핑한 객체가 관리하는 외래 키가 다른 테이블에 있다는 점이다. 다른 테이블에 외래 키가 있으면 연관관계 처리를 위한 update sql을 추가로 실행해야 한다.

#### 일대다 단방향 매핑보다는 다대일 양방향 매핑을 사용하자

일대다 단방향 매핑을 사용하면 엔티티를 매핑한 테이블이 아닌 다른 테이블의 외래 키를 관리해야 한다. 이것은 성능 문제도 있지만 관리도 부담스러우므로 다대일 양방향 매핑을 권장한다.

### 일대다 양방향

```java
@Entity
public class Team {
    
    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
    
    private String name;

    @OneToMany
    @JoinColumn(name = "TEAM_ID")
    private List<Member> members = new ArrayList<Members>();
    
    //Getter, Setter ...
    ...
}
```

```java
@Entity
public class Member {
    
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    
    private String username;
    
    // 두 객체에서 같은 키를 관리하므로 다대일 쪽은 읽기 전용으로 설정했
    @ManyToOne
    @JoinColumn(name = "TEAM_ID", insertable = false, updatable = false)
    private Team team;
    
    //Getter, Setter ...
    ...
}
```

#### 양방향 매핑에서 @OneToMany는 연관관계의 주인이 될 수 없다

일대다 양방향 매핑은 존재하지 않는다. 관계형 데이터베이스의 특성상 일대다, 다대일 관계는 항상 다 쪽에 외래 키가 있다. 따라서 @OneToMany, @ManyToOne 둘 중에 연관관계의 주인은 항상 다 쪽인 @ManyToOne을 사용한 곳이다. 이런 이유로 `@ManyToOne에는 mappedBy 속성이 없다.`   
일대다 양방향 매핑이 가능하게 하려면, 일대다 단방향 매핑 반대편에 같은 외래 키를 사용하는 다대일 단방향 매핑을 읽기 전용으로 추가하면 된다.

## 일대일 \[1:1\]

### 주 테이블에 외래 키

```java
// 주 테이블에 외래키 - 단방향

@Entity
public class Member {
    
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    
    private String username;
    
    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;
    
    //Getter, Setter ...
    ...
}

@Entity
public class Locker {

    @Id
    @GeneratedValue
    @Column(name = "LOCKER_ID")
    private Long id;
    
    private String name;
    
    //Getter, Setter ...
    ...
}
```

```java
// 주 테이블에 외래키 - 양방향

@Entity
public class Member {
    
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    
    private String username;
    
    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;
    
    //Getter, Setter ...
    ...
}

@Entity
public class Locker {

    @Id
    @GeneratedValue
    @Column(name = "LOCKER_ID")
    private Long id;
    
    private String name;
    
    @OneToOne(mappedBy = "locker")
    private Membber membber;
    
    //Getter, Setter ...
    ...
}
```

#### 주 테이블에 외래 키

외래 키를 객체 참조와 비슷하게 사용할 수 있어서 객체지향 개발자들이 선호한다. 이 방법의 장점은 주 테이블이 외래 키를 가지고 있으므로 주 테이블만 확인해도 대상 테이블과 연관관계가 있는지 알 수 있다.

### 대상 테이블에 외래 키

```java
// 대상 테이블에 외래키 - 방향

@Entity
public class Member {
    
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    
    private String username;
    
    @OneToOne(mappedBy = "member")
    private Locker locker;
    
    //Getter, Setter ...
    ...
}

@Entity
public class Locker {

    @Id
    @GeneratedValue
    @Column(name = "LOCKER_ID")
    private Long id;
    
    private String name;
    
    @OneToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;
    
    //Getter, Setter ...
    ...
}
```

#### 대상 테이블에 외래 키

전통적인 데이터베이스 개발자들은 보통 대상 테이블에 외래 키를 두는 것을 선호한다. 이 방법의 장점은 테이블 관계를 일대일에서 일대다로 변경할 때 테이블 구조를 그대로 유지할 수 있다. 

{% hint style="danger" %}
프록시를 사용할 때 외래 키를 직접 관리하지 않는 일대일 관계는 지연 로딩으로 설정해도 즉시 로딩된다. Locker.member는 지연 로딩할 수 있지만, Member.locekr는 지연 로딩으로 설정해도 즉시 로딩된다.  
프록시의 한계 때문에 발생하는 문제인데 프록시 대신에 bytecode instrumentation을 사용하면 해결할 수 있다.
{% endhint %}

## 다대다 \[N:N\]

### 다대다 단방향

```java
@Entity
public class Member {
    
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    
    private String username;
    
    @ManyToOne
    @JoinTable(name = "MEMBER_PRODUCT",
               joinColumns = @JoinColumn(name = "MEMBER_ID"),
               inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID")) 
    private List<Product> products = new ArrayList<Product>();
    
    //Getter, Setter ...
    ...
}
```

```java
@Entity
public class Product {

    @Id
    @GeneratedValue
    @Column(name = "PRODUCT_ID")
    private String id;
    
    private String name;
    
    //Getter, Setter ...
    ...
}
```

### 다대다 양방향

```java
@Entity
public class Member {
    
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    
    private String username;
    
    @ManyToOne
    @JoinTable(name = "MEMBER_PRODUCT",
               joinColumns = @JoinColumn(name = "MEMBER_ID"),
               inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID")) 
    private List<Product> products = new ArrayList<Product>();
    
    // 연관관계 편의 메소드
    public void addProduct(Product product) {
        this.products.add(product);
        if (!product.getMembers().contains(this)) {
            product.getMembers().add(this);
        }
    }
}
```

```java
@Entity
public class Product {

    @Id
    @GeneratedValue
    @Column(name = "PRODUCT_ID")
    private String id;
    
    private String name;
    
    @ManyToMany(mappedBy = "products")
    private List<Member> members;
    
    // 연관관계 편의 메소드
    public void addMember(Member member) {
        this.members.add(member);
        if (!member.getProducts().contains(this)) {
            member.getProducts().add(this);
        }
    }
}
```

#### 다대다 매핑의 한계와 극복, 연결 엔티티 사용

@ManyToMany를 실무에서 사용하기에는 한계가 있다. 보통 연결 테이블에 키 컬럼 외의 추가 컬럼이 더 필요하다. 하지만 컬럼을 추가하면 더는 @ManyToMany를 사용할 수 없다. 왜냐하면 추가한 컬럼들을 매핑할 수 없기 때문이다. 결국 `연결 테이블을 매핑하는 연결 엔티티를 만들고` 이곳에 추가한 컬럼들을 매핑해야 한다. 그리고 엔티티 간의 관계도 `다대다에서 일대다, 다대일 관계로` 풀어야 한다.

{% hint style="info" %}
#### 복합 기본 키

JPA에서 복합 키를 사용하려면 별도의 식별자 클래스를 만들어야 한다. 그리고 엔티티에 @IdClass를 사용해서 식별자 클래스를 지정하면 된다.  
복합 키를 위한 식별자 클래스는 다음과 같은 특징이 있다.

* 복합 키는 별도의 식별자 클래스로 만들어야 한다.
* Serializable을 구현해야 한다.
* equals와 hasCode 메소드를 구현해야 한다.
* 기본 생성자가 있어야 한다.
* 식별자 클래스는 public 이어야 한다.
* @IdClass를 사용하는 방법 외에 @EmbeddedId를 사용하는 방법도 있다.
{% endhint %}

{% hint style="info" %}
#### 식별 관계 \(Identifying Relationship\)

부모 테이블의 기본 키를 받아서 자신의 기본 키 + 외래 키로 사용하는 것을 데이터베이스 용어로 식별 관계라 한다.
{% endhint %}

#### 다대다 매핑 시, 새로운 기본 키 사용

다대다 매핑 시 추천하는 기본 키 생성 전략은 데이터베이스에서 자동으로 생성해주는 대리 키를 Long 값으로 사용\(`비식별 관계`\)하는 것이다. 이것의 장점은 간편하고 거의 영구히 쓸 수 있으며 비즈니스에 의존하지 않는다. 그리고 ORM 매핑 시에 복합 키를 만들지 않아도 되므로 간단히 매핑을 완성할 수 있다.

