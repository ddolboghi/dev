# 스프링부트에서 JPA 사용 설정
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
# 데이터베이스 종류 설정
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.H2Dialect  
# 엔티티를 기준으로 테이블을 생성하는 규칙 정의
spring.jpa.hibernate.ddl-auto=update
```
- JPA는 설정한 데이터베이스 종류에 맞는 SQL을 만듬
- 테이블 생성 규칙
	- none: 엔티티가 변경되도 데이터베이스 변경 안함
	- update: 엔티티의 변경된 부분만 적용(ALTER)
	- validate: 변경사항이 있는지 검사만 함
	- create: 애플리케이션을 실행할때 모두 drop하고 다시 생성(DROP -> CREATE)
	- create-drop: create와 동일, 종료할때도 모두 drop(DROP -> CREATE -> DROP)
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
# JPA
- ORM(object-relational mapping)은 VO가 가진 정보를 데이터베이스가 아닌 java.util.Map 같은 컬렉션에 저장하는 것과 유사한 개념으로, SQL을 작성하지 않고도 테이블을 객체로 조작 가능하게 함
- 자바의 ORM 표준은 JPA(Java Persistence API)
- JPA는 인터페이스 모음이므로 이를 구현하는 ORM프레임워크를 선택해야함(e.g. hibernate)
- JPA는 자바 애플리케이션과 JDBC 사이에 존재하면서 JDBC의 복잡한 데이터베이스 연동 작업을 대신 처리해줌
- JPA는 데이터베이스 연동뿐만 아니라 SQL까지 제공해줌
# spring data JPA
- 기존에는 메서드 호출로 엔티티 상태를 관리했지만 spring data JPA를 사용하면 repository역할을 하는 인터페이스를 만들어 데이터베이스 CRUD 작업을 쉽게 할 수 있음
- JPA는 메서드 규칙에따라 메서드를 선언하면 이름을 분석해 자동으로 쿼리 생성
# 영속성 컨텍스트(Persistence Context)
- 영속성 컨텍스트는 엔티티 객체를 관리하는 가상의 공간
- 엔티티 객체가 영속성 컨텍스트에 들어오면 JPA는 엔티티 객체의 매핑 정보를 데이터베이스에 반영
- 영속 객체(Persistence Object): 영속성 컨텍스트에 들어와 JPA의 관리 대상이 된 엔티티 객체
- Flush: 영속성 컨텍스트에 저장된 엔티티를 데이터베이스에 반영하는 과정
- 영속성 컨텍스트의 생명주기: 데이터베이스에 접근하기 위한 세션이 생성되면 영속성 컨텍스트가 만들어지고, 세션이 종료되면 영속성 컨텍스트도 없어짐
## 1차 캐시
- 영속성 컨텍스트 내부에 있는 Map같은 컬렉션
- 캐시의 key는 (`@Id`)PK 필드고, value는 엔티티 객체
- 1차 캐시에 저장된 엔티티는 곧바로 실제 데이터베이스에 반영되지 않고, 트랜잭션을 종료할때 반영됨
- 엔티티 조회시 1차 캐시에서 먼저 조회하고 있으면 반환, 없으면 데이터베이스에서 조회해 1차 캐시에 저장한 다음 반환
- 캐시된 데이터를 조회할때는 데이터베이스를 거치지 않아 빠름
## 쓰기 지연(transactional write-behind)
- 트랜잭션을 커밋하기 전까지는 데이터베이스에 실제로 쿼리를 보내지 않고 모았다가 트랜잭션을 커밋하면 모았던 쿼리를 한번에 실행
- 적당한 묶음으로 쿼리 요청할 수 있어 데이터베이스 시스템의 부담 줄임
## 변경 감지
1. 트랜잭션을 커밋하면 1차 캐시에 저장된 엔티티의 값과 현재 엔티티의 값을 비교
2. 변경된 값이 있다면 변경 사항을 감지해 데이터베이스에 변경 값을 자동으로 반영
- 적당한 묶음으로 쿼리 요청할 수 있어 데이터베이스 시스템의 부담 줄임
## 지연 로딩(lazy)
- 쿼리로 요청한 데이터를 한번에 다 가져오지 않고 필요할때 쿼리를 날려 데이터를 가져옴
  <>즉시 로딩(eager) : 조회할때 한 방 쿼리를 보내 연관된 모든 데이터를 가져옴
- 대부분의 비즈니스 로직에서 연관 관계에 있는 객체를 함께 사용한다면 lazy 로딩시 select 쿼리가 2번 발생해 손해임
- 하지만 **실무에서는 거의 lazy 로딩만 사용함**
- 왜냐하면 eager 로딩시 예상치 못한 쿼리와 N+1 문제가 발생함
- 자주 함께 사용하는 연관 관계의 객체들은 lazy 로딩시 N+1문제가 발생함 --> **JPQL의 fetch join으로 해당 시점에 한 방 쿼리로 가져옴**
- 테스트 코드에서 여러 레포지토리 사용할때 하나의 레포지토리를 사용한 뒤에는 DB세션이 끊겨 **LazyInitializationException** 발생 --> 테스트 메서드에 `@Transactional` 붙여 메서드가 종료될때까지 DB세션 유지
- 실제 서버에서 JPA 프로그램들을 실행할 때는 DB세션이 종료되지 않음
# entity
- 데이터베이스의 테이블과 매핑되는 자바 클래스
- 데이터베이스와 직접 연결되어 데이터베이스에 영향을 미치는 쿼리를 실행
- 모델 또는 도메인 모델이라 부르기도 함
# entity manager
- 데이터베이스에 접근해서 CRUD 수행
- **엔티티 매니저는 엔티티를 영속성 컨텍스트에 추가해 영속 객체로 만들고, 영속성 컨텍스트와 데이터베이스를 비교하면서 실제 데이터베이스를 대상으로 작업 수행**
- 엔티티 매니저 팩토에서 엔티티 매니저 만듬
- 스프링 부트는 내부적으로 엔티티 매니저 팩토리를 하나만 생성해서 관리하고 @PersistenceContext나 @Autowired로 프록시 엔티티 매니저 사용
- 엔티티 매니저가 필요할때 데이터베이스 트랜잭션과 관련된 실제 엔티티 매니저 호출
- 엔티티 매니저는 Spring Data JPA에서 관리하므로 개발자가 직접 생성하거나 관리할 필요 없음
## 엔티티의 상태
- new(비영속) : 영속성 컨텍스트에 추가되지 않은 엔티티 객체의 상태
- managed(영속) : 영속성 컨텍스트가 관리하는 상태
- detached(준영속) : 영속성 컨텍스트에 의해 관리되던 엔티티 객체가 컨텍스트와 분리된 상태
- removed : 영속성 컨텍스트에서 엔티티가 제거되고 테이블의 레코드도 삭제된 상태
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
## 엔티티 클래스에 사용되는 애너테이션
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
### `@Entity`
- Member객체를 JPA가 관리하는 엔티티로 지정해 실제 데이터베이스의 테이블과 매핑시킴
- name속성으로 name값을 가진 테이블 이름과 매핑
- name지정안하면 클래스 이름과 같은 테이블 매핑
### `@NoArgsConstructor(access = AccessLevel.PROTECTED)`
- protected 기본 생성자(인자가 없음) 생성
- **엔티티는 반드시 기본 생성자가 있어야함**
### `@Id`
- 해당 속성을 테이블의 기본 키(primary key)로 지정
### `@GeneratedValue(strategy = GenerationType.[strategy])`
- 해당 속성에 값을 따로 세팅하지 않아도 1씩 자동으로 증가하여 저장됨
- strategy: 기본 키의 생성 방식 결정
- strategy옵션을 생략하면 해당 컬럼이 모두 동일한 시퀀스로 번호 생성하므로 보통 `GenerationType.IDENTITY`사용

|strategy| |
|---|---|
|AUTO|선택한 데이터베이스 방언에 따라 방식을 자동 선택|
|IDENTITY|기본키 생성을 데이터베이스에 위임(=AUTO_INCREMENT)|
|SEQUENCE|데이터베이스 시퀀스를 사용해서 기본 키를 할당. 오라클에서 주로 사용|
|TABLE|키 생성 테이블 사용|
### `@Column`
- 엔티티 클래스의 속성은 `@Column`을 사용하지 않아도 테이블 컬럼으로 인식함
- ==테이블 컬럼으로 인식하고 싶지 않은 경우에만 `@Transient`사용==
- 속성명이 카멜케이스여도 실제 테이블에는 `_`로 단어가 구분되어 저장
- 엔티티에는 Setter가 없어야 안전함
	- 엔티티 객체를 생성해야할 경우 lombok의 `@Builder`로 빌드패턴 사용하기
	- 데이터를 변경해야 할 경우 그에 해당되는 메서드를 엔티티에 추가하기

| 속성             |                                      | 기본값    |
| ---------------- | ------------------------------------ | --------- | 
| name             | 필드와 매핑할 컬럼 이름              | 필드 이름 |     
| nullable         | 컬럼의 null 허용 여부                | true      |     
| unique           | 컬럼의 유일한 값 여부                | false     |     
| columnDefinition | 컬럼 속성 정의. default값 줄 수 있음 |           |      
| length           | 컬럼의 길이 설정                     |           |    
- `columnDefinition = "TEXT"`: 컬럼의 글자 수를 제한할 수 없는 경우
### `@JoinColumn`
[참고1](https://ksh-coding.tistory.com/105#%E2%9C%85%202-2-1.%20OneToOne%20%2F%20ManyToOne%20%3A%20Source%20Entity(Table)%EC%97%90%20FK%20%EC%9C%84%EC%B9%98-1)
[참고2](https://hyeon9mak.github.io/omit-join-column-when-using-many-to-one/)
정리하기
### `CascadeType`
- `ALL`: 상위 엔티티에서 하위 엔터티로 모든 작업 전파, 아래 모든 작업 포함
- `PERSIST`: 하위 엔티티까지 영속성 전달 = 상위 엔티티 저장하면 하위 엔티티도 저장
- `MERGE`: 하위 엔티티까지 병합 수행 = 조회한 후 업데이트
- `REMOVE`: 상위 엔티티 제거하면 연결된 하위 엔티티까지 제거
- `REFRESH`: 데이터베이스로부터 인스턴스값 다시 읽어옴(새로 고침) --> 연결된 하위 엔티티까지 인스턴스값 새로고침
- `DETACH`: 상위 엔티티를 영속성 컨텍스트에서 제거하면 연결된 하위 엔티티까지 제거

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
- JpaRepository 인터페이스는 스프링 데이터의 PagingAndSortingRepository인터페이스를 상속받음
```java
//Question 엔티티의 레포지토리
import org.springframework.data.jpa.repository.JpaRepository;

