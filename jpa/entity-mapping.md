---
description: 자바 ORM 표준 JPA 프로그래밍
---

# 4장. 엔티티 매핑

## @Entity

| 속성 | 기 | 기본 |
| :--- | :--- | :--- |
| name | JPA에서 사용할 엔티티 이름을 지정 | 설정하지 않으면 클래스 이름을 그대로 사용 |

{% hint style="danger" %}
### 주의 사항

* 기본 생성자는 필수 \(public or protected\)
* final 클래스, enum, interface, inner 클래스에는 사용 불가
* 저장할 필드에 final 사용 금
{% endhint %}

## @Table

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#xC18D;&#xC131;</th>
      <th style="text-align:left">&#xAE30;</th>
      <th style="text-align:left">&#xAE30;&#xBCF8;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">name</td>
      <td style="text-align:left">&#xB9E4;&#xD551;&#xD560; &#xD14C;&#xC774;&#xBE14; &#xC774;&#xB984;</td>
      <td
      style="text-align:left">&#xC5D4;&#xD2F0;&#xD2F0; &#xC774;&#xB984;&#xC744; &#xC0AC;&#xC6A9;</td>
    </tr>
    <tr>
      <td style="text-align:left">catalog</td>
      <td style="text-align:left">catalog &#xAE30;&#xB2A5;&#xC774; &#xC788;&#xB294; &#xB370;&#xC774;&#xD130;&#xBCA0;&#xC774;&#xC2A4;&#xC5D0;&#xC11C;
        catalog&#xB97C; &#xB9E4;&#xD551;</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">schema</td>
      <td style="text-align:left">schema &#xAE30;&#xB2A5;&#xC774; &#xC788;&#xB294; &#xB370;&#xC774;&#xD130;&#xBCA0;&#xC774;&#xC2A4;&#xC5D0;&#xC11C;
        schema&#xB97C; &#xB9E4;&#xD551;</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>uniqueConstraints</p>
        <p>(DDL)</p>
      </td>
      <td style="text-align:left">DDL &#xC0DD;&#xC131; &#xC2DC;&#xC5D0; &#xC720;&#xB2C8;&#xD06C; &#xC81C;&#xC57D;&#xC870;&#xAC74;&#xC744;
        &#xB9CC;&#xB4E6;</td>
      <td style="text-align:left"></td>
    </tr>
  </tbody>
</table>## 데이터베이스 스키마 자동 생성

{% tabs %}
{% tab title="JPA" %}
```markup
<!-- JPA -->
<!-- 애플리케이션 실행 시점에 데이터베이스 테이블을 자동 생성한다 -->
<!-- create, create-drop, update, validate, none -->
<property name="hibernate.hbm2ddl.auto" value="create">
<!-- 콘솔에 실행되는 쿼리를 출력한다 -->
<property name="hibernate.show_sql" value="true">
```
{% endtab %}

{% tab title="Spring Data JPA" %}
```yaml
# Spring Data JPA
# https://docs.spring.io/spring-boot/docs/1.0.x/reference/html/howto-database-initialization.html
# (boolean) true, false
spring.jpa.generate-ddl: true
# (enum) none, validate, update, create-drop
spring.jpa.hibernate.ddl-auto: create-drop
spring.jpa.show-sql: true
```
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
### 주의 사항

* 운영 서버에서는 create, create-drop, update처럼 테이블을 수정하는 옵션은 절대 사용하면 안됨
* 개발 초기 단계, 자동화된 개발자 테스트 환경, CI 서버는 create, create-drop 추천
* 테스트 서버는 update, validate 추천
* 스테이징과 운영 서버는 validate, none 추천
{% endhint %}

## 기본 키 매핑

### 직접 할당

* 기본 키를 애플리케이션에서 직접 할당한다
* em.persist\(\) 를 호출하기 전에 애플리케이션에서 직접 식별자 값을 할당해야 한다. 만약 식별자 값이 없으면 예외 발생
* 적용 가능 자바 타입
  * 자바 기본형
  * 자바 Wrapper
  * String
  * java.util.Date
  * java.sql.Date
  * java.math.BigDecimal
  * java.math.BigInteger

