## 6.1 다대일
### 6.1.1 다대일 단방향 [N:1]
회원 엔티티(Member)와 팀 엔티티(Team)는 다대일 관계이다.
```java
@Entity
public class Member {

    ...
    
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    
    ...
}
@Entity
public class Team {

    ...
    
    @Id
    @Column(name = "TEAM_ID")
    private Long id;
    
    ...
}
```

- 회원은 Member.team을 통해 팀 엔티티를 참조할 수 있다.
- 팀에는 회원을 참조하는 필드가 없기 때문에 다대일 단방향 연관관계다.

## 6.1.2 다대일 양방향 [N:1]
```java
@Entity
public class Member {

    ...
    
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    
    public void setTeam(Team team) {
    	this.team = team;
        
        if(!team.getMembers().contains(this)) {
        	team.getMembers().add(this);
        }
}
@Entity
public class Team {

    ...
    
    @Id
    @Column(name = "TEAM_ID")
    private Long id;
    
    @OneToMany(mappedBy = "team") // Member.team (Team은 연관관계의 주인이 아님을 밝힘)
    private List<Member> members = new ArrayList<>();
    
    public void addMember(Member member) {
    	this.member.add(member);
        
        // 무한루프 방지 코드
        if(member.getTeam() != this) {
        	member.setTeam(this);
        }
    }
}
```
- 양방향은 외래 키가 있는 곳이 연관관계의 주인이다.
    - 일대다와 다대일 연관관계는 항상 다(N)에 외래 키가 있다.
    - Member.team이 연관관계의 주인이다.
- 양방향 연관관계는 항상 서로를 참조해야한다.

## 6.2 일대다
### 6.2.1 일대다 단방향 [1:N]
- 하나의 팀은 여러 회원을 참조할 수 있다. <-> 회원은 팀을 참조하지 않는다.
- Member 테이블의 Team_ID를 외래키로 갖지만, 일대다 단방향 매핑은 반대쪽 테이블에 있는 외래 키를 관리한다.
- 일대다 단방향 관계를 매핑할 때는 @JoinColumn을 명시한다. (그렇지 않으면 조인 테이블 전략을 기본으로 사용하여 매핑)

```java
@Entity
public class Team {

    ...
    
    @Id
    @Column(name = "TEAM_ID")
    private Long id;
    
    @OneToMany
    @JoinColumn(name = "TEAM_ID") // Member 테이블의 FK
    private List<Member> members = new ArrayList<>();
}
```
### 일대다 단방향 단점
- Member 테이블에서 외래 키를 관리해야하지만, 여기선 Team 테이블에서 관리하므로 엔티티를 매핑한 테이블이 아닌 다른 테이블의 외래 키를 관리해야 한다.
- 일대다 단방향 매핑대신 다대일 양방향 매핑을 권장한다.

## 6.2.2 일대다 양방향 [1:N, N:1]
- 일대다 양방향 매핑은 존재하지 않는다.
    - 양방향 매핑에서 @OneToMany가 연관관계의 주인이 될 수 없기 때문이다.
    - 다대일 양방향 매핑을 사용한다.

## 6.3 일대일 [1:1]
- 일대일 관계는 주 테이블이나 대상 테이블 둘 중 어느곳이나 외래 키를 가질 수 있다. (연관관계의 주인이 어디든 상관없다.)
### 6.3.1 주 테이블에 외래 키
주 테이블에 외래키가 있으면 주 테이블만 확인해도 대상 테이블과 연관관계가 있는지 알 수 있다.
1. 단방향
```java
@Entity
public class Member {

    ...
    
    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;
    
    ...
}

@Entity
public class Locker {

    @Id
    @Column(name = "LOCKER_ID")
    private Long id;
    
    ...
}
```
2. 양방향
```java
@Entity
public class Member {

    ...
    
    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;
    
    ...
}

@Entity
public class Locker {

    @Id
    @Column(name = "LOCKER_ID")
    private Long id;
    
    @OneToOne(mappedBy = "locker") // Member.locker가 연관관계의 주인이다.
    private Member member;
}
```

### 6.3.2 대상 테이블에 외래 키
1. 단방향 (X)
- JPA에서 일대일 관계 중 대상 테이블에 외래 키가 있는 단방향 관계 지원하지 않는다.
- Locker에 Member_ID를 FK로 갖고 있다면, Member의 locker로 연관관계를 매핑할 수 없다.
2. 양방향
```java
@Entity
public class Member {

    @Id
    @Column(name = "MEMBER_ID")
    private Long id;
    
    @OneToOne(mappedBy = "member")
    private Locker locker;
    
    ...
}

@Entity
public class Locker {

    @Id
    @Column(name = "LOCKER_ID")
    private Long id;
    
    @OneToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;
}
```
## 6.4 다대다 [N:N]
- 관계형 데이터베이스는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없다.
- 보통 다대다 관계를 일대다, 다대일 관계로 풀어내는 연결 테이블을 사용한다.
- Member >-< Product (X)
- Member -< Member_Product >- Product (다대다 연결 테이블)
- Member >-< Product (다대다 엔티티), @ManyToMany를 사용하여 편리하게 매핑한다.

### 6.4.1 다대다 : 단방향
```java
@Entity
public class Member {

  @Id
  @Column(name = "MEMBER_ID")
  private Long id;
    
    ...

  @ManyToMany
  @JoinTable(name = "MEMBER_PRODUCT",
      joinColumns = @JoinColumn(name = "MEMBER_ID"),
      inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID"))
  private List<Product> products = new ArrayList<>();
}
```

- @ManyToMany와 @JoinTable 을 사용해서 연결 테이블을 바로 매핑한 것이다.
- @JoinTable 속성
  - name : 연결 테이블을 지정한다.
  - joinColumns : 현재 방향인 회원과 매핑할 조인 컬럼 정보를 지정한다.
  - inverseJoinColums : 반대 방향인 상품과 매핑할 조인 컬럼 정보를 지정한다.

### 6.4.2 다대다 : 양방향
- 위와 동일하나 원하는 곳에 mappedBy로 연관관계 주인을 지정한다. (mappedBy가 없는 곳이 주인!)

### 6.4.3 다대다 : 매핑의 한계와 극복, 연결 엔티티 사용
- 연결 테이블에 단순히 주문한 회원 아이디와 상품 아이디만 담고 끝나지 않고 더 다른 컬럼들이 추가된다.
- 예를 들면, 주문 수량이나 주문 날짜와 같은 컬럼이 추가된다.
- 이럴 경우 @ManyToMany를 사용할 수 없다. => 연결 테이블을 매핑하는 연결 엔티티를 만든다.
- MemberProduct 테이블 대신 Order라는 엔티티를 만든다.

### 6.4.4 다대다 : 새로운 기본 키 사용
- 식별 관계 : 받아온 식별자를 기본 키 + 외래 키(Member, Product Id)로 사용한다.
- 비식별 관계 : 받아온 식별자는 외래 키로만 사용하고 새로운 식별자를 추가한다.
- 식별자 클래스를 생성하여 식별관계로 키를 만드는 것보다, 새로운 식별자를 추가하는 것이 낫다.

```java
@Entity
public class Order {

    @Id
    @Column(name = "ORDER_ID")
    private Long id;
    
    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;
    
    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product;
    
    private int orderAmount; // 추가한 컬럼
    
    ...

}
```