public interface MyEntityRepository extends JpaRepository<엔티티, 엔티티 PK의 데이터 타입> {} 
```
- DI에 의해 스프링이 자동으로 레포지토리 객체를 생성하며, 이때 프록시 패턴이 사용됨
- 레포지토리 객체의 메서드가 실행될때 JPA가 해당 메서드명을 분석해 쿼리를 만들고 실행함
- `레포지토리.count()`: 해당 레포지토리의 총 데이터 개수 반환
## 간단한 CRUD 예시
### 조회
- `findAll()`: 데이터베이스의 모든 데이터 조회
- `findById(int id)`: id값으로 데이터 조회. id는 1부터 시작. `Optional`을 반환하므로 null이 아닌지 확인한 후에 `get()`으로 실제 엔티티 객체를 얻어야함
```java
@Test  
void findQuestionTest() {  
    Optional<Question> question = this.questionRepository.findById(1);  
       question.ifPresent(value -> assertThat(value.getSubject())  
          .isEqualTo("what is the jpa?"));  
}
```
- `findBy+엔티티 속성명()`: 해당 속성의 값으로 데이터 조회
### 수정
1. findBy메서드로 수정할 엔티티 객체 찾기
2. 엔티티 객체에 미리 정의한 메서드를 통해 값 수정
3. 레포지토리.save(수정된 엔티티 객체)
```java
Optional<Question> oq = this.questionRepository.findById(1);  
if (oq.isPresent()) {
	Question q = oq.get();   
	q.setSubject("수정된 제목");  
	questionRepository.save(q);
}
```
### 삭제
1. findBy메서드로 삭제할 엔티티 객체 찾기
2. 레포지토리.delete(1에서 찾은 엔티티 객체)

# JPQL(JPA Query Language)
- JPA에서 사용할 수 있는 쿼리로, SQL과 매우 비슷함
- 테이블이나 컬럼명 대신 엔티티명과 필드명 사용
## 쿼리 메서드
- Repository에 별도로 정의한 메서드
- `리턴타입 주제+By+서술어(인자)` 구조
- 일부 주제와 By 사이에 All 추가해 여러 반환값 가져올 수 있음(e.g. findAllBy)

| 주제 키워드 |  |
| ---- | ---- |
| findBy / readBy / getBy / queryBy / searchBy / streamBy | 조회 |
| existsBy | - 특정 데이터가 존재하는지 여부<br>- boolean 타입 반환 |
| countBy | 조회 쿼리 결과로 나온 레코드의 개수 반환 |
| deleteBy / removeBy | - 삭제 쿼리<br>- deleteBy는 리턴 타입 없음<br>- removeBy는 long 타입 리턴 |
| ...First{number}... / ...Top{number}... | - 조회 쿼리의 리턴 개수 제한<br>- 주제와 By 사이에 위치<br>- 한 번에 여러 건 조회할때 사용<br>- 단 건 조회하려면 number 생략 |

|조건 키워드 | |예시 |
|---|---|---|
|And|`findByLastnameAndFirstname`|… where x.lastname = ?1 and x.firstname = ?2|
|Or|`findByLastnameOrFirstname`|… where x.lastname = ?1 or x.firstname = ?2|
|Is, Equals|`findByFirstname`<br>`findByFirstnameIs`<br>`findByFirstnameEquals` |… where x.firstname = ?1|
|Between|`findByStartDateBetween`|… where x.startDate between ?1 and ?2|
|LessThan |`findByAgeLessThan`|… where x.age < ?1|
|LessThanEqual|`findByAgeLessThanEqual`|… where x.age <= ?1|
|GreaterThan |`findByAgeGreaterThan`|… where x.age > ?1|
|GreaterThanEqual|`findByAgeGreaterThanEqual`|… where x.age >= ?1|
|After|`findByStartDateAfter`|… where x.startDate > ?1|
|Before|`findByStartDateBefore`|… where x.startDate < ?1|
|(Is)Null |`findByAge(Is)Null`|… where x.age is null|
|(Is)NotNull |`findByAge(Is)NotNull`|… where x.age not null|
|Like|`findByFirstnameLike`|… where x.firstname like ?1|
|NotLike|`findByFirstnameNotLike`|… where x.firstname not like ?1|
|StartingWith<br>(=StartsWith) |`findByFirstnameStartingWith`|… where x.firstname like ?1 (parameter bound with appended %)|
|EndingWith<br>(=EndsWith) |`findByFirstnameEndingWith`|… where x.firstname like ?1 (parameter bound with prepended %)|
|Containing<br>(=Contains) |`findByFirstnameContaining`|… where x.firstname like ?1 (parameter bound wrapped in %)|
|OrderBy|`findByAgeOrderByLastnameDesc`|… where x.age = ?1 order by x.lastname desc|
|Not|`findByLastnameNot`|… where x.lastname <> ?1|
|In|`findByAgeIn(Collection<Age> ages)`|… where x.age in ?1|
|NotIn|`findByAgeNotIn(Collection<Age> ages)`|… where x.age not in ?1|
|True|`findByActiveTrue()`|… where x.active = true|
|False|`findByActiveFalse()`|… where x.active = false|
|IgnoreCase|`findByFirstnameIgnoreCase`|… where UPPER(x.firstame) = UPPER(?1)|
- OrderBy + 필드명 + Asc(오름차순)/Desc(내림차순)
- **응답 결과가 여러 건이면 메서드의 리턴타입을 `List<>`로 해야함**
## `@Query`
- 직접 쿼리 작성하기 위한 에너테이션
- 엔티티 객체가 아닌 원하는 컬럼의 값만 추출할 수 있음. 이때 메서드의 리턴 타입은 Object 배열의 리스트 형태여야함
```java
@Query("SELECT p.name, p.price, p.stock FROM Product p WHERE p.name = :name")
List<Object[]> findByNameParam(@Param("name") String name);
```
- `@Query`는 문자열을 입력하므로 컴파일 시점에 에러를 잡지 못하고 런타임 에러 발생할 수 있음 --> QueryDSL 사용하면 컴파일 시점에 에러 잡을 수 있음
## QueryDSL
- 코드로 쿼리 생성해주는 프레임워크(QueryDSL이 제공하는 Fluent API를 활용해 쿼리 생성)
- IDE의 코드 자동 완성 기능 사용 가능
- 문법적으로 잘못된 쿼리 허용 안함
- 고정된 SQL 쿼리를 작성하지 않아 동적으로 쿼리 생성
- QClass를 사용해 쿼리를 작성하면 도메인 타입과 프로퍼티를 안전하게 참조할 수 있음
### 의존성 추가
```gradle
configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

