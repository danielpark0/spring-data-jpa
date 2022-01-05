# 실전 Spring Data JPA
## 예제 도메인 모델
*엔티티 클래스*
![](/img/entityClass.png)

 *Spring Data JPA가 구현 클래스 대신 생성*
![](/img/jpaRepoInterface.png)
- @Repository annotation은 생략가능
	- 컴포넌트 스캔을 Spring Data JPA가 자동으로 처리
	- JPA 예외를 스프링 예외로 변환하는 과정도 자동을 처리
