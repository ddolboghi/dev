# JPA
- ORM(object-relational mapping)을 이용하면 데이터베이스 테이블블을 객체처럼 사용할 수 있음
- 자바의 표준은 JPA(java persistence API)
- JPA는 인터페이스므로 이를 구현하는 ORM프레임워크를 선택해야함
- hibernate : JPA를 구현하는 대표적인 ORM프레임워크. 내부적으로 JDBC API사용
## 스프링부트에서 JPA 사용 설정
1. build.gradle > dependencies 추가
```gradle
dependencies {
	...
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
}
```
2. application.properties 설정 추가
```properties
#jpa  
# 데이터베이스 엔진 종류 설정
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.H2Dialect  
# 엔티티를 기준으로 테이블을 생성하는 규칙 정의
spring.jpa.hibernate.ddl-auto=update
```
테이블 생성 규칙
- none: 엔티티가 변경되도 데이터베이스 변경 안함
- update: 엔티티의 변경된 부분만 적용
- validate: 변경사항이 있는지 검사만 함
- create: 스프링부트 서버가 시작될때 모두 drop하고 다시 생성
- create-drop: create와 동일, 종료할때도 모두 drop
개발 환경에서는 보통 update 모드 사용하고 운영환경에서는 none 또는 validate 모드 사용
# H2
- 파일 기반의 경량 데이터베이스
- 개발시 H2로 빠르게 개발하고 실제 운영시스템에는 좀 더 규모있는 DB사용
## 스프링부트에서 H2 사용 설정
1. build.gradle > dependencies 추가
```gradle
dependencies {
	...
	runtimeOnly 'com.h2database:h2'
}
```
2. application.properties 설정 추가
```properties
#DATABASE
# H2의 콘솔 접속 허용 여부
spring.h2.console.enabled=true
#콘솔 접속을 위한 URL경로
spring.h2.console.path=/h2-console
#데이터베이스 접속을 위한 경로
spring.datasource.url=jdbc:h2:~/local
#데이터베이스 접속시 사용하는 드라이버
spring.datasource.driverClassName=org.h2.Driver
#데이터베이스 사용자명(기본값 sa)
spring.datasource.username=sa
#데이터베이스 패스워드
spring.datasource.password=
#hibernate에서 실행되는 sql 쿼리 보기  
spring.jpa.properties.hibernate.format_sql=true  
spring.jpa.properties.hibernate.show_sql=true
```
- 데이터베이스 접속 경로에 데이터베이스 파일인 local.mv.db 생성해야함
	- `jdbc:h2:~/test`라면 test.mv.db 파일 생성
- 홈디렉터리 주소: `C:\Users\사용자명`

---
# entity
- 데이터베이스의 테이블과 매핑되는 자바 클래스
- 데이터베이스와 직접 연결되어 데이터베이스에 영향을 미치는 쿼리를 실행
- 모델 또는 도메인 모델이라 부르기도 함
### 엔티티의 상태
- transient : 영속성 컨텍스트와 전혀 관계가 없는 비영속 상태
	- 엔티티를 처음 만들면 엔티티는 비영속 상태가 됨
- managed : 영속성 컨텍스트가 관리하는 상태
- detached : 영속성 컨텍스트가 관리하고 있지 않는 상태
- removed : 삭제된 상태
```java
public class EntityManagerTest {
	@Autowired
	EntityManager em;
	public void example() {
		Member member = new Member(1L, "홍길동");//transient
		em.persist(member);//managed
		em.detach(member);//detached
		em.remove(member);//removed
	}
}
```
---
## entity manager
- 엔티티를 관리해 데이터베이스와 애플리케이션 사이에서 객체를 생성, 수정, 삭제
- ==엔티티 매니저는 엔티티를 영속성 컨텍스트에 저장함==
- entity manager factory에서 엔티티 매니저 만듬
- 스프링 부트는 내부에서 엔티티 매니저 팩토리를 하나만 생성해서 관리하고 @PersistenceContext나 @Autowired로 프록시 엔티티 매니저 사용.
  엔티티 매니저가 필요할때 데이터베이스 트랜잭션과 관련된 실제 엔티티 매니저 호출
