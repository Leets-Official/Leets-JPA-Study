### @Entity

- 테이블과 매핑할 클래스는 필수로 붙여야 한다.

| 속성 | 기능 | 기본값 |
| ---- | ---- | ------ |
| name | JPA에서 사용할 엔티티 이름 지정 | 클래스 이름 |

**주의사항**
- 기본 생성자 필수 (@NoArgsConstructor)
- final 클래스, enum, interface, inner 클래스 사용 X
- 저장할 필드에 final 사용 X

### @Table

- 엔티티와 매핑할 테이블 지정

| 속성 | 기능 | 기본값 |
| ---- | ---- | ------ |
| name | 매핑할 테이블 이름 | 엔티티 이름 |
| catalog | catalog 기능이 있는 db에서 매핑 | - |
| schema | schema 기능이 있는 db에서 매핑 | - |
| uniqueConstraints (DDL) | DDL 생성 시 유니크 제약조건 생성. 2개 이상의 복합 유니크 제약 조건도 만들 수 있다. DDL을 만들 때만 사용. | - |

### 데이터베이스 스키마 자동 생성

- JPA는 데이터베이스 스키마를 자동으로 생성하는 기능을 지원한다.
- JPA는 매핑 정보와 데이터베이스 방언을 사용해서 스키마를 생성한다.

**`hibernate.hbm2ddl.auto` 속성**

| 옵션 | 설명 |
| ---- | ---- |
| create | 기존 테이블을 삭제 + 새로 생성 (DROP + CREATE) |
| create-drop | create 속성에 추가로 애플리케이션 종료시 생성한 DDL 제거 (DROP + CREATE + DROP) |
| update | db 테이블과 엔티티 매핑 정보를 비교해서 변경 사항만 수정 |
| validate | db 테이블과 엔티티 매핑 정보를 비교해서 차이가 있으면 경고를 남기고 애플리케이션 실행 X |
| none | 자동 생성 기능 사용하지 않을시 속성 자체를 삭제하거나 유효하지 않은 옵션 값을 사용 (none은 유효하지 않은 값) |

### 기본 키 매핑

```java
@Entity
public class Member {

  @Id
  @Column(name = "ID")
  private String id;
  ...
}
```
## 기본 키 생성 전략

**직접 할당**: 기본 키를 애플리케이션에서 직접 할당한다. (`@Id`로 매핑)

- **@Id 적용 가능 자바 타입**:
    - 자바 기본형
    - 자바 Wrapper 형
    - String
    - Date
    - BigDecimal, BigInteger

- `em.persist()`로 엔티티를 저장하기 전에 애플리케이션에서 기본 키를 직접 할당하는 방법

```
Board board = new Board();
board.setId("id1"); // 기본 키 직접 할당
em.persist(board);
```

## 자동 생성: 대리키 사용 방식

**IDENTITY**: 기본 키 생성을 데이터베이스에 위임한다.
- `@GeneratedValue` 어노테이션 + `GenerationType.IDENTITY` 지정
- `em.persist()` 호출 -> 엔티티 저장 -> 할당된 식별자 값 출력 (em.persist() 호출 즉시 INSERT SQL 동작, 쓰기 지연 X)
- ex) MySQL, PostgreSQL

**SEQUENCE**: 데이터베이스 시퀀스를 사용해서 기본 키 할당
- 유일한 값을 순서대로 생성하는 특별한 데이터베이스 object
- `@SequenceGenerator`를 사용해서 시퀀스 생성기를 등록한다.
- `GenerationType.SEQUENCE` + `generator = "시퀀스 생성기 name"`
- `em.persist()` 호출 -> 데이터베이스 시퀀스 사용해서 식별자 조회 -> 조회한 식별자를 엔티티에 할당 -> 엔티티를 영속성 컨텍스트에 저장 -> 트랜잭션 커밋 및 플러시 -> 엔티티 데이터베이스 저장
- ex) Oracle, PostgreSQL, DB2, H2