```java
@Id
@Column(name = "id")
private String id;
```

### IDENTITY

* 기본 키 생성을 데이터베이스에 위임한다
* MySQL\(AUTO\_INCREMENT\), PostgreSQL, SQL Server, DB2 등에서 사용
* 데이터베이스에 값을 저장하고 나서야 기본 키 값을 구할 수 있을 때 사용한다
* 기본 키 값을 얻어오기 위해 데이터베이스에 insert한 후에 추가로 조회를 한다
* 데이터베이스에 엔티티를 저장해서 식별자 값을 획득한 후 영속성 컨텍스트에 저장한

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

{% hint style="info" %}
하이버네이트는 IDENTITY 전략에서 추가로 조회하는 부분을 최적화하기 위해 Statement.getGeneratedKEys\(\)를 사용하여 데이터베이스와 한 번만 통신한다. \(데이터 저장과 동시에 조회\)
{% endhint %}

### SEQUENCE

* 데이터베이스 시퀀스를 사용해서 기본 키를 할당한다 \(자동할당\)
* 데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트이다
* 오라클, PostgreSQL, DB2, H2 등에서 사용
* em.persist\(\) 를 호출할 때 먼저 데이터베이스 시퀀스를 사용해서 식별자를 조회한다.  그 후 식별자를 엔티티에 할당한 후에 엔티티를 영속성 컨텍스트에 저장한다.

```sql
CREATE SEQUENCE BOARD_SEQ START WITH 1 INCREMENT BY 1;
```

```java
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE,
                generator = "BOARD_SEQ_GENERATOR")
private Long id;
```

{% hint style="info" %}
JPA는 시퀀스에 접근하는 횟수를 줄이기 위해 @SequenceGenerator.allocationSize를 50으로 기본값으로 사용한다.

이 최적화 방법은 시퀀스 값을 선점하므로 여러 JVM이 동시에 동작해도 기본 키 값이 충돌하지 않는 장점이 있지만, 시퀀스 값이 한번에 많이 증가한다는 단점이 있다.
{% endhint %}

### TABLE

* 키 생성 테이블을 사용한다
* 이 전략은 테이블을 사용하므로 모든 데이터베이스에 적용할 수 있다
* SEQUENCE 전략과 내부 동작방식이 같

```sql
create table MY_SEQUENCES (
    sequence_name varchar(255) not null,
    next_val bigint,
    primary key ( sequence_name)
)
```

```java
@Entity
@TableGenerator(
    name = "BOARD_SEQ_GENERATOR",
    table = "MY_SEQUENCES",
    pkColumnValue = "BOARD_SEQ"
    allocationSize = 1)
public class Board {
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE,
                generator = "BOARD_SEQ_GENERATOR")
    private Long id;
}
```

### AUTO

* AUTO는 선택한 데이터베이스 방언에 따라 IDENTITY, SEQUENCE, TABLE 중 하나를 자동으로 선택한





## 참고

{% page-ref page="entity-mapping.md" %}

[https://www.popit.kr/%ED%95%98%EC%9D%B4%EB%B2%84%EB%84%A4%EC%9D%B4%ED%8A%B8%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%9E%90%EB%8F%99-%ED%82%A4-%EC%83%9D%EC%84%B1-%EC%A0%84%EB%9E%B5%EC%9D%84-%EA%B2%B0%EC%A0%95%ED%95%98/](https://www.popit.kr/%ED%95%98%EC%9D%B4%EB%B2%84%EB%84%A4%EC%9D%B4%ED%8A%B8%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%9E%90%EB%8F%99-%ED%82%A4-%EC%83%9D%EC%84%B1-%EC%A0%84%EB%9E%B5%EC%9D%84-%EA%B2%B0%EC%A0%95%ED%95%98/)