dependencies {
	implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'
	annotationProcessor "com.querydsl:querydsl-apt:${dependencyManagement.importedProperties['querydsl.version']}:jakarta"
	annotationProcessor 'jakarta.persistence:jakarta.persistence-api'
	annotationProcessor 'jakarta.annotation:jakarta.annotation-api'
}

def querydslDir = '$buildDir/generated/querydsl'
sourceSets {
  main.java.srcDirs += [ querydslDir ]
}

tasks.withType(JavaCompile) {
	options.getGeneratedSourceOutputDirectory().set(file(querydslDir))
}

clean.doLast {
  file(querydslDir).deleteDir()
}
```
- `implementation 'com.querydsl:querydsl-jpa'`: QueryDSL을 사용하기 위한 라이브러리, 실제 쿼리를 위해 사용되는 QClass는 생성되지 않음
- `annotationProcessor`: QClass를 생성하기 위한 애너테이션 설정, Q 파일을 찾지 못해서 발생하는 에러 방지(persistence-api, annotation-api)
- `sourceSets {...}`:  빌드시 엔티티로 등록한 클래스들이 QClass로 생성된 후 저장되는 곳 지정
- `compileJava {...}`: 해당 내용 명시해주지 않으면 Q파일 내 Generated를 import할때 java9에만 있는 javax.annotation.processing.Generated로 import하기때문에, 다른 버전에서도 사용할 수 있도록 java.annotation.Generated로 import하도록 설정
- `tasks.withType...`: query QClass 파일 생성 위치 지정
- `clean.doLast {...}`: gradle clean시 충돌 막기위해 마지막에 QClass 디렉터리 삭제
### 설정 클래스 추가
- QueryDSL을 사용하려면 엔티티 매니저를 받아 생성된 `JPAQuery`나 `JPAQueryFactory`객체를 활용해야함
	- `JPAQueryFactory`는 `select()`, `selectFrom()` 메서드 사용 가능
```java
@Configuration
public class QueryDSLConfiguration {
	@PersistenceContext
	private EntityManager entityManager;