- 엔티티 매니저는 Spring Data JPA에서 관리하므로 개발자가 직접 생성하거나 관리할 필요 없음
- ---
## 영속성 컨텍스트
- 영속성 컨텍스트는 엔티티를 관리하는 가상의 공간
### 1차 캐시
- 영속성 컨텍스트 내부에 있는 캐시로 키-값으로 데이터 저장
- 캐시의 키는 기본키 역할을하는 식별자고, 값은 엔티티
- 엔티티 조회하면 1차 캐시에서 조회하고 값이 있으면 반환
  값이 없으면 데이터베이스에서 조회해 1차 캐시에 저장한 다음 반환
  -> 캐시된 데이터를 조회할때는 데이터베이스를 거치지 않아 빠름
### 쓰기 지연(transactional write-behind)
- 트랜잭션을 커밋하기 전까지는 데이터베이스에 실제로 쿼리를 보내지 않고 모았다가 트랜잭션을 커밋하면 모았던 쿼리를 한번에 실행
- 적당한 묶음으로 쿼리 요청할 수 있어 데이터베이스 시스템의 부담 줄임
### 변경 감지
1. 트랜잭션을 커밋하면 1차 캐시에 저장된 엔티티의 값과 현재 엔티티의 값을 비교
2. 변경된 값이 있다면 변경 사항을 감지해 데이터베이스에 변경 값을 자동으로 반영
- 적당한 묶음으로 쿼리 요청할 수 있어 데이터베이스 시스템의 부담 줄임
### 지연 로딩(lazy)
- 쿼리로 요청한 데이터를 한번에 다 가져오지 않고 필요할때 쿼리를 날려 데이터를 가져옴
  <>즉시 로딩(eager) : 조회할때 쿼리를 보내 연관된 모든 데이터를 가져옴
- 대부분의 비즈니스 로직에서 연관 관계에 있는 객체를 함께 사용한다면 lazy 로딩시 select 쿼리가 2번 발생해 손해임
- 하지만 **실무에서는 거의 lazy 로딩만 사용함**
- 왜냐하면 eager 로딩시 예상치 못한 쿼리와 N+1 문제가 발생함
- 자주 함께 사용하는 연관 관계의 객체들은 lazy 로딩과 함께 JPQL의 fetch join으로 해당 시점에 한 방 쿼리로 가져옴
- 테스트 코드에서 여러 레포지토리 사용할때 하나의 레포지토리를 사용한 뒤에는 DB세션이 끊겨 **LazyInitializationException** 발생
  --> ==테스트 메서드에 `@Transactional` 붙여 메서드가 종료될때까지 DB세션 유지==
- 실제 서버에서 JPA 프로그램들을 실행할 때는 DB세션이 종료되지 않음
---
# spring data
- 데이터베이스 사용기능을 클래스 레벨에서 추상화
- CRUD, 페이징 처리 기능 등 
- 메서드명으로 자동으로 쿼리 만들어줌
- 각 데이터베이스의 특성에 맞춰 기능 확장(spring data JPA, spring data MongoDB)
# spring data JPA
- spring data의 공통적인 기능에서 JPA의 유용한 기술이 추가된 것
- JpaRepository인터페이스는 스프링 데이터의 PagingAndSortingRepository인터페이스를 상속받음
- 기존에는 메서드 호출로 엔티티 상태를 관리했지만 스프링 데이터 JPA를 사용하면 repository역할을 하는 인터페이스를 만들어 데이터베이스 CRUD 작업을 쉽게 할 수 있음
- JPA는 메서드 규칙에따라 메서드를 선언하면 이름을 분석해 자동으로 쿼리 생성
```JAVA
public interface EntityNameRepository extends JpaRepository<엔티티 이름, 엔티티 기본기의 타입> {
}
```

