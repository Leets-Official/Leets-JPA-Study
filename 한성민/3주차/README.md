💡 목표: 객체의 참조와 테이블의 외래 키를 매핑하는 것

## 단방향 연관관계
- 회원은 하나의 팀에만 소속될 수 있다.
- 회원과 팀은 다대일 관계다.


- 객체 연관관계
    - 회원 객체와 팀 객체는 단방향 관계.
    - Member.team 필드로 팀 객체와 연관관계를 맺는다.
    - member -> team의 조회는 가능, 반대는 불가.
    
- 테이블 연관관계
  - 회원 테이블은 TEAM_ID 외래키로 팀 테이블과 연관관계를 맺는다.
  - 회원과 팀 테이블은 양방향 관계 (회원, 팀 둘 다 조인 가능).

  
### 객체 연관관계와 테이블 연관관계 정리

- 객체는 참조(주소)로 연관관계를 맺는다.
    - 참조를 사용하는 객체의 연관관계는 단방향이다.
    - 객체를 양방향으로 참조하려면 단방향 연관관계를 2개 만들어야 한다.
- 테이블은 외래 키로 연관관계를 맺는다.
    - 외래 키를 사용하는 테이블의 연관관계는 양방향이다.

  
### 객체 관계 매핑
- 매핑한 회원 엔티티
```java
@Entity
public class Member {

@Id
@GeneratedValue
@Column(name = "MEMBER_ID")
private Long id;

private String username;

// 연관관계 매핑
@ManyToOne
@JoinColumn(name = "TEAM_ID")
private Team team;

// 연관관계 설정
public void setTeam(Team team) {
this.team = team;
}
}
```

- 매핑한 팀 엔티티
```java
@Entity
public class Team {

@Id
@GeneratedValue(strategy = GenerationType.AUTO)
@Column(name = "TEAM_ID")
private Long id;

private String name;
}
```
- 객체 연관관계 : 회원 객체의 Member.team 필드 사용.
- 테이블 연관관계 : 회원 테이블의 Member.TEAM_ID 외래 키 컬럼 사용.

### @JoinColumn : 외래 키를 매핑할 때 사용
| 속성                  | 기능                                            | 기본값                       |
|-----------------------|-----------------------------------------------|------------------------------|
| name                  | 매핑할 외래 키 이름                                   | 필드명 _ 참조하는 테이블 PK 컬럼명 |
| referencedColumnName  | 외래 키가 참조하는 대상 테이블의 컬럼명                        | 참조하는 테이블의 PK 컬럼명  |
| foreignKey(DDL)       | 외래 키 제약조건을 직접 지정할 수 있다. <br> 테이블 생성할 때만 사용한다. | 테이블 생성할 때만 사용한다. |

### @ManyToOne : 다대일 관계에서 사용
| 속성          | 기능                                                        | 기본값                                     |
|---------------|-------------------------------------------------------------|--------------------------------------------|
| optional      | false로 설정하면 연관된 엔티티가 항상 있어야 한다.          | true                                       |
| fetch         | 글로벌 페치 전략을 설정한다.                                | - @ManyToOne=FetchType.EAGER <br> - @ManyToOne=FetchType.LAZY |
| cascade       | 영속성 전이 기능을 사용한다.                                |                                            |
| targetEntity  | 연관된 엔티티의 타입 정보를 설정한다. <br> 거의 사용하지 않는다. |                                            |
<hr>

## 연관관계 사용
### 저장
```java
member1.setTeam(team1); // 회원 -> 팀 참조
em.persist(member1); // 저장
```

- 회원 엔티티는 팀 엔티티를 참조하고 저장한다.
- JPA는 참조한 팀의 식별자(Team_ID)를 외래 키로 사용해서 등록 쿼리를 생성한다.

### 조회
- 연관관계가 있는 엔티티를 조회하는 방법
1. 객체 그래프 탐색(객체 연관관계를 사용한 조회)
- member.getTeam()을 사용해서 member와 연관된 team 엔티티를 조회한다.