	@Bean
	public JPAQueryFactory jpaQueryFactory() {
		return new JPAQueryFactory(entityManager);
	}
}
```
### 사용법
1. QueryDSL로 작성할 메서드를 위한 커스텀 레포지토리 인터페이스 만들기 
2. 구현 클래스에 QueryDSL로 메서드 구현하기
3. 기존 레포지토리에서 커스텀 레포지토리 인터페이스 상속 받기
4. 이러면 기존 레포지토리로 접근해도 QeuryDSL로 정의한 메서드 사용 가능!
- QueryDSL > 4.0.1부터는 List타입 값을 리턴 받으려면 `fetch()`대신 `list()`메서드 사용해야함 --> 꼭 그런건 아닌 것 같음
- `T fetchOne()`: 단 건의 조회 결과 반환
- `T fetchFirst()`: 여러 건의 조회 결과 중 처음 1건만 반환
- `Long fetchCount()`: 조회 결과의 개수 반환
- `QueryResult<T> fetchResults()`: 조회 결과 리스트와 개수를 포함한 QueryResults 반환
```java
public interface ProductRepositoryCustom {
	List<Product> findByName(String name);
}
```

```java
import static ...QProduct.product; //QClass 정적 임포트

@Repository
public class ProductRepositoryCustomImpl implements ProductRepositoryCustom {
	pricate final JPAQueryFactory jpaQueryFactory;
	
