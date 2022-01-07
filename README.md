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
