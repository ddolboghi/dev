# JPA
- ORM(object-relational mapping)을 이용하면 데이터베이스의 값을 객체처럼 사용할 수 있음
- 자바의 표준은 JPA(java persistence API)
- JPA는 인터페이스므로 이를 구현하는 ORM프레임워크를 선택해야함
- hibernate : JPA를 구현하는 ORM프레임워크. 내부적으로 JDBC API사용
---
## entity
- 데이터베이스의 테이블과 매핑되는 객체
- 데이터베이스와 직접 연결되어 데이터베이스에 영향을 미치는 쿼리를 실행
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
### 지연 로딩(lazy loading)
- 쿼리로 요청한 데이터를 애플리케이션에 바로 로딩하지 않고 필요할때 쿼리를 날려 데이터 조회
- <>즉시 로딩 : 조회할때 쿼리를 보내 연관된 모든 데이터를 가져옴
---
# spring data
- 데이터베이스 사용기능을 클래스 레벨에서 추상화
- CRUD를 포함한 여러 메서드 포함
- 알아서 쿼리 만들어줌
- 퍼이징 처리 기능
- 메서드 이름으로 자동으로 쿼리 빌딩
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
자주 사용하는 쿼리 메서드

|코드| |쿼리|
|---|---|---|
|`findByName()`|"name"컬럼의 값 중 파라미터로 들어오는 값과 같은 데이터 반환|`...WHERE name = ?1`
|`findByNameAndAge()`|파라미터로 들어오는 값 중 첫번째 값은 "name"컬럼에서 조회하고, 두번째 값은 'age'컬럼에서 조회한 데이터 반환|`...WHERE name = ?1 AND age=?2`|
|`findByNameOrAge()`|파리미터로 들어오는 값 중 첫번째 값이 "name"컬럼에서 조회되거나 두번째 값이 "age"에서 조회되는 데이터 반환|`...WHERE name = ?1 AND age=?2`|
|`findByAgeLessThan()`|"age"컬럼의 값 중 파라미터로 들어온 값보다 작은 데이터 반환|`...WHERE age < ?1`|
|`findByAgeFreaterThan()`|'age'컬럼의 값 중 파라미터로 들어온 값보다 큰 데이터 반환|`...WHERE age > ?1`|
|`findByName(Is)Null()`|"name"컬럼의 값 중 null인 데이터 반환|`...WHERE name IS NULL`

---
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
`@Entity`
- Member객체를 JPA가 관리하는 엔티티로 지정해 실제 데이터베이스의 테이블과 매핑시킴
- name속성으로 name값을 가진 테이블 이름과 매핑
- name지정안하면 클래스 이름과 같은 테이블 매핑
---
`@NoArgsConstructor(access = AccessLevel.PROTECTED)`
- protected 기본 생성자 생성
- 엔티티는 반드시 기본 생성자가 있어야함
---
`@GeneratedValue(strategy = GenerationType.IDENTITY)`
- 기본키의 생성 방식 결정

|strategy| |
|---|---|
|AUTO|선택한 데이터베이스 방언에 따라 방식을 자동 선택|
|IDENTITY|기본키 생성을 데이터베이스에 위임(=AUTO_INCREMENT)|
|SEQUENCE|데이터베이스 시퀀스를 사용해서 기본 키를 할당. 오라클에서 주로 사용|
|TABLE|키 생성 테이블 사용|

---
`@Column`

|속성| |기본값|
|---|---|---|
|name|필드와 매핑할 컬럼 이름|필드 이름
|nullable|컬럼의 null 허용 여부|true|
|unique|컬럼의 유일한 값 여부|false|
|columnDefinition|컬럼 정보 설정. default값 줄 수 있음|

