---
description: 자바 ORM 표준 JPA 프로그래밍
---

# 2장. JPA 시작

## 하이버네이트 <a id="&#xD558;&#xC774;&#xBC84;&#xB124;&#xC774;&#xD2B8;"></a>

### 라이브러리 <a id="&#xB77C;&#xC774;&#xBE0C;&#xB7EC;&#xB9AC;"></a>

* hibernate-core: 하이버네이트 라이브러리
* hibernate-entitymanager: 하이버네이트가 JPA 구현체로 동작하도록 JPA 표준을 구현한 라이브러리
* hibernate-jpa-{version}-api: JPA {version} 표준 API를 모아둔 라이브러리

### 속성 <a id="&#xC18D;&#xC131;"></a>

* javax.persistence로 시작하는 속성은 JPA 표준 속성으로 특정 구현체에 종속되지 않는다
* hibernate로 시작하는 속성은 하이버네이트 전용 속성

### 데이터베이스 방언 <a id="&#xB370;&#xC774;&#xD130;&#xBCA0;&#xC774;&#xC2A4;-&#xBC29;&#xC5B8;"></a>

* hibernate.dialect 속성
* JPA는 특정 데이터베이스에 종속적이지 않은 기술이므로 다른 데이터베이스로 쉽게 교체 가능
* 다만, 각 데이터베이스에서 제공하는 SQL 문법과 함수가 차이가 있음 \(데이터 타입, 함수명, 페이징 등\)
* Dialect\(방언\) : SQL표준을 지키지 않거나 특정 데이터베이스만의 고유한 기능을 JPA에서 일컫는 용어
* 개발자는 JPA 표준 문법을 사용하면, 특정 데이터베이스에 의존적인 SQL은 데이터베이스 방언이 처리해준다. 따라서 데이터베이스가 변경되어도 애플리케이션 코드를 변경할 필요 없이 데이터베이스 방언만 교체하면 된다

## 엔티티 매니저 설정 <a id="&#xC5D4;&#xD2F0;&#xD2F0;-&#xB9E4;&#xB2C8;&#xC800;-&#xC124;&#xC815;"></a>

### 엔티티 매니저 팩토리 생성 <a id="&#xC5D4;&#xD2F0;&#xD2F0;-&#xB9E4;&#xB2C8;&#xC800;-&#xD329;&#xD1A0;&#xB9AC;-&#xC0DD;&#xC131;"></a>

```text
// META-INF/persistence.xml에서 이름이 jpa인 영속성 유닛을 찾아서 엔티티 매니저 팩토리 생성
EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpa");
```

* persistence설정 정보를 읽어서 JPA 기반 객체를 생성하고, 구현체에 따라 커넥션 풀도 생성하므로 엔티티 매니저 팩토리를 생성하는 비용은 아주 크다
* `엔티티 매니저 팩토리는 애플리케이션 전체에서 딱 한 번만 생성하고 공유해서 사용해야 한다`

### 엔티티 매니저 생성 <a id="&#xC5D4;&#xD2F0;&#xD2F0;-&#xB9E4;&#xB2C8;&#xC800;-&#xC0DD;&#xC131;"></a>

```text
// 엔티티 매니저 팩토리를 통해 엔티티 매니저를 생성
EntityManager em = emf.createEntityManager();
```

* JPA의 기능 대부분은 엔티티 매니저가 제공한다\(CRUD\)
* 엔티티 매니저는 내부에 데이터소스\(데이터베이스 커넥션\)를 유지하면서 데이터베이스와 통신한다
* `엔티티 매니저는 DB 커넥션과 밀접한 관계가 있으므로 스레드간에 공유하거나 재사용하면 안 된다`

### 종료 <a id="&#xC885;&#xB8CC;"></a>

```text
// 엔티티 매니저 종료
em.close();

// 엔티티 매니저 팩토리 종료
emf.close();
```

## JPA 데이터 처리 <a id="jpa-&#xB370;&#xC774;&#xD130;-&#xCC98;&#xB9AC;"></a>

### 트랜잭션 관리 <a id="&#xD2B8;&#xB79C;&#xC7AD;&#xC158;-&#xAD00;&#xB9AC;"></a>

* JPA를 사용하면 항상 트랜잭션 안에서 데이터를 변경해야 한다
* 트랜잭션 없이 데이터를 변경하면 예외가 발생

### CRUD <a id="crud"></a>

```text
// 등록
em.persist(member);

// 수정
// JPA는 엔티티의 값만 변경하면 데이터베이스에 값을 변경한다 (dirty checking)
member.setAge(20);

// 삭제
em.remove(member);

// 조회 (단건)
em.find(Member.class, id);
```

### JPQL <a id="jpql"></a>

```text
// 목록 조회
// 여기서 from Member는 회원 엔티티 객체. Member 테이블이 아님
TypedQuery<Member> query = em.createQuery("select m from Member m", Member.class);
List<Member> members = query.getResultList();
```

* SQL을 추상화한 객체지향 쿼리 언어
* JPA는 엔티티 객체를 중심으로 개발하고 처리한다
* 애플리케이션이 필요한 데이터만 데이터베이스에서 불러오려면 검색 조건이 포함된 SQL을 사용해야 한다 \(엔티티 객체를 대상으로 검색하려면 DB의 모든 데이터를 애플리케이션으로 불러와서 객체로 변경한 다음 검색해야하는 문제 발생\)
* JPQL은 엔티티 객체\(클래스와 필드\)를 대상으로 쿼리
* SQL은 데이터베이스 테이블을 대상으로 쿼리
* JPQL은 데이터베이스 테이블을 전혀 알지 못하며, 엔티티 객체를 이용하여 처리한다

### 참고 <a id="&#xCC38;&#xACE0;"></a>

* [더티 체킹 \(Dirty Checking\)이란?](https://jojoldu.tistory.com/415)