	public ProductRepositoryCustomImpl(JPAQueryFactory jpaQueryFactory) {
		this.jpaQueryFactory = jpaQueryFactory;
	}

	@Override
	public List<Product> findByName(String name) {
		return jpaQueryFactory.selectFrom(product)
				.where(product.name.eq(name))
				.fetch();
	}
}
```
### QuerydslPredicateExecutor 인터페이스 활용
- QueryDSL을 Predicate를 활용해 사용할 수 있게하는 인터페이스
	- Predicate: 표현식을 작성할 수 있게 QueryDSL에서 제공하는 인터페이스
- Repository에 `extends ... QuerydslPredicateExecutor<엔티티>`로 추가
- join이나 fetch 기능 사용할 수 없음
### QuerydslRepositorySupport 추상 클래스 활용
- 일반적으로 커스텀 레포지토리를 활용해 레포지토리를 구현하는 방식으로 사용
- QueryDSL 3.x 버전을 대상으로 만들어 JPAQueryFactory로 시작할 수 없음
- 쿼리 작성시 `from()`메서드로 시작해야함 --> QuerydslRepositorySupport를 상속받는 커스텀 클래스 만들어서 `select()`, `selectFrom()`으로 시작하게 작성 가능
- spring data sort 기능이 정상 동작하지 않음
- 메서드 체인이 끊김
- 안쓰는게 나아 보임
![[QuerydslRepositorySupport사용법]]
1. 커스텀 레포지토리에는 QueryDSL을 사용해 작성할 메서드들을 정의함
2. 기존 엔티티 레포지토리는 JpaRepository와  커스텀 레포지토리를 상속 받음
3. 커스텀 레포지토리 구현 클래스는 QuerydslRepositorySupport를 상속받아 정의한 메서드들을 QueryDSL로 구현함
	- QuerydslRepositorySupport를 상속받으면 생성자를 통해 도메인 클래스를 부모 클래스에 전달해야 함
4. 기존 엔티티 레포지토리로 접근하면 QueryDSL로 작성한 메서드 사용 가능
```java
public interface ProductRepositoryCustom {
	List<Product> findByName(String name);
}
```

```java
@Componen
public class ProductRepositoryCustomImpl extends QuerydslRepositorySupport implements ProductRepositoryCustom {
	
