## 다운로드 및 실행 
- https://h2database.com/html/main.html
- Default 경로 C:\Program Files (x86)\H2\bin 에서 h2.bat 실행 (Window 기준)
![sec6캡쳐2](https://user-images.githubusercontent.com/81945553/126860792-8e31fe94-4cf9-4f16-9462-d62b651f7cb2.png)

- ※  프로세스 종료 : netstat-ano (프로세스 pid 확인) , taskkill /pid 12324 /f (12324 프로세스 강제종료) 

## H2 파일 생성 및 접속 방법 
- jdbc:h2:~/test (최초 한번), dir 명령어를 통해 test.mv.db 파일 생성 확인  
![sec6 캡쳐3](https://user-images.githubusercontent.com/81945553/126860819-455692ad-fc1f-4d32-8d77-e26a436cb7ee.png)  
![sec6캡처1](https://user-images.githubusercontent.com/81945553/126860821-afe13574-2bdd-4ad0-b25b-1d27abc02a01.png)  

- 이 후 Generic H2 (Server)로 변경 후 jdbc:h2:tcp://localhost//test로 접속 

## 접속 화면 

![sec6 캡쳐4](https://user-images.githubusercontent.com/81945553/126860917-c7698ee3-ebc0-4eea-b90b-529acae88e45.png)
- `Shift` + `Enter` : 쿼리문 실행 단축키 
- 여러 개의 쿼리문 실행 가능 

## 스프링 부트에 연결 프로퍼티 설정 
```Java
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
```
※ 스프링부트 2.4부터는 spring.datasource.username=sa 를 꼭 추가해주어야 한다

## <ORM 기술 과정> Jdbc -> JdbcTemplate -> Jpa -> Spring Jpa

- Jdbc : Datasource (import javax.sql.DataSource) 와 Connection 등을 사용하여 연결 , 예외 처리문이 많고 코드가 복잡하다.  
 쿼리 ex) String sql = "select * from member where name = ?";
- JdbcTemplate : JdbcTemplate (import org.springframework.jdbc.core.JdbcTemplate) 를 사용하여 기존 예외처리가 없어지고 간결해진다. 쿼리 동일  
- Jpa : EntityManager (import javax.persistence.EntityManager) 와 생성자 DI를 통해 사용하며 개발자가 생성한 Repository를 implements 하여 개발   
-  쿼리 ex) List result = em.createQuery("select m from Member m where m.name = :name", Member.class)  
- Spring Data Jpa : 인터페이스로 생성 후에 JpaRepository를 추가로 JpaRepository를 extends 한다. <객체클래스, 키 값> ,   
Repository -> JpaRepository에서 제공하는 CRUD를 사용하여 구현 가능하기 때문에 로직이 간단하다.   
-> 실무에서는 복잡한 쿼리는 Jpa에서 제공하는 네이티브 쿼리를 사용하거나 JdbcTemplate을 혼합하여 사용한다.

## JPA 
- SQL을 JPA가 직접 만들어서 실행시켜준다. 
- SQL과 데이터 중심의 설계에서 객체 중심의 설계로 패러다임 전환 가능
- 개발 생산성을 크게 증가시킬 수 있다. 


```Java
public interface SpringDataJpaMemberRepository extends JpaRepository<Member, Long>, MemberRepository {

    //JPQL select m from Member m where m.name = ?
    @Override
    Optional<Member> findByName(String name);
}
```
- 인터페이스는 다중상속 가능, 인터페이스는 extends로 상속만 가능
- JpaRepository를 통해 자동 빈 등록 및 CRUD 기능 제공
- 실무에서는 해결하기 어려운 쿼리는 JPA에서 제공하는 네이티브 쿼리를 사용하거나 JdbcTemplate을 사용하면 된다.

## Spring Data JPA 제공 기능
![sec6캡쳐5](https://user-images.githubusercontent.com/81945553/126861356-3376d6c6-421b-4536-940c-81b126f896cc.png)

-  인터페이스를 통한 CRUD 기능 
-  메서드 이름만으로 조회 기능 제공 (findByName() 등)
-  페이징 기능 제공
