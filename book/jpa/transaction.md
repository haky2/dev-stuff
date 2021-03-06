---
description: 자바 ORM 표준 JPA 프로그래밍
---

# 16장. 트랜잭션과 락, 2차 캐시

## 트랜잭션과 락

### 트랜잭션과 격리 수준

트랜잭션은 ACID를 보장해야 한다. 트랜잭션은 원자성, 일관성, 지속성을 보장한다. 

* 원자성 \(Atomicity\)
  * 트랜잭션 내에서 실행한 작업들은 하나의 작업처럼 모두 성공하든가 모두 실패해야 한다.
* 일관성 \(Consistency\)
  * 모든 트랜잭션은 일관된 데이터베이스 상태를 유지해야 한다.
* 격리성 \(Isolcation\)
  * 동시에 실행되는 트랜잭션들이 서로에게 영향을 미치지 않도록 격리한다.  격리성은 동시성과 관련된 성능 이슈로 인해 격리 수준을 선택할 수 있다.
* 지속성 \(Durability\)
  * 트랜잭션을 성공적으로 끝내면 그 결과가 항상 기록되어야 한다.

격리성을 완벽히 보장하려면 트랜잭션을 거의 차례대로 실행해야 한다. 이렇게 하면 동시성 처리 성능이 매우 나빠진다.   
이런 문제로 인해 ANSI 표준은 트랜잭션의 격리 수준을 4단계로 정의했다.

| ISOLATION LEVEL | DRITY READ | NON-REPEATABLE READ | PHANTOM READ |
| :--- | :--- | :--- | :--- |
| READ UNCOMMITTED | O | O | O |
| READ COMMITTED |  | O | O |
| REPEATABLED READ |  |  | O |
| SERIALIZABLE |  |  |  |

| 격리 수준에 따른 문제점 | DESC |
| :--- | :--- |
| DRITY READ | 커밋하지 않은 데이터를 읽을 수 있다. |
| NON-REPEATABLE READ | 반복해서 같은 데이터를 읽을 수 없다. |
| PHANTOM READ | 반복 조회 시 결과 집합이 달라진다. |

격리 수준이 낮을수록 더 많은 문제가 발생하며, 격리 수준이 높을 수록 동시성 처리 성능이 급격히 떨어진다.  
애플리케이션 대부분은 동시성 처리가 중요하므로 보통 `READ COMMITTED` 격리 수준을 기본으로 사용한다

### Lock

JPA의 영속성 컨텍스트\(1차 캐시\)를 적절히 활용하면 데이터베이스 트랜잭션이 READ COMMITTED 수준이어도 애플리케이션 레벨에서 REPEATABLE READ가 가능하다. \(엔티티 조회인 경우\)

JPA는 트랜잭션 격리 수준을 READ COMMITTED로 가정한다. 만약 더 높은 수준의 격리 수준이 필요하면 `낙관적 락`과 `비관적 락` 중 하나를 사용한다

* 낙관적 락 : 트랜잭션 대부분은 충돌이 발생하지 않는다고 가정. 애플리케이션이 제공하는 락을 사용한다. 트랜잭션을 커밋하기 전까지는 충돌을 알 수 없다.
* 비관적 락 : 트랜잭션의 충돌이 발생한다고 가정하고 우선 락을 거는 방법. 데이터베이스가 제공하는 락을 사용한다.

데이터베이스 트랜잭션 범위를 넘어서는 경우도 발생한다.  
A, B가 동시에 데이터를 수정하면 나중에 완료\(수정\)한 데이터만 남게 된다. 이러한 문제를 `두 번의 갱신 분실 문제`라 한다.  
이러한 문제가 발생한 경우 아래의 방법 중 하나를 선택하여 처리한다.

* 마지막 커밋만 인정하기
* 최초 커밋만 인정하기
* 충돌하는 갱신 내용 병합하기

기본적으로 마지막 커밋만 인정하기가 사용된다.

### @Version

버전 관리 기능을 해주는 어노테이션. 적용 가능한 타입은 아래와 같다

* Long \(long\)
* Integer \(int\)
* Short \(short\)
* Timestamp

엔티티를 수정할 때 마다 버전이 하나씩 자동으로 증가한다. \(연관관계 필드는 외래 키를 관리하는 연관관계의 주인 필드를 수정할 때만 버전이 증가한다.\) 그리고 엔티티를 수정할 때 조회 시점의 버전과 수정 시점의 버전이 다르면 예외가 발생한다. 따라서 버전 정보를 사용하면 `최초 커밋만 인정하기`가 적용된다.

