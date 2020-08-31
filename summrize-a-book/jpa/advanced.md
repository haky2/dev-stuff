---
description: 자바 ORM 표준 JPA 프로그래밍
---

# 15장. 고급 주제와 성능 최적화

## 예외 처리

JPA 표준 예외들은 PersistenceException의 자식 클래스이며 이 클래스는 RuntimeException의 자식이다.  
따라서 JPA 예외는 모두 `언체크 예외`다.

서비스 계층에서 데이터 접근 계층의 구현 기술에 직접 의존하는 것은 좋은 설계라 할 수 없다.  
예외도 마찬가지이며, 스프링 프레임워크는 이러한 문제를 해결하려고 데이터 접근 계층에 대한 예외를 추상화해서 개발자에게 제공한다.

## 엔티티 비교

영속성 컨텍스트 내부에는 엔티티 인스턴스를 보관하기 위한 1차 캐시가 있다. 이 1차 캐시는 영속성 컨텍스트와 생명주기를 같이 한다.

### 영속성 컨텍스트가 같을 때 엔티티 비교

* 동일성 비교가 같다
* 동등성 비교가 같다
* 데이터베이스 동등성 비교가 같다 \(데이터베이스 식별자\)

### 영속성 컨텍스트가 다를 때 엔티티 비교

* 동일성 비교가 다르다
* 동등성 비교가 같다 \(equals\(\)를 구현해야 한다. 보통 비즈니스 키로 구현한다.\)
* 데이터베이스 동등성 비교가 같다

## 프록시 심화 주제

### 영속성 컨텍스트와 프록시

영속성 컨텍스트는 프록시로 조회된 엔티티에 대해서 같은 엔티티를 찾는 요청이 오면 원본 엔티티가 아닌 처음 조회된 프록시를 반환한다. 따라서 프록시로 조회해도 영속성 컨텍스트는 영속 엔티티의 동일성을 보장한다.

원본 엔티티를 먼저 조회하면 영속성 컨텍스트는 원본 엔티티를 이미 조회했으므로 프록시를 반환할 이유가 없다.   
프록시를 호출해도 원본을 반환하므로 영속성 컨텍스트는 영속 엔티티의 동일성을 보장한다.

### 프록시 타입 비교

프록시는 원본 엔티티를 상속 받아서 만들어지므로 프록시로 조회한 엔티티의 타입을 비교할 때는 `instanceof`를 사용해야 한다.

### 프록시 동등성 비교

엔티티의 동등성을 비교하려면 비즈니스 키를 사용해서 equals\(\)를 오버라이딩하고 비교하면 된다.  
프록시는 실제 데이터를 가지고 있지 않아서 멤버변수에 직접 접근하면 아무값도 조회할 수 없다. 따라서 `접근자(getter)`를 사용하여 비교 코드를 작성해야 한다.

### 상속관계와 프록시

프록시를 부모 타입으로 조회하면 부모의 타입을 기반으로 프록시가 생성되는 문제가 있다.

* instanceof 연산을 사용할 수 없다.
* 하위 타입으로 다운캐스팅을 할 수 없다.

해결 방법은 다음과 같다.

* JPQL로 대상 직접 조회
  * 간단하지만 다형성을 활용할 수 없다.
* 프록시 벗기기 \(하이버네이트\)
  * 프록시에서 원본 엔티티를 직접 꺼내기 때문에 프록시와 원본 엔티티의 동일성 비교가 실패한다는 문제가 발생한다.
* 별도 인터페이스 제공
  * 공통 인터페이스를 만들고 공통 메소드를 구현한다. 다형성 활용에 좋은 방법이다.
* visitor patten 사용
  * 프록시에 대한 걱정 없이 안전하게 원본 엔티티에 접근할 수 있다.
  * instanceof와 타입캐스팅 없이 코드를 구현할 수 있다.
  * 알고리즘과 객체 구조를 분리해서 구조를 수정하지 않고 새로운 동작을 추가할 수 있다.
  * 너무 복잡하고 더블 디스패치를 사용하기 때문에 이해하기 어렵다.
  * 객체 구조가 변경되면 모든 visitor를 수정해야 한다.

## 성능 최적화

### N + 1 문제

