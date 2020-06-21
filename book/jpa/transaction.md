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

* 낙관적 락 : 트랜잭션 대부분은 충돌이 발생하지 않는다고 가정. 애플리케이션이 제공하는 락을 사용한다.                 트랜잭션을 커밋하기 전까지는 충돌을 알 수 없다.
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