	public ProductRepositoryCustomImpl() {
		super(Product.class); //부모 클래스에 도메인 클래스 전달
	}

	@Override
	public List<Product> findByName(String name) {
		QProduct product = QProduct.product;
		return from(product)
				.where(product.name.eq(name))
				.select(product)
				.fetch();
	}
}
```

```java
@Repository
public interface ProductRepository extends JpaRepository<Product, Long>, ProductRepositoryCustom { }
```
## N + 1 문제 해결하기
[참고](https://cobbybb.tistory.com/18)
- 연관 관계에서 발생하는 이슈로, 연관 관계가 설정된 엔티티를 조회할 경우 조회된 데이터 개수(N)만큼 연관 관계의 조회 쿼리가 추가로 발생하는 것
	- 즉시로딩(eager)은 JPQL로 전달되는 과정에서 N개의 쿼리가 추가로 발생
	- 지연로딩(lazy)은 연관된 엔티티는 프록시로 캐싱해두고 이 엔티티를 사용할때 추가 쿼리 발생
- 일반 join: 오직 JPQL에서 조회 주체가 되는 엔티티만 SELECT하여 영속화
```java
@Query("SELECT distinct t FROM Team t join t.members") public List<Team> findAllWithMemberUsingJoin();
```

```sql
select
	distinct team_0.id as id1_1_,
	team0_.name as name2_1_
from
	team team0_
inner join
	member members1_
		on team0_.id=members1_.team_id
```
Team의 컬럼인 id와 name만 가져옴
### fetch join
- fetch join: 조회의 주체 엔티티뿐만 아니라 연관 엔티티도 SELECT하여 모두 영속화
- FetchType이 lazy인 엔티티를 참조하더라도 이미 영속성 컨텍스트에 들어있기 때문에 따로 쿼리가 실행되지 않은 채로 N+1 문제 해결​
- 페이징에 fetch join 적용하면 인메모리에 모든 값을 조회해서 가져와 out of memory 발생할 수 있음
#### `@Query`만 사용
```java
@Query("select gb from GrammarBook gb join fetch gb.grammars where gb.id = :id")  
Optional<GrammarBook> findById(@Param("id") Long id);
```
실행 쿼리:
```
select
        gb1_0.id,
        g1_0.grammar_book_id,
        g1_0.id,
        g1_0.sentence,
        gb1_0.name 
    from
        grammar_book gb1_0 
    join
        grammar g1_0 
            on gb1_0.id=g1_0.grammar_book_id 
    where
        gb1_0.id=?