{% hint style="info" %}
벌크 연산은 버전을 무시한다.  
벌크 연산에서 버전을 증가하려면 버전 필드를 강제로 증가시켜야 한다.
{% endhint %}

### JPA Lock

{% hint style="success" %}
#### JPA를 사용할 때 추천하는 전략

READ COMMITTED + 낙관적 버전 관리 \(두 번의 갱신 내역 분실 문제 예방\)
{% endhint %}

Lock은 다음 위치에 적용할 수 있다.

* EntityManager.lock\(\), EntityManager.find\(\), EntityManager.refresh\(\)
* Query.setLockMode\(\)
* @NamedQuery

```java
// ex1
Board board = em.find(Board.class, id, LockModeType.OPTIMISTIC);

// ex2
Board board = em.find(Board.class, id);
...
em.lock(board, LockModeType.OPTIMISTIC);
```

### JPA Optimistic Lock

JPA가 제공하는 낙관적 락은 버전\(@Version\)을 사용한다. 낙관적 락은 트랜잭션을 커밋하는 시점에 충돌을 알 수 있다는 특징이 있다. 낙관적 락의 옵션은 아래와 같다.

{% tabs %}
{% tab title="NONE" %}
* 용도 : 조회 시점부터 수정 시점까지 보장
* 동작 : 엔티티를 수정할 때 버전을 증가한다. 버전이 다르면 예외가 발생한다.
* 이점 : 두 번의 갱신 분실 문제를 예방한다.
{% endtab %}

{% tab title="OPTIMISTIC" %}
* 용도 : 조회 시점부터 트랜잭션이 끝날 때까지 조회한 엔티티가 변경되지 않음을 보장
* 동작 : 트랜잭션을 커밋할 때 버전을 조회하여 다르면 예외 발생
* 이점 : DIRTY READ와 NON-REPEATABLE READ 방지
* 비고 : @Version만 사용할 때에는 수정해야 버전 정보를 확인하지만, OPTIMISTIC 옵션은 엔티티를 수정하지 않고 단순히 조회만 해도 버전을 확인
{% endtab %}

{% tab title="OPTIMISTIC\_FORCE\_INCREMENT" %}
* 용도 : 논리적인 단위의 엔티티 묶음을 관리할 수 있다. \(ex. 첨부파일 수정할때 게시글 버전 강제 업데이트\)
* 동작 : 엔티티를 수정하지 않아도 트랜잭션을 커밋할 때 버전 정보를 강제로 증가. 
* 이점 : 강제로 버전을 증가해서 논리적인 단위의 엔티티 묶음을 버전 관리 가능
{% endtab %}
{% endtabs %}

### JPA Pessmistic Lock

