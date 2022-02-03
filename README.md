# 실전 Spring Data JPA
## 예제 도메인 모델
*엔티티 클래스*
![](/img/entityClass.png)

 *Spring Data JPA가 구현 클래스 대신 생성*
![](/img/jpaRepoInterface.png)
- @Repository annotation은 생략가능
	- 컴포넌트 스캔을 Spring Data JPA가 자동으로 처리
	- JPA 예외를 스프링 예외로 변환하는 과정도 자동을 처리

  *공통 인터페이스 구성*
  ![](/img/commonInterface.png)

  *제네릭 타입*
  - T : 엔티티
  - ID : 엔티티의 식별자 타입
  - S : 엔티티와 그 자식 타입

  *주요 메서드*
  - save(S) : 새로운 엔티티는 저장하고 이미 있는 엔티티는 병합한다.
  - delete(T) : 엔티티 하나를 삭제한다. 내부에서 `EntityManager.remove()` 호출
  - findById(ID) : 엔티티 하나를 조회한다. 내붇에서 `EntityManager.find()` 호출
  - getOne(ID) : 엔티티를 프록시로 조회한다. 내부에서 `EntityManager.getReference()`호출
  - findAll(…) : 모든 엔티티를 조회한다. 정렬이나 페이징 조건을 파라미터로 제공할 수 있다.

  *스프링 데이터 JPA는 메소드 이름을 분석해서 JPQL을 생성하고 실행*

  *스프링 데이터 JPA가 제공하는 쿼리 메소드 기능*
  - 조회 : find…By, read…By, query…By
  - COUNT : count…By return long
  - EXIST : exist…By return boolean
  - DELETE : delete…By, remove…By long
  - DISTINCT : findDistinct
  - LIMIT : findFirst3, findFirst, findTop
  > 참고 : 이 기능은 엔티티의 필드명이 변경되면 인터페이스에 정의한 메서드 이름도 꼭 함께 변경해야 한다. 그렇지 않으면 애플리케이션을 시작하는 시점에 오류가 발생한다.
  > 이렇게 애플리케이션 로딩 시점에 오류를 인지할 수 있는 것이 스프링 데이터 JPA의 큰 장점이다.

  ## 스프링 데이터 JPA 페이징과 정렬

  *페이징과 정렬 파라미터*
  - `org.springframework.data.domain.Sort` : 정렬기능
  - `org.springframework.data.domain.Pageable` : 페이징 기능(내부에 sor 포함)

  *특별한 반환 타입*
  - `org.springframework.data.domain.Page` :  추가  count 쿼리 결과를 포함하는 페이징
  - `org.springframework.data.domain.Slice` : 추가 count 쿼리 없이 다음 페이지만 확인가능(내부적으로 limit+1 조회)
  - `List` : 추가 count 쿼리 없이 겨로가만 반환

  > 주의 : 페이지는 1이 아니라 0부터 시작된다.

  ## 벌크연산
  - @Query, @Modifying 사용해서 벌크연산 가능
  - 벌크 연산을 던지면 영속성 콘텍스트는 그대로기 때문에 조회 작업을 할경우에 clear 필요함
  - @Modifying(clearAutomatically = true) 옵션으로 벌크연산 후 클리어되게 할 수 있음.

  ## Entity Graph
  - fetch join 을 쉽게 해줌

  ## JPA Hint & Lock
  ### JPA Hint
  - JPA 쿼리 힌트(SQL 힌트가 아니라 JPA 구현체에게 제공하는 힌트)

  ## 사용자 정의 레포지토리
  - spring data jpa repository는 인터페이스만 정의하고 구현체는 스프링이 자동 생성
  - spring data jpa가 제공하는 인터페이스를 직접 구현하면 구현해야하는 기능이 너무 많음
  - 다양한 이유로 인터페이스의 메서드를 직접 구현하고 싶다면?
  	- JPA 직접 사용(entitymanger)
  	- spring jdbc template 사용
  	- mybatis 사용
  	- 데이터베이스 커넥션 직접 사용 등등

  *사용자 정의 구현 클래스*
  - 규칙 : repository interface 이름 + ‘impl’
  - spring data jpa 가 인식해서 스프링 빈으로 등록

  > 실무에서 복잡한 쿼리를 사용해야할 때 사용하면 됨 (QueryDSL, JDBC Template 등 …)

  > 항상 사용자 정의 리포지토리가 필요한 것은 아님.
  > 그냥 임의의 레포지토리로 만들어도 된다.
  > 커맨드와 쿼리를 분리하고, 복잡한 화면과 핵심 비지니스 로직을 분리하는 것이 중요하다.

  ## Auditing
  - 엔티티를 생성, 변경할 때 변경한 사람과 시간을 추적하고 싶으면?
  	- 등록일
  	- 수정일
  	- 등록자
  	- 수정자

  1. 순수 JPA 사용할 경우
  	- @prePersist, @preUpdate 사용해서 baseentity 만들고 상속 받으면 됨.
  2. Spring Data JPA 사용할 경우
  	- @CreatedBy 사용하고 @MappedSuperclass, @EntitiyListeners(AuditingEntityListener.class)하고 사용.

  ## Controller Paging
  - 페이징과 정렬을 스프링 MVC에서 사용할 수 있음.
  - 파라미터로 Pageable을 받을 수 있음.
  - Pageable은 인터페이스, 실제로는 org.springframework.data.domain.PageRequest 객체 생성

  *요청 파라미터*
  예) /members?page=0&size=3&sort=id,desc&sort=username,desc

  *접두사*
  - 한페이지에 페이징 정보가 둘 이상이면 접두사로 구분
  - @Qualifier에 접두사명 추가 “{접두사명}_xxx”
  - 예제 : /members?member_page=0&order_page=1
  ```
  public String list (
  	@Qualifier("member") Pageable memberPageable,
  	@Qualifier("order") Pageable orderPageable, ...
  ```


  ## Spring Data JPA 구현체 분석
  - @Repository 적용 : JPA 예외를 스프링이 추상화한 예외로 변환
  - @Transactional 트랜잭션 적용
  	- JPA의 모든 변경은 트랜잭션안에서 동작
  	- 서비스 계층에서 트랜잭션을 시작하지않으면 레파지토리에서 시작
  	- 서비스 계층에서 시작하면 이어받아서 사용
  	- 그래서, spring data jpa 사용시 트랜잭션이 없어도 데이터 등록, 변경이 가능
  - @Transactional(readOnly = true)
  	- 트랜잭션이 끝날때 flush를 생략해서 약간의 성능 향상을 얻을 수 있음
  - *save() 메서드*
  	- 새로운 엔티티면 저장(persist)
  	- 새로운 엔티티가 아니면 병합(merge)
  > 데이터를 업데이트 할 때 merge를 사용하면 안됨.
  > merge는 영속상태 entity가 영속상태를 벗어났을때 다시 영속상태로 만들때 사용.

  ## 새로운 엔티티를 구별하는 방법
  - *save() 메서드*
  	- 새로운 엔티티면 저장(persist)
  	- 새로운 엔티티가 아니면 병합(merge)

  - 새로운 엔티티 판단 기본전략
  	- 식별자(ID, PK)가 객체일 때 null로 판단
  	- 식별자가 자바 기본 타입일 때 0으로 판단
  	- persistable 인터페이스를 구현해서 판단 로직 변경 가능
  > JPA 식별자 생성 전략이 @GenerateValue면 save() 호출 시점에 식별자 없으므로,
  > 새로운 엔티티로 인식해서 정상작동.
  > 그러나, 직접 할당할 경우 객체가 들어가기 때문에 save()를 호출 할때 merge()가 호출된다.
  > 그래서 persistable 인터페이스를 구현해서 새로운 엔티티인지 확인하는 isNew()를 구현하면된다.
  ```
  @Override
  public boolean isNew() {
      return createdDate == null;
  }
  ```
  를 사용해서 존재하는지 판단하도록 하면 됨.

  ## Specifications(명세)
  - 스프링 데이터 jpa는 jpa criteria를 활용해서 이 개념을 사용할 수 있도록 지원
  *술어로 이루어짐*
  - 참 또는 거짓으로 평가
  - and or 같은 연산자 조합해서 다양한 검색조건 쉽게 생성
  - specification을 구현하면 명세를 조립할 수 있음
  - *너무 복잡하고, 어려움*
  > 실무에서는 어려워서 사용안함.. QueryDSL을 대신 사용하면 됨.

  ## Query By Example
  - 도메인 객체 그대로 가지고 검색조건 만듬
  - 데이터 저장소를 RDB에서 NoSQL로 변경해도 코드 변경이 없게 추상화 되어있음.
  - 무시할 조건을 추가해 줘야함 (ExampleMatcher)
  - join 조건 있는 검색시에는 연관관계 맺어서 검색하면 됨.
  *단점*
  - innerJoin만 가능, outerJoin 불가
  - 중첩 제약조건 안됨
  - 매칭 조건이 매우 단순
  > 실무에서는 QueryDSL 사용…

  ## Projections
  - 엔티티 대신에 DTO를 편리하게 조회할 때 사용
  - 전체 엔티티가 아니라 만약 회원 이름만 딱 조회하고 싶다면?
  - interface 만들어서 반환하는 메소드 선언
  - dto class 로 가져올 수도 있음.
  - 프로젝션 대상이 root 엔티티면 유용
  - 프로젝션 대상이 root 엔티티를 넘어가면 JPQL SELECT 최적화가 안됨.
  > 복잡하면 QueryDSL 사용하는게 좋음…

  ## 네이티브 쿼리
  - 가급적 사용하지 않는게 좋음
  *Spring Data JPA 기반 네이티브 쿼리*
  - 페이징 지원
  - 반환타입
  	- Object[]
  	- Tuple
  	- DTO(스프링 데이터 인터페이스 Projections 지원)
  - 제약
  	- Sort 파라미터를 통한 정렬이 정상 동작하지 않을 수 있음
  	- JPQL처럼 애플리케이션 로딩 시점에 문법 확인 불가
  	- 동적 쿼리 불가