```
#### `@EntityGraph` 사용
- JPQL로만 fetch join할때 하드 코딩 최소화
- `type = EntityGraph.EntityGraphType.FETCH`: 내부적으로 fetch join 사용, EntityGraph에 명시한 attribute는 eager로 패치하고 나머지는 lazy로 패치
- `type = EntityGraph.EntityGraphType.LOAD`: 내부적으로 일반 join 사용, 나머지 attribute는 엔티티에 명시한 fetch type이나 기본값으로 패치
```java
@EntityGraph(attributePaths = {"grammars"}, type = EntityGraph.EntityGraphType.FETCH)  
@Query("select gb from GrammarBook gb join gb.grammars where gb.id = :id")  
Optional<GrammarBook> findById(@Param("id") Long id);
```
실행 쿼리:
```
select
        gb1_0.id,
        g2_0.grammar_book_id,
        g2_0.id,
        g2_0.sentence,
        gb1_0.name 
    from
        grammar_book gb1_0 
    join
        grammar g1_0 
            on gb1_0.id=g1_0.grammar_book_id 
    left join
        grammar g2_0 
            on gb1_0.id=g2_0.grammar_book_id 
    where
        gb1_0.id=?
```
--> 중복 join 발생 --> `@EntityGraph`에서 내부적으로 fetch join하는데 `@Query`에서 또 join하기 때문 --> 엔티티 1개만 조회 목적이므로 `@Query`에서 join 사용 안해도 됨

수정된 코드
```java
@EntityGraph(attributePaths = {"grammars"}, type = EntityGraph.EntityGraphType.FETCH)  
@Query("select gb from GrammarBook gb where gb.id = :id")  
Optional<GrammarBook> findById(@Param("id") Long id);
```
실행 쿼리:
```
select
        gb1_0.id,
        g1_0.grammar_book_id,
        g1_0.id,
        g1_0.sentence,
        gb1_0.name 
    from
        grammar_book gb1_0 
    left join
        grammar g1_0 
            on gb1_0.id=g1_0.grammar_book_id 
    where
        gb1_0.id=?
```
--> left join된 결과에 대해 where 조건 적용되어 select

> [!TIP] SQL 실행 순서
> FROM → ON → JOIN → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → 
> ORDER BY 
> 
> 연산 논리 순서: ( ) → NOT → AND → OR
#### 둘 이상의 Collection fetch join(~ToMany) 불가능
- `~ToMany`에서 컬렉션 join시 fetch join한다면 인메모리에서 모든 값을 다 가져옴
  --> 컬렉션 join이 2개 이상이면 `MultipleBagFetchException` 발생
- 엔티티의 `~ToMany` 필드를 Set 자료형으로 바꾸기(마찬가지로 인메모리에서 가져옴)
- 필드에 `@BatchSize` 명시하기
	- List 자료구조를 꼭 사용해야할때
	- 2개 이상의 컬렉션 join & 페이징해야해서 out of memory 방지해야할때
	- batch size에 fetch join 걸면 안됨(fetch join이 우선시되어 batch size가 무시되기 때문)
> [!tip] fetch join 적용시 주의점
> 1. lazy 로딩 기본
> 2. 페이징 안하면 Set 자료구조로 `MultipleBagFetchException`방지
> 3. 페이징할때는 `@BatchSize`사용
## 정렬

## 페이징 처리
- **fetch join은 `~ToOne`관계에서만 사용**해야 모든 엔티티를 조회하지 않고 limit을 걸어 필요한 데이터만 가져옴
- `~ToMany`관계에서 collection fetch join하면 Pagination에서 개수 판단하기 힘들어 인메모리에서 조정함 --> **fetch join 사용X**, 조회할 컬렉션 필드에 대해서 `@BatchSize(size=조회할 주체의 개수)`걸어 해결
	- batch성 로딩 --> size만큼 한번에 가져와 지연로딩 방지
	- batch size는 최적화된 데이터 사이즈를 알기 힘들어 일반적으로 100~1000 사용, 확실하게 모르면 안좋음
- `@Fetch(FetchMode.SUBSELECT)`: `@BatchSize`와 같은 기능 but 한번에 **모든** 데이터 가져옴
# JPA Auditing
[참고](https://webcoding-start.tistory.com/53)
`@EnableJpaAuditing`
- 엔티티 객체가 생성또는 수정되었을때 자동으로 엔티티 객체의 특정 필드값을 변경할수 있도록 JPA Auditing 활성화
- `@CreatedDate`, `@LastModifiedDate`등이 자동으로 업데이트됨

`@EntityListeners(AuditingEntityListener.class)`
- 적용된 클래스에 Auditing 기능을 포함시킴