JPA가 제공하는 비관적 락은 데이터베이스 트랜잭션 락 메커니즘에 의존하는 방법이다. 주로 SQL 쿼리에 [select for update](https://dololak.tistory.com/446) 구문을 사용하면서 시작하고 버전 정보는 사용하지 않는다. 주로 PESSIMISTIC\_WRITE 모드를 사용한다.

* 엔티티가 아닌 스칼라 타입을 조회할 때도 사용할 수 있다.
* 데이터를 수정하는 즉시 트랜잭션 충돌을 감지할 수 있다.

{% tabs %}
{% tab title="PESSIMISTIC\_WRITE" %}
* 용도 : 데이터베이스에 쓰기 락을 건다.
* 동작 : 데이터베이스 select for update를 사용해서 락을 건다.
* 이점 : NON-REPEATABLE READ를 방지. 락이 걸린 로우는 다른 트랜잭션이 수정할 수 없다.
{% endtab %}

{% tab title="RESSIMISTIC\_READ" %}
* 용도 : 데이터를 반복 읽기만 하고 수정하지 않을때 사용 \(일반적으로 사용하지 않음\)
{% endtab %}

{% tab title="PESSIMISTIC\_FORCE\_INCREMENT" %}
* 용도 : 비관적 락 중 유일하게 버전 정보 사용. 버전 정보 강제 증가 시킴
{% endtab %}
{% endtabs %}

## 2차 캐시

영속성 컨텍스트 내부에는 엔티티를 보관하는 저장소가 있는데 이것을 `1차 캐시`라 한다. 일반적인 웹 애플리케이션 환경은 트랜잭션을 시작하고 종료할 때까지만 1차 캐시가 유효하다.

하이버네이트를 포함한 대부분의 JPA 구현체들은 애플리케이션 범위의 캐시를 지원하는데 이것을 `공유 캐시 또는 2차 캐시`라 한다.

### 1차 캐시

1차 캐시는 영속성 컨텍스트 내부에 있다. 엔티티 매니저로 조회하거나 변경하는 모든 엔티티는 1차 캐시에 저장된다. 트랜잭션을 커밋하거나 플러시를 호출하면 1차 캐시에 있는 엔티티의 변경 내역을 데이터베이스에 동기화 한다.

1차 캐시는 끄고 켤 수 있는 옵션이 아니다. 영속성 컨텍스트 자체가 사실상 1차 캐시다.

* 1차 캐시는 같은 엔티티가 있으면 해당 엔티티를 그대로 반환한다. 따라서 1차 캐시는 객체 동일성을 보장한다.
* 1차 캐시는 기본적으로 영속성 컨텍스트 범위의 캐시다.

### 2차 캐시

애플리케이션에서 공유하는 캐시를 JPA는 공유 캐시\(shared cache\)라 하는데 일반적으로 2차 캐시\(second level cache, L2 cache\)라 부른다. 애플리케이션을 종료할 때까지 캐시가 유지되며 분산 캐시나 클러스터링 환경의 캐시는 더 오래 유지될 수도 있다.

* 2차 캐시는 영속성 유닛 범위의 캐시다.
* 2차 캐시는 조회한 객체를 그대로 반환하는 것이 아니라 복사본을 만들어서 반환한다.
* 2차 캐시는 데이터베이스 기본 키를 기준으로 캐시하지만 영속성 컨텍스트가 다르면 객체 동일성을 보장하지 않는다.

### JPA 2차 캐시 기능

#### 캐시 모드 설정

* @Cacheable 어노테이션을 사용하면 된다. 기본값은 @Cacheable\(true\)이다
* sharedCacheMode를 설정해서 애플리케이션 전체에 적용할 수 있다.

#### 캐시 조회, 저장 방식 설정

* retrieveMode : 캐시 조회 모드 프로퍼티 이름
  * CacheRetrieveMode : 조회 모드 설정 옵션
    * USE : 캐시에서 조회한다 \(기본값\)
    * BYPASS : 캐시를 무시하고 데이터베이스에 직접 접근한다.
* storeMode : 캐시 보관 모드 프로퍼티 이름
  * CacheStoreMode : 보관 모드 설정 옵션
    * USE : 조회한 데이터를 캐시에 저장한다. 이미 있는 경우 최신 상태로 갱신하지는 않는다. \(기본값\)
    * BYPASS : 캐시에 저장하지 않는다.
    * REFRESH : USE + 데이터베이스에서 조회한 엔티티를 최신 상태로 다시 캐시한다.

### 하이버네이트의 캐시

하이버네이트가 지원하는 캐시는 크게 3가지가 있다.

* 엔티티 캐시 : 식별자로 엔티티를 조회하거나 컬렉션이 아닌 연관된 엔티티를 로딩할 때 사용
* 컬렉션 캐시 : 컬렉션이 엔티티를 담고 있으면 식별자 값만 캐시한다
* 쿼리 캐시 : 쿼리와 파리미터 정보를 키로 사용해서 캐시한다. 결과가 엔티티면 식별자 값만 캐시한다.

#### 쿼리 캐시와 컬렉션 캐시의 주의점

엔티티 캐시는 엔티티 정보를 모두 캐시하지만 쿼리 캐시와 컬렉션 캐시는 결과 집합의 식별자 값만 캐시한다. 이 식별자 값을 하나씩 엔티티 캐시에서 조회해서 실제 엔티티를 찾는다.  
따라서 쿼리 캐시나 컬렉션 캐시를 사용하면 결과 대상 엔티티에는 꼭 엔티티 캐시를 적용해야 한다.

## 참고

* [Transaction의 Isolation Level](https://suhwan.dev/2019/06/09/transaction-isolation-level-and-lock/)
* [다중화 클러스티링과 리플리케이션](https://kgh940525.tistory.com/entry/Database-DB-%EC%84%9C%EB%B2%84%EC%9D%98-%EB%8B%A4%EC%A4%91%ED%99%94Multiplexing-%ED%81%B4%EB%9F%AC%EC%8A%A4%ED%84%B0%EB%A7%81-%EB%A6%AC%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98Replication)