**TABLE**: 키 생성 테이블을 사용
- 테이블을 만들어 데이터베이스 시퀀스를 흉내내는 전략이다. (모든 데이터베이스에 적용 가능)
- `@TableGenerator`를 사용해서 테이블 키 생성기를 등록한다.
- `GenerationType.TABLE` + `generator = "테이블 키 생성기 name"`
- 시퀀스 전략과 내부 동작 방식은 같다. (시퀀스 대신 테이블 사용)

**AUTO**: 데이터베이스 방언에 따라 IDENTITY, SEQUENCE, TABLE 전략 중 하나를 자동으로 선택
- `@GeneratedValue.strategy`의 기본값

## 필드와 컬럼 매핑 : 레퍼런스
### @Column
- 객체 필드를 테이블컬럼에 매핑
- name, nullable 주로 사용

| 속성                    | 기능                                                                     | 기본값                                 |
|-----------------------|------------------------------------------------------------------------|-------------------------------------|
| name                  | 필드와 매핑할 테이블의 컬럼 이름                                                     | 객체의 필드 이름                           |
| insertable (거의 사용 X)  | 엔티티 저장 시 이 필드도 같이 저장(false : 읽기 전용)                                    | true                                |
| updateable (거의 사용 X)  | 엔티티 수정 시 이 필드도 같이 수정(false : 읽기 전용)                                    | true                                |
| table (거의 사용 X)       | 하나의 엔티티를 두개 이상의 테이블에 매핑할 때 사용                                          | 현재 클래스가 매핑된 테이블                     |
| nullable(DDL)         | null 값의 허용 여부 설정(false : not null 제약조건)                                | true                                |
| unique(DDL)           | @Table의 uniqueConstraints와 동일,한 컬럼에 유니크 제약조건을 걸 때 사용                   | &nbsp;                              |
| columnDefinition(DDL) | db 컬럼 정보를 직접 줄 수 있다.                                                   | 필드의 자바 타입과 방언 정보를 사용해 적절한 컬럼 타입을 생성 |
| length(DDL)           | 문자 길이 제약조건, String 타입에만 사용                                             | 255                                 |
| precision, scale(DDL) | BigDecimal, BigInteger에서 사용,precision : 소수점 포함한 전체 자릿수,scale : 소수의 자릿수 | precision = 19, scale = 2           |

### @Enumerated
- 자바의 enum 타입을 매핑할 때 사용

  | 속성    | 기능                                                                           | 기본값              |
  |-------|------------------------------------------------------------------------------|------------------|
  | value | - EnumType.ORDINAL : enum 순서를 db에 저장 <br>- EnumType.STRING : enum 이름을 db에 저장 | EnumType.ORDINAL |

- EnumType.ORDINAL은 enum에 정의된 순서대로 db에 저장하기 때문에, enum의 값이 바뀔 시(순서) 이미 저장된 이전 enum의 순서를 변경할 수 없는 단점이 있다.

### @Temporal
- 날짜 타입을 매핑할 때 사용

| 속성    | 기능                                                                                                                                                                                               | 기본값                  |
|-------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------|
| value | - TemporalType.DATE : 날짜, db date 타입과 매핑 (2013-10-11)<br>- TemporalType.TIME : 시간, db time 타입과 매핑 (11:11:11)<br>- TemporalType.TIMESTAMP : 날짜, 시간, db timestamp 타입과 매핑 &nbsp; (2013-10-11 11:11:11) | TemporalType은 필수로 지정 |

## @Lob
- db BLOB, CLOB 타입과 매핑
- **속성 정리**
    - **CLOB**: 필드 타입이 문자일 경우 / String, char[], java.sql.CLOB
    - **BLOB**: CLOB이 아닌 나머지 / byte[], java.sql.BLOB

## @Transient
- 이 필드는 매핑하지 않는다. (데이터베이스에 저장, 조회 X)
- 객체에 임시로 어떤 값을 보관하고 싶을 때 사용

## @Access
- JPA가 엔티티 데이터에 접근하는 방식을 지정
    - **필드 접근**: AccessType.FIELD, 필드에 직접 접근한다. private 필드도 접근 가능하다. (@Id의 위치가 필드에 있는 경우)
    - **프로퍼티 접근**: AccessType.PROPERTY, 접근자(Getter)를 사용한다.