#### 즉시 로딩과 N + 1

즉시 로딩은 JPQL을 실행할 때 처음 실행한 SQL의 결과 수만큼 추가로 SQL을 실행하여 N + 1 문제가 발생할 수 있다. \(루트 엔티티 조회\(1\) + 연관 엔티티 즉시 조회 \(N\)\)

#### 지연 로딩과 N + 1

JPQL에서는 발생하지 않지만, 모든 루트 엔티티에 대해 연관된 엔티티 컬렉션을 사용\(초기화\)할 때 발생한다. \(회원이 5명이면 회원에 따른 주문도 5번 조회된다.\)

### N + 1 해결

#### 페치 조인 사용

페치 조인은 SQL 조인을 사용해서 연관된 엔티티를 함꼐 조회하므로 N + 1 문제가 발생하지 않는다.

#### 하이버네이트 @BatchSize

@BatchSize 어노테이션을 사용하면 연관된 엔티티를 조회할 때 지정한 size만큼 SQL의 IN 절을 사용해서 조회한다.  
만약 조회한 회원이 10명인데 size=5로 지정하면 2번의 SQL만 추가로 실행한다.

#### 하이버네이트 @Fetch\(FetchMode.SUBSELECT\)

@Fetch\(FetchMode.SUBSELECT\)를 사용하면 연관된 데이터를 조회할 때 서브 쿼리를 사용해서 N + 1 문제를 해결한다.

### 읽기 전용 쿼리의 성능 최적화

엔티티가 영속성 컨텍스트에 관리되면 1차 캐시부터 변경 감지까지 얻을 수 있는 혜택이 많지만, 스탭샷 인스턴스를 보관하므로 더 많은 메모리를 사용하는 단점이 있다. 딱 한 번만 읽어서 사용할 경우에는 읽기 전용으로 엔티티를 조회하면 메모리 사용량을 최적화 할 수 있다.

* 스칼라 타입으로 조회
* 읽기 전용 쿼리 힌트 사용
* 읽기 전용 트랜잭션 사용 \(@Transactional\(readOnly = true\)\)
* 트랜잭션 밖에서 읽기

### 배치 처리

#### JPA 등록 배치

대량의 엔티티를 한 번에 등록할 때 주의할 점은 영속성 컨텍스트에 엔티티가 계속 쌓이지 않도록 일정 단위마다 영속성 컨텍스트의 엔티티를 데이터베이스에 플러시하고 영속성 컨텍스트를 초기화해야 한다.

#### JPA 페이징 배치 처리

페이징 쿼리로 조회하면서 페이지 단위마다 영속성 컨텍스트를 플러시하고 초기화한다.

#### 하이버네이트 scroll 사용

하이버네이트는 scroll 이라는 이름으로 JDBC 커서를 지원한다.

#### 하이버네이트 무상태 세션 사용

무상태 세션은 영속성 컨텍스트를 만들지 않고 2차 캐시도 사용하지 않는다. 직접 update를 호출한다.

### SQL 쿼리 힌트 사용

JPA는 데이터베이스 SQL 힌트 기능을 제공하지 않는다. SQL 힌트를 사용하려면 하이버네이트를 직접 사용해야 한다.

### 트랜잭션을 지원하는 쓰기 지연과 성능 최적화

네트워크 호출 한 번은 단순한 메소드를 수만 번 호출하는 것보다 더 큰 비용이 든다. JDBC가 제공하는 SQL 배치 기능을 사용하면 SQL을 모아서 한번에 보낼 수 있다. \(hibernate.jdbc.batch\_size\)

트랜잭션을 지원하는 쓰기 지연과 변경 감지 기능은 데이터베이스 테이블 로우에 락이 걸리는 시간을 최소화하는 장점이 있다. 트랜잭션을 커밋해서 영속성 컨텍스트를 플러시하기 전까지는 데이터베이스에 등록, 수정, 삭제하지 않는다. 따라서 커밋 직전까지 데이터베이스 로우에 락을 걸지 않는다.

## 참고

* [예외 처리](http://www.nextree.co.kr/p3239/)
* [더블 디스패치](http://wonwoo.ml/index.php/post/1490)
* [sql 커서](https://rh-cp.tistory.com/50)


