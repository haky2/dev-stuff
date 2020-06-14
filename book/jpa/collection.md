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

