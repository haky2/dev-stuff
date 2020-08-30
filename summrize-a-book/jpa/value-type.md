---
description: 자바 ORM 표준 JPA 프로그래밍
---

# 9장. 값 타입

## 기본값 타입

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;
    
    // 기본 값 타입
    private String name;
    private int age;
    ...
}
```

## 임베디드 타입

새로운 값 타입을 직접 정의해서 사용할 수 있는데, JPA에서는 이것을 임베디드 타입이라 한다.  임베디드 타입도 int, String 처럼 값 타입이다. 임베디드 타입은 `기본 생성자가 필수`다.

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;
    private String name;
    
    // 값 타입을 사용하는 곳에 표시 (@Embedded, @Embeddable 둘 중 하나 생략 가능)
    @Embedded
    private Period workPerido;
    
    @Embedded
    private Address homeAddress;
}

// 값 타입을 정의하는 곳에 표시
@Embeddable
public class Period {

    @Temporal(TemporalType.DATE)
    private Date startDate;
    
    @Temporal(TemporalType.DATE)
    private Date endDate;
    
    public boolean isWork(Date date) {
        // ...
    }
}

@Embeddable
public class Address {

    @Column(name = "city")
    private String city;
    private String street;
    private String zipcode;
    ...
}
```

### 임베디드 타입과 테이블 매핑

임베디드 타입은 엔티티의 값일 뿐이다. 따라서 값이 속한 엔티티의 테이블에 매핑한다. 잘 설계한 ORM 애플리케이션은 매핑한 테이블의 수보다 클래스의 수가 더 많다.

### @AttributeOverride

임베디드 타입에 정의한 매핑정보를 재정의하려면 엔티티에 @AttributeOverride를 사용하면 된다.

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;
    private String name;
    
    @Embedded
    Address homeAddress;
    
    @Embedded
    @AttributeOverrides({
        @AttributeOverride(name = "city", column = @Column(name = "COMPANY_CITY")),
        @AttributeOverride(name = "street", column = @Column(name = "COMPANY_STREET")),
        @AttributeOverride(name = "zipcode", column = @Column(name = "COMPANY_ZIPCODE"))
    })
    Address companyAddress;
}
```

{% hint style="info" %}
@AttributeOverrides는 엔티티에 설정해야 한다.   
임베디드 타입이 임베디드 타입을 가지고 있어도 엔티티에 설정해야 한다.
{% endhint %}

### 임베디드 타입과 null

임베디드 타입이 null이면 매핑한 컬럼 값은 모두 null이 된다.

## 값 타입과 불변 객체

### 값 타입 공유 참조

```java
member1.setHomeAddress(new Address("OldCity"));
Address address = member1.getHomeAddress();

// 회원1의 address 값을 공유해서 사용하여 회원1의 주소도 변경됨
address.setCity("NewCity");
member2.setHomeAddress(address);
```

### 값 타입 복사

```java
member1.setHomeAddress(new Address("OldCity"));
Address address = member1.getHomeAddress();

// 회원1의 address 값을 복사해서 새로운 newAddress 값을 생성
Address newAddress = address.clone();

// 값을 복사해서 사용하면 공유 참조로 인해 발생하는 부작용을 피할 수 있다.
newAddress.setCity("NewCity");
member2.setHomeAddress(newAddress);
```

{% hint style="danger" %}
#### 객체의 공유 참조는 피할 수 없다

근본적인 해결책이 필요한데 가장 단순한 방법은 객체의 값을 수정하지 못하게 막으면 된다.   
setter 같은 수정자 메소드를 모두 제거하자.
{% endhint %}

### 불변 객체

객체를 불변하게 만들면 값을 수정할 수 없으므로 부작용을 원천 차단할 수 있다. 따라서 값 타입은 될 수 있으면 불변 객체로 설계해야 한다.

```java
@Embeddable
public class Address {

    private String city;
    
    //JPA에서 기본 생성자는 필
    protected Address() {}
    
    // 생성자로 초기 값을 설정한다
    public Address(String city) {
        this.city = city;
    }
    
    // 접근자(getter)는 노출한다
    public String getCity() {
        return city;
    }
    
    // 수정자(setter)는 만들지 않는다
}
```

## 값 타입의 비교

* 동일성 비교 : 인스턴스의 참조 값을 비교 \(==\)
* 동등성 비교 : 인스턴스의 값을 비교 \(equals\(\)\)

## 값 타입 컬렉션

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;
    
    @Embedded
    private Address homeAddress;
    
    @ElementCollection
    @CollectionTable(name = "FAVORITE_FOODS",
        joinColumns = @JoinColumn(name = "MEMBER_ID"))
    @Column(name = "FOOD_NAME")
    private Set<String> favoriteFoods = new HashSet<String>();
    
    @ElementCollection
    @CollectionTable(name = "ADDRESS",
        joinColumns = @JoinColumn(name = "MEMBER_ID"))
    private List<Address> addressHistory = new ARrayList<Address>();
    
    ...
}

@Embeddable
public class Address {

    @Column
    private String city;
    private String street;
    private String zipcode;
    ...
}
```

{% hint style="info" %}
#### @CollectionTable을 생략하면 기본값을 사용해서 매핑한다.

기본값 : \(엔티티 이름\)\_\(컬렉션 속성 이름\)  
ex. Member\_addressHistory
{% endhint %}

{% hint style="info" %}
값 타입 컬렉션은 영속성 전이\(cascade\) + 고아 객체 제거\(orphan remove\) 기능을 필수로 가진다고 볼 수 있다.
{% endhint %}

### 값 타입 컬렉션의 제약사항

값 타입 컬렉션에 보관된 값 타입들은 별도의 테이블에 보관된다. 여기에 보관된 값 타입의 값이 변경되면 데이터베이스에 있는 원본 데이터를 찾기 어렵다는 문제가 있다.  
이런 문제로 인해 JPA 구현체들은 값 타입 컬렉션에 변경사항이 발생하면, 값 타입 컬렉션이 매핑된 테이블의 연관된 모든 데이터를 삭제하고, 현재 값 타입 컬렉션 객체에 있는 모든 값을 데이터베이스에 다시 저장한다.  
실무에서는 값 타입 컬렉션이 매핑된 테이블에 데이터가 많다면 값 타입 컬렉션 대신에 일대다 관계를 고려해야 한다.

추가로 값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어서 기본 키를 구성해야 한다. 데이터베이스 기본 키 제약 조건으로 인해 컬럼에 null을 입력할 수 없고, 같은 값을 중복해서 저장할 수 없는 제약도 있다.

이러한 문제를 해결하려면 값 타입 컬렉션을 사용하는 대신에 새로운 엔티티를 만들어서 일대다 관계로 설정하면 된다. 여기에 추가로 cascade + orphan remove 를 적용하면 값 타입 컬렉션처럼 사용할 수 있다. 