# 엔티티 클래스에 사용되는 애너테이션
```java
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor  
@Getter  
@Entity  
public class Member {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id", updatable = false)  
    private Long id;  
  
    @Column(name = "name", nullable = false)  
    private String name;  
}
```
## `@Entity`
- Member객체를 JPA가 관리하는 엔티티로 지정해 실제 데이터베이스의 테이블과 매핑시킴
- name속성으로 name값을 가진 테이블 이름과 매핑
- name지정안하면 클래스 이름과 같은 테이블 매핑
---
## `@NoArgsConstructor(access = AccessLevel.PROTECTED)`
- protected 기본 생성자 생성
- 엔티티는 반드시 기본 생성자가 있어야함
---
## `@Id`
- 해당 속성을 테이블의 기본 키(primary key)로 지정
## `@GeneratedValue(strategy = GenerationType.[strategy])`
- 해당 속성에 값을 따로 세팅하지 않아도 1씩 자동으로 증가하여 저장됨
- strategy: 기본 키의 생성 방식 결정
- strategy옵션을 생략하면 해당 컬럼이 모두 동일한 시퀀스로 번호 생성하므로 보통 `GenerationType.IDENTITY`사용

|strategy| |
|---|---|
|AUTO|선택한 데이터베이스 방언에 따라 방식을 자동 선택|
|IDENTITY|기본키 생성을 데이터베이스에 위임(=AUTO_INCREMENT)|
|SEQUENCE|데이터베이스 시퀀스를 사용해서 기본 키를 할당. 오라클에서 주로 사용|
|TABLE|키 생성 테이블 사용|

---
## `@Column`
- 엔티티 클래스의 속성은 `@Column`을 사용하지 않아도 테이블 컬럼으로 인식함
- ==테이블 컬럼으로 인식하고 싶지 않은 경우에만 `@Transient`사용==
- 속성명이 카멜케이스여도 실제 테이블에는 `_`로 단어가 구분되어 저장
- 엔티티에는 Setter가 없어야 안전함
	- 엔티티를 생성할 경우 lombok의 `@Builder`로 빌드패턴 사용
	- 데이터를 변경해야 할 경우 그에 해당되는 메서드를 엔티티에 추가

| 속성             |                                      | 기본값    |
| ---------------- | ------------------------------------ | --------- | 
| name             | 필드와 매핑할 컬럼 이름              | 필드 이름 |     
| nullable         | 컬럼의 null 허용 여부                | true      |     
| unique           | 컬럼의 유일한 값 여부                | false     |     
| columnDefinition | 컬럼 속성 정의. default값 줄 수 있음 |           |      
| length           | 컬럼의 길이 설정                     |           |    
- `columnDefinition = "TEXT"`: 컬럼의 글자 수를 제한할 수 없는 경우