```java
Member member = em.find(Member.class, "member1");
Team team = member.getTeam(); // 객체 그래프 탐색
```

2. 객체지향 쿼리 사용(JPQL)
```java
String jpql = "select m from Member m join m.team t where " + "t.name=:teamName";

List<Member> resultList = em.createQuery(jpql, Member.class)
.setParameter("teamName", "팀1")
.getResultList();
```

### 수정
```java
// 새로운 팀2
Team team2 = new Team("team2", "팀2");
em.persist(team2);

// 회원1에 새로운 팀2 설정
Member member = em.find(Member.class, "member1");
member.setTeam(team2);
```

- 3장에서 이야기했듯이 불러온 엔티티 값만 변경해두면 트랜잭션을 커밋할 때 플러시가 일어나면서 변경 감지 기능이 작동한다.
- 변경사항을 db에 자동으로 반영한다.

### 연관관계 제거 및 연관된 엔티티 삭제
```java
Member member1 = em.find(Member.class, "member1");

// 연관관계 제거
member1.setTeam(null);

// 연관된 엔티티 삭제
em.remove(team);
```
<hr>

## 양방향 연관관계
### 양방향 연관관계 매핑
```java
@Entity
public class Team {

@Id
@GeneratedValue(strategy = GenerationType.AUTO)
@Column(name = "TEAM_ID")
private Long id;

private String name;

@OneToMany(mappedBy = "team")
private List<Member> members = new ArrayList<>();
}

```
### 연관관계의 주인

- 테이블은 외래 키 하나로 두 테이블의 연관관계를 관리한다.
- 하지만, 객체 연관관계는 양방향 관계를 단방향 연관관계 2개로 각각 만든다.
- 객체의 참조는 둘인데 외래 키는 하나인 차이가 발생한다.
→  연관관계의 주인 : 두 객체 연관관계 중 하나를 정해서 테이블의 외래 키를 관리하는 것

→  연관관계의 주인만이 외래 키를 관리(등록, 수정, 삭제)할 수 있다.

→  연관관계의 주인은 외래 키가 있는 곳으로 정해야 한다.

* mappedBy 속성

  * 주인은 mappedBy 속성을 사용하지 않는다.
  * 주인이 아니면 mappedBy를 사용해서 속성의 값으로 연관관계의 주인을 지정해야 한다.
  
```java
class Team {

      // mappedBy 속성의 값은 연관관계의 주인인 Member.team
      @OneToMany(mappedBy = "team") 
      private List<Member> members = new ArrayList<>();
      ...
  }
```
## 양방향 연관관계 저장
```java
Team team1 = new Team("team1", "팀1");
em.persist(team1);

Member member1 = new Member("member1", "회원1");
member1.setTeam(team1);
em.persist(member1);
```
단방향 연관관계에서 회원과 팀을 저장하는 코드와 완전히 같다.
왜냐하면, 연관관계의 주인인 Member.team에서 관리하기 때문이다.

### 주의점
```java
Team team1 = new Team("team1", "팀1");

Member member1 = new Member("member1", "회원1");
em.persist(member1);

// 주인이 아닌 곳만 연관관계 설정
team1.getMembers().add(member1);

em.persist(team1);
```
- Team.members는 연관관계의 주인이 아니기때문에 TEAM_ID 가 null이 입력된다.

→  객체 관점에서 양쪽 방향에 모두 값을 입력해주는 것이 가장 안전하다.

```java
Team team1 = new Team("team1", "팀1");
em.persist(team1);

Member member1 = new Member("member1", "회원1");

// 양방향 연관관계 설정
member1.setTeam(team1);
team1.getMembers().add(member1);
em.persist(member1);
```

### 정리
- 단방향 매핑만으로 테이블과 객체의 연관관계 매핑은 이미 완료된다.
- 단방향을 양방향으로 만들면 반대방향으로 객체 그래프 탐색 기능이 추가된다.
- 양방향 연관관계를 매핑하려면 객체에서 양쪽 방향을 모두 관리해야 한다.


