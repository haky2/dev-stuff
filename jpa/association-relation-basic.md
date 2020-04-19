---
description: 자바 ORM 표준 JPA 프로그래밍
---

# 5장. 연관관계 매핑 기초

## 객체 연관관계와 테이블 연관관계의 차이

객체는 `참조를 이용한 단방향 관계`이며, 테이블은 `외래 키를 이용한 양방향 관계`이다.  
객체 양쪽에서 서로 참조하는 것은 `양방향 관계가 아니라 서로 다른 단방향 관계 2개`이다.

## 객체 그래프 탐색

객체는 참조를 사용해서 연관관계를 탐색 할 수 있다.

```java
Team findTeam = member1.getTeam();
```

## 단방향 연관관계

### 객체 관계 매핑

```java
@Entity
public class Member {
    // 다대일(N:1) 관계 매핑 어노테이션
    @ManyToOne
    // 외래 키를 매핑하는 어노테이션 (생략 가능)
    @JoinColumn(name="TEAM_ID")
    private Team team;
}


@Entity
public class Team {
    @Id
    @Column(name = "TEAM_ID")
    private String id;
}
```

{% hint style="info" %}
#### @JoinColumn 생략

@JoinColumn을 생략하면 외래 키를 찾을 때 기본 전략을 사용한다.

* 기본 전략 : 필드명 + \_ + 참조하는 테이블의 컬럼
* ex. 필드명\(team\) + \_\(밑줄\) + 참조하는 테이블의 컬럼명\(TEAM\_ID\) = team\_TEAM\_ID
{% endhint %}

### CRUD

```java
public void create() {
    
    Team team1 = new Team("team1", "팀1");
    em.persist(team1);
    
    Member member1 = new Member("member1", "회원1");
    // JPA는 참조한 엔티티의 식별자를 외래 키로 사용해서 적절한 등록 쿼리 생성
    member.setTeam(team1);
    em.persist(member1);
}

public void readByEntityGraph() {
    
    Member member = em.find(Member.class, "member1");
    Team team = member.getTeam();
}

public void readByJPQL() {

    String jpql = "select m from Member m join m.team t";
    List<Member> resultList = em.createQuery(jpql, Member.class)
                                .getResultList();
}

public void update() {

    Team team2 = new Team("team2", "팀2");
    em.persist(team2);
    
    Member member = em.find(Member.class, "member1");
    member.setTeam(team2);
}

public void delete() {
    
    Member member = em.find(Member.class, "member1");
    member.setTeam(null);
    // 연관된 엔티티를 삭제하려면 기존에 있던 연관관계를 먼저 제거하고 삭제해야 한다.
    em.remove(team);
}
```

## 양방향 연관관계

### 객체 관계 매핑

```java
@Entity
public class Member {
    @ManyToOne
    @JoinColumn(name="TEAM_ID")
    private Team team;
}


@Entity
public class Team {
    @Id
    @Column(name = "TEAM_ID")
    private String id;
    
    // mappedBy 속성은 반대쪽 매핑의 필드 이름
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<Member>();
}
```

### 연관관계의 주인

엔티티를 양방향 연관관계로 설정하면 객체의 참조는 둘인데 외래 키는 하나다. 따라서 둘 사이에 차이가 발생한다.  
JPA에서는 두 객체 연관관계 중 하나를 정해서 테이블의 외래키를 관리해야 하는데 이것을 `연관관계의 주인(owner)`이라 한다.

* 연관관계의 주인만이 데이터베이스 연관관계와 매핑되고 외래 키를 관리\(등록, 수정, 삭제\)할 수 있다. 반면에 주인이 아닌 쪽은 읽기만 할 수 있다.
* 주인은 mappedBy 속성을 사용하지 않으며, `주인이 아니면 mappedBy 속성을 사용해서 주인을 지정`해야한다.
* 연관관계의 주인을 정한다는 것은 사실 외래 키 관리자를 선택하는 것이다.
* 연관관계의 주인은 테이블에 `외래 키가 있는 곳`으로 정해야 한다.

{% hint style="info" %}
데이터베이스 테이블의 다대일, 일대다 관계에서는 항상 다 쪽이 외래 키를 가진다.  
다 쪽인 @ManyToOne은 항상 연관관계의 주인이 되므로 mappedBy를 설정할 수 없다.  
따라서 @ManyToOne에는 mappedBy 속성이 없다.
{% endhint %}

### 양방향 연관관계의 주의점

연관관계의 주인만이 외래 키의 값을 변경 할 수 있다.

```java
public void save() {
    
    Member member1 = new Memeber("member1", "회원1");
    em.persist(member1);
    
    Member member2 = new Memeber("member2", "회원2");
    em.persist(member2);
    
    Team team1 = new Team("team1", "팀1");
    // 연관관계의 주인이 아닌 team에는 값을 저장할 수 없다 (null)
    team1.getMembers().add(member1);
    team1.getMembers().add(member2);
    
    // 연관관계의 주인에서 등록할 수 있다
    member1.setTeam(team1);
}
```

`객체 관점에서는 양쪽 방향에 모두 값을 입력해주는 것이 가장 안전하다.` 양쪽 방향 모두 값을 입력하지 않으면 JPA를 사용하지 않는 순수한 객체 상태에서 심각한 문제가 발생할 수 있다.

```java
public class Member {

    private Team team;
    
    /**
     * 연관관계 편의 메소드
     * 양방향 연관관계 모두에 값을 저장
     */
    public void setTeam(Team team) {
        // 연관관계 변경시 기존 관계가 삭제되도록 코드를 작성해야 한다.
        if (this.team != null) {
            this.team.getMembers().remove(this);
        }
        this.team = team;
        team.getMembers().add(this);
    }
}
```

{% hint style="danger" %}
양방향 매핑 시에는 무한 루프에 빠지지 않게 조심해야 한다.  
예를 들어 Member.toString\(\)에서 getTeam\(\)을 호출하고 Team.toString\(\)에서 getMember\(\)를 호출하면 무한 루프에 빠질 수 있다.
{% endhint %}