---
## `@JoinColumn`
[참고1](https://ksh-coding.tistory.com/105#%E2%9C%85%202-2-1.%20OneToOne%20%2F%20ManyToOne%20%3A%20Source%20Entity(Table)%EC%97%90%20FK%20%EC%9C%84%EC%B9%98-1)
[참고2](https://hyeon9mak.github.io/omit-join-column-when-using-many-to-one/)
정리하기

---
## `cascade`
[참고](https://data-make.tistory.com/668)
정리하기

---
# JPA 연관 관계
## 단방향 vs 양방향
- JPA는 객체와 데이터베이스 테이블을 매핑시킴
- 데이터베이스는 foreign key 하나로 양 쪽 테이블 조인이 가능해 단방향, 양방향 나눌 필요 없음
- 반대로 객체는 참조용 필드가 있는 객체만 다른 객체를 참조할 수 있음
- 두 객체 중 하나만 참조용 필드를 가지면 단방향 관계
- 두 객체 모두가 각각 참조용 필드를 가지면 양방향 관계(엄밀하게는 두 객체가 단방향 참조를 각각 가져서 양방향 관계처럼 사용하고 말하는 것)
- 비즈니스 로직에서 두 객체 사이의 참조가 필요한지 고민하고 어떤 연관 관계를 결정
	- `Grammar.getGrammarBook()`처럼 참조가 필요하면 Grammar -> GrammarBook 단방향 참조
	- `GrammarBook.getGrammar()`처럼 참조가 필요하면 GrammarBook -> Grammar 단방향 참조
- 무조건 양방향 참조를 하면 하나의 엔티티가 수많은 엔티티와 불필요한 연관 관계를 맺어 복잡해짐
- 기본적으로 단방향 매핑으로 하고 나중에 역방향으로 객체 탐색이 꼭 필요하다고 느낄 때 추가하기
## 연관 관계의 주인
- 양방향 매핑 시 두 객체들 중 연관 관계의 주인을 지정해야함
- 연관 관계의 주인을 지정하는 것은 두 단방향 관계 중, FK를 비롯한 테이블 레코드를 저장, 수정, 삭제 등의 처리를 할 수 있는게 무엇인지 JPA에게 알려주는 것
- **연관 관계의 주인이 아니면 조회만 가능함**
- 연관 관계의 주인이 아닌 객체에서 `mappedBy`속성으로 주인을 지정해야함
> [!TIP]
>FK가 있는 곳을 연관 관계의 주인으로 정하면 됨 
- 연관 관계의 주인을 지정하는 이유: 양방향 연관 관계의 관리 포인트가 두 개면 Grammar의 GrammarBook을 수정하려 할때, Grammar객체에서 `setGrammarBook()`같은 메서드로 수정할지 GrammarBook객체에서 `getGrammar()`같은 메서드로 수정할지 어떤 메서드에서 FK를 수정할지 JPA는 모름. 연관 관계의 주인을 Grammar로 지정하면 Grammar에서 GrammarBook을 수정할때만 FK를 수정하겠다고 JPA한테 알려줄 수 있음
- 데이터베이스에서는 FK가 있는 테이블을 수정하려면 연관 관계의 주인만 수정하면 되지만, 각각 참조하는 객체 입장에서는 데이터 동기화를 위해 둘 다 수정해줘야 함
## N : 1 단방향
```java
@Entity
@Getter
public class Grammar {
	@Id  
	@GeneratedValue(strategy = GenerationType.IDENTITY)  
	private Long id;  
	  
	@Column  
	private String sentence;  
	  
	@ManyToOne(optional = false)  
	private GrammarBook grammarBook;
}
```
```java
@Getter  
@Entity  
@DynamicInsert  
public class GrammarBook {  
  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
  
    @Column  
    @ColumnDefault(value = "일반")  
    private String name;
}
```
- N 쪽인 Grammar에서만 `@ManyToOne` 추가
- 1 쪽인 GrammarBook은 Grammar객체를 참조하지 않음
## N : 1 양방향
```java
@Getter  
@Entity  
public class Grammar {  
  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
  
    @Column  
    private String sentence;  
  
    @ManyToOne(optional = false)  
    @JoinColumn(name = "GRAMMAR_BOOK_ID")  
    private GrammarBook grammarBook;  
}
```
```java
@Getter  
@Entity  
@DynamicInsert  
public class GrammarBook {  
  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
  
    @Column  
    @ColumnDefault(value = "일반")  
    private String name;  
  
    @OneToMany(mappedBy = "grammarBook", cascade = CascadeType.ALL)  
    private List<Grammar> grammars;  
}
```
- N 쪽은 그대로
- 1 쪽에 `@OneToMany`추가하고 `mappedBy`값은 N 쪽에서 `@ManyToOne` 지정한 필드명 똑같이!
> [!tip]
> `mappedBy`를 가진 엔티티는 매핑된 엔티티, 즉 연관 관계의 주인을 조회만 할 수 있음
## 1 : N 단방향 (안쓴다고 생각해라)
- 데이터베이스는 무조건 N 쪽에서 FK 관리하지만 1 : N은 1쪽에서 N쪽 객체를 조작함
- 1쪽에서는 FK를 저장할 방법이 없어 조인 및 업데이트 쿼리가 발생함
- 1 : N 양방향은 공식적으로 존재하지 않지만 구현할 수는 있음
- **실무에서는 1 : N 거의 쓰지 않지만**, JPA 값 타입을 사용하는 것을 대신하여 사용하는 경우 등 일부 유용한 경우 있음
```java
@Getter  
@Entity  
public class Grammar {  
  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
  
    @Column  
    private String sentence;
}
```
```java
@Getter  
@Entity  
@DynamicInsert  
public class GrammarBook {  
  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
  
    @Column  
    @ColumnDefault(value = "일반")  
    private String name;  
  
    @OneToMany
    @JoinColumn(name = "GRAMMAR_ID")  
    private List<Grammar> grammars;  
}
```
- 새로운 GrammarBook 레코드를 저장할때, GrammarBook은 Grammar테이블의 FK(GRAMMAR_ID)를 저장할 방법이 없기 때문에 조인 및 업데이트 쿼리(`.getGrammars().add(grammar)`)가 발생함
## 1 : 1
- 1 : 1 양방향은 똑같이 `@OneToOne` 설정하고 `mappedBy` 설정해서 읽기 전용으로 만들면 됨
- 1 : 1 단방향은 주 테이블이 FK를 갖는 경우와 대상 테이블이 FK를 갖는 경우가 있음
	- 주 테이블이 FK를 가지면 주 테이블을 조회할때마다 FK를 이용할 수 있으므로 성능상 이득임
	- 대상 테이블이 FK를 가지면 후에 대상 테이블: 주 테이블 = N : 1로 바뀔때 유리함
## N : N (실무 사용 금지)
- 중간 테이블이 자동생성되 자기도 모르는 복잡한 조인 쿼리가 발생할 수 있음
- 중간 테이블은 두 테이블의 FK만 저장되지만 다른 정보가 추가되는 경우가 많기 때문에 문제가 될 확률이 높음
- N : N --> 1 : N, N : 1로 풀고 중간 테이블을 엔티티로 만드는게 추후 변경에도 유연하게 대처할 수 있음
# Repository
- 엔티티에 의해 생성된 데이터베이스 테이블에 접근하는 메서드들을 사용하기 위한 **인터페이스**
- CRUD를 어떻게 처리할지 정의하는 계층
- 레포지터리명: 엔티티명+Repository
- JpaRepository 인터페이스를 상속해야함
- JpaRepository 인터페이스를 상속할때 제네릭타입으로 `<엔티티 타입, 엔티티의 PK 속성 타입>` 지정해야함
```java
//Question 엔티티의 레포지토리
import org.springframework.data.jpa.repository.JpaRepository;

public interface QuestionRepository extends JpaRepository<Question, Integer> {} 
```
- DI에 의해 스프링이 자동으로 레포지토리 객체를 생성하며, 이때 프록시 패턴이 사용됨
- 레포지토리 객체의 메서드가 실행될때 JPA가 해당 메서드명을 분석해 쿼리를 만들고 실행함
- `레포지토리.count()`: 해당 레포지토리의 총 데이터 개수 반환
## JPA 메서드 네이밍 룰
|키워드|예시|JPQL snippet|
|---|---|---|
|And|`findByLastnameAndFirstname`|… where x.lastname = ?1 and x.firstname = ?2|
|Or|`findByLastnameOrFirstname`|… where x.lastname = ?1 or x.firstname = ?2|
|Is, Equals|`findByFirstname`,`findByFirstnameIs`,`findByFirstnameEquals`|… where x.firstname = ?1|
|Between|`findByStartDateBetween`|… where x.startDate between ?1 and ?2|
|LessThan|`findByAgeLessThan`|… where x.age < ?1|
|LessThanEqual|`findByAgeLessThanEqual`|… where x.age <= ?1|
|GreaterThan|`findByAgeGreaterThan`|… where x.age > ?1|
|GreaterThanEqual|`findByAgeGreaterThanEqual`|… where x.age >= ?1|
|After|`findByStartDateAfter`|… where x.startDate > ?1|
|Before|`findByStartDateBefore`|… where x.startDate < ?1|
|IsNull, Null|`findByAge(Is)Null`|… where x.age is null|
|IsNotNull, NotNull|`findByAge(Is)NotNull`|… where x.age not null|
|Like|`findByFirstnameLike`|… where x.firstname like ?1|
|NotLike|`findByFirstnameNotLike`|… where x.firstname not like ?1|
|StartingWith|`findByFirstnameStartingWith`|… where x.firstname like ?1 (parameter bound with appended %)|
|EndingWith|`findByFirstnameEndingWith`|… where x.firstname like ?1 (parameter bound with prepended %)|
|Containing|`findByFirstnameContaining`|… where x.firstname like ?1 (parameter bound wrapped in %)|
|OrderBy|`findByAgeOrderByLastnameDesc`|… where x.age = ?1 order by x.lastname desc|
|Not|`findByLastnameNot`|… where x.lastname <> ?1|
|In|`findByAgeIn(Collection<Age> ages)`|… where x.age in ?1|
|NotIn|`findByAgeNotIn(Collection<Age> ages)`|… where x.age not in ?1|
|True|`findByActiveTrue()`|… where x.active = true|
|False|`findByActiveFalse()`|… where x.active = false|
|IgnoreCase|`findByFirstnameIgnoreCase`|… where UPPER(x.firstame) = UPPER(?1)|
- OrderBy + 속성명 + Asc(오름차순)/Desc(내림차순)
- ==**응답 결과가 여러 건이면 메서드의 리턴타입을 `List<>`로 해야함**==
## 데이터 조회하기
- `findAll()`: 데이터베이스의 모든 데이터 조회
- `findById(int id)`: id값으로 데이터 조회. id는 1부터 시작. `Optional`을 반환하므로 `isPresent()`로 null이 아닌지 확인한 후에 `get()`으로 실제 엔티티 객체를 얻음
```java
@Test  
void findQuestionTest() {  
    Optional<Question> question = this.questionRepository.findById(1);  
       question.ifPresent(value -> assertThat(value.getSubject())  
          .isEqualTo("what is the jpa?"));  
}
```
- `findBy+엔티티 속성명()`: 해당 속성의 값으로 데이터 조회
## 데이터 수정하기
1. findBy메서드로 수정할 엔티티 객체 찾기
2. 엔티티 객체의 setter를 통해 값 수정
3. 레포지토리.save(수정된 엔티티 객체)
```java
Optional<Question> oq = this.questionRepository.findById(1);  
if (oq.isPresent()) {
	Question q = oq.get();   
	q.setSubject("수정된 제목");  
	questionRepository.save(q);
}
```
## 데이터 삭제하기
1. findBy메서드로 삭제할 엔티티 객체 찾기
2. 레포지토리.delete(1에서 찾은 엔티티 객체)


# JPA Auditing
[참고](https://webcoding-start.tistory.com/53)
`@EnableJpaAuditing`
- 엔티티 객체가 생성또는 수정되었을때 자동으로 엔티티 객체의 특정 필드값을 변경할수 있도록 JPA Auditing 활성화
- `@CreatedDate`, `@LastModifiedDate`등이 자동으로 업데이트됨

`@EntityListeners(AuditingEntityListener.class)`
- 적용된 클래스에 Auditing 기능을 포함시킴