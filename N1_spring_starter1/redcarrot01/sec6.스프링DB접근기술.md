### 목표

- H2 DB 설치
- 순수 JDBC
- 스프링 통합 테스트
- 스프링 JdbcTemplate
- JPA
- 스프링 데이터 JPA

# 1. H2 DB 실행

[윈도우]

- 실행 : H2\bin >` h2.bat`
- 데이터베이스 파일 생성 방법
  - `jdbc:h2:~/test` (최초 한번)
  - 본인계정 >` ~/test.mv.db `파일 생성 확인
  - 이후부터는` jdbc:h2:tcp://localhost/~/test `이렇게 접속

# 2. 순수 JDBC

JDBC는 예전에 사용하던 방식으로, Jdbc의 구현 흐름 정도만 이해하자.

#### build.gradle 파일에 jdbc, h2 데이터베이스 관련 라이브러리 추가

~~~
implementation 'org.springframework.boot:spring-boot-starter-jdbc'
runtimeOnly 'com.h2database:h2'
~~~



#### 스프링 부트 데이터베이스 연결 설정 추가

~~~
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.driver-class-name=org.h2.Driver
~~~



#### Jdbc 회원 리포지토리

MemberRepository가 회원 저장을 하지만, 디비랑 연동하기 위해 JdbcMemberRepository가 필요하다.

~~~java
package hello.hellospring.repository;
import hello.hellospring.domain.Member;
import org.springframework.jdbc.datasource.DataSourceUtils;
import javax.sql.DataSource;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;
public class JdbcMemberRepository implements MemberRepository {
    private final DataSource dataSource;
    public JdbcMemberRepository(DataSource dataSource) {
        this.dataSource = dataSource;
    }
    @Override
    public Member save(Member member) {
        String sql = "insert into member(name) values(?)";
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null; try {
            conn = getConnection();
            pstmt = conn.prepareStatement(sql,
                    Statement.RETURN_GENERATED_KEYS);
            pstmt.setString(1, member.getName());
            pstmt.executeUpdate();
            rs = pstmt.getGeneratedKeys();
            if (rs.next()) {
                member.setId(rs.getLong(1));
            } else {
                throw new SQLException("id 조회 실패");
            }
            return member;
        } catch (Exception e) {
            throw new IllegalStateException(e);
        } finally {
            close(conn, pstmt, rs);
        }
    }
    @Override
    public Optional<Member> findById(Long id) {
        String sql = "select * from member where id = ?";
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        try {
            conn = getConnection();
            pstmt = conn.prepareStatement(sql);
            pstmt.setLong(1, id);
            rs = pstmt.executeQuery(); if(rs.next()) {
                Member member = new Member();
                member.setId(rs.getLong("id"));
                member.setName(rs.getString("name"));
                return Optional.of(member);
            } else {
                return Optional.empty();
            }
        } catch (Exception e) {
            throw new IllegalStateException(e);
        } finally {
            close(conn, pstmt, rs);
        }
    }
    @Override
    public List<Member> findAll() {
        String sql = "select * from member";
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        try {
            conn = getConnection();
            pstmt = conn.prepareStatement(sql);
            rs = pstmt.executeQuery();
            List<Member> members = new ArrayList<>();
            while(rs.next()) {
                Member member = new Member();
                member.setId(rs.getLong("id"));
                member.setName(rs.getString("name"));
                members.add(member);
            } return members;
        } catch (Exception e) {
            throw new IllegalStateException(e);
        } finally {
            close(conn, pstmt, rs);
        }
    }
    @Override
    public Optional<Member> findByName(String name) {
        String sql = "select * from member where name = ?";
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        try {
            conn = getConnection();
            pstmt = conn.prepareStatement(sql);
            pstmt.setString(1, name);
            rs = pstmt.executeQuery();
            if(rs.next()) {
                Member member = new Member();
                member.setId(rs.getLong("id"));
                member.setName(rs.getString("name"));
                return Optional.of(member);
            }
            return Optional.empty();
        } catch (Exception e) {
            throw new IllegalStateException(e);
        } finally {
            close(conn, pstmt, rs);
        }
    }
    private Connection getConnection() { return DataSourceUtils.getConnection(dataSource);
    }
    private void close(Connection conn, PreparedStatement pstmt, ResultSet rs)
    {
        try {
            if (rs != null) {
                rs.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        try {
            if (pstmt != null) {
                pstmt.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        try {
            if (conn != null) {
                close(conn);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    private void close(Connection conn) throws SQLException {
        DataSourceUtils.releaseConnection(conn, dataSource);
    }
}
~~~

> DataSource는 데이터베이스 커넥션을 획득할 때 사용하는 객체다. 스프링부트는 데이터베이스 커넥션 정보를 바탕으로 DataSource를 생성하고 스프링 빈으로 만들어둔다. 그래서 DI를 받을 수 있다.



#### 스프링 설정 변경

~~~java
@Configuration
public class SpringConfig {

    private DataSource dataSource;

    @Autowired
    public SpringConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
//        return new MemoryMemberRepository();
        return new JdbcMemberRepository(dataSource);
    }
}
~~~

> 스프링부트가 resources/application.properties에 있는 datasource를 빈으로 만들고, @Autowired를 통해서 주입을 해주는 것이다.
>
>  MemoryMemberRepository() -> JdbcMemberRepository(dataSource) 로 바꾼 것 외에 다른 코드는 건드리지 않았다. **객체지향은 기존 코드를 변경하지 않고 구현체(인터페이스) 변경이 가능하다.** -> 객체 지향의 다형성, 큰 장점!!

![image](https://user-images.githubusercontent.com/38436013/126604334-36946d43-c2be-440e-85d3-cb596ce38c81.png)

> 개방 - 폐쇄 원칙 (OCP, open-closed principle)
>
> - 확장(기능 추가)에는 열려 있고, 수정 , 변경에는 닫혀 있다.
> - 다형성을 잘 활용하면, 기능 변경해도 애플리케이션 코드를 변경하지 않는다.
>
> 스프링의 DI(Dependencies Injection) 을 사용하면 기존 코드 변경X, 설정만으로 구현 클래스 변경 가능하다.



# 3. 스프링 통합 테스트

**이전에는 순수 자바 코드로 테스팅했으나, db 연결되어 있는 부분을 테스트 하기 위해 스프링 통합 테스트를 지금부터 진행한다.**

#### 회원 서비스 스프링 통합 테스트

~~~JAVA
package hello.hellospring.service;
import hello.hellospring.domain.Member;
import hello.hellospring.repository.MemberRepository;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.transaction.annotation.Transactional;
import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;
@SpringBootTest
@Transactional
class MemberServiceIntegrationTest {
    @Autowired MemberService memberService;
    @Autowired MemberRepository memberRepository;
    @Test
    public void 회원가입() throws Exception {
        //Given
        Member member = new Member();
        member.setName("hello");
        //When
        Long saveId = memberService.join(member); //Then
        Member findMember = memberRepository.findById(saveId).get();
        assertEquals(member.getName(), findMember.getName());
    }
    @Test
    public void 중복_회원_예외() throws Exception {
        //Given
        Member member1 = new Member();
        member1.setName("spring");
        Member member2 = new Member();
        member2.setName("spring");
        //When
        memberService.join(member1);
        IllegalStateException e = assertThrows(IllegalStateException.class,
                () -> memberService.join(member2));//예외가 발생해야 한다.
        assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");
    }
}
~~~

> TEST는 필드 기반의 어노테이션 사용도 나쁘지 않다.
>
> 테스트는 반복할 수 있어야 한다. 이전 테스트 결과(메모리)가 다음 테스트에 영향을 주면 안된다. DB의 트랜잭션은 커밋을 해줘야 반영이 된다. 커밋 전에 롤백이 일어나는 경우, DB의 데이터는 유실된다.

>@SpringBootTest : 스프링 컨테이너와 테스트를 함께 실행한다. 
>
>@Transactional : 테스트 케이스에 이 애노테이션이 있으면, 테스트 시작 전에 트랜잭션을 시작하고,  테스트 완료 후에 항상 롤백한다. 이렇게 하면 DB에 데이터가 남지 않으므로 다음 테스트에 영향을 주지 않는다.



# 4. 스프링 JdbcTemplate

- 순수 Jdbc와 동일한 환경설정을 하면 된다.  
- 스프링 JdbcTemplate과 MyBatis 같은 라이브러리는 JDBC API에서 본 반복 코드를 대부분 제거해준다. 
- 하지만 SQL은 직접 작성해야 한다.

#### 스프링 JdbcTemplate 회원 리포지토리

~~~java
package hello.hellospring.repository;
import hello.hellospring.domain.Member;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
import org.springframework.jdbc.core.simple.SimpleJdbcInsert;
import javax.sql.DataSource;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Optional;
public class JdbcTemplateMemberRepository implements MemberRepository {
    private final JdbcTemplate jdbcTemplate;
    public JdbcTemplateMemberRepository(DataSource dataSource) {
        jdbcTemplate = new JdbcTemplate(dataSource);
    }
    @Override
    public Member save(Member member) {
        SimpleJdbcInsert jdbcInsert = new SimpleJdbcInsert(jdbcTemplate);
        jdbcInsert.withTableName("member").usingGeneratedKeyColumns("id");
        Map<String, Object> parameters = new HashMap<>();
        parameters.put("name", member.getName());
        Number key = jdbcInsert.executeAndReturnKey(new
                MapSqlParameterSource(parameters));
        member.setId(key.longValue());
        return member; }
    @Override
    public Optional<Member> findById(Long id) {
        List<Member> result = jdbcTemplate.query("select * from member where id = ?", memberRowMapper(), id);
        return result.stream().findAny();
    }
    @Override
    public List<Member> findAll() {
        return jdbcTemplate.query("select * from member", memberRowMapper());
    }
    @Override
    public Optional<Member> findByName(String name) {
        List<Member> result = jdbcTemplate.query("select * from member where name = ?", memberRowMapper(), name);
        return result.stream().findAny();
    }
    private RowMapper<Member> memberRowMapper() {
        return (rs, rowNum) -> {
            Member member = new Member();
            member.setId(rs.getLong("id"));
            member.setName(rs.getString("name"));
            return member;
        };
    }
}

~~~

> 생성자가 하나면 스프링 빈에 등록되므로 @Autowired는 생략 가능하다.
>
> RowMapper 부분에서 객체를 만들어서 return 해주는 것이다.

#### JdbcTemplate을 사용하도록 스프링 설정 변경

~~~java
@Configuration
public class SpringConfig {
 private final DataSource dataSource;
 public SpringConfig(DataSource dataSource) {
 this.dataSource = dataSource;
 }
 @Bean
 public MemberService memberService() {
 return new MemberService(memberRepository());
 }
 @Bean
 public MemberRepository memberRepository() {
// return new MemoryMemberRepository();
// return new JdbcMemberRepository(dataSource);
 return new JdbcTemplateMemberRepository(dataSource);
 }
}
~~~



# 5. JPA

- orm기술로, 객체와 관계형db를 매핑해준다.

- JPA는 기존의 반복 코드는 물론이고, 기본적인 SQL도 JPA가 직접 만들어서 실행해준다.

- JPA를 사용하면, SQL과 데이터 중심의 설계에서 객체 중심의 설계로 패러다임을 전환을 할 수 있다. 

- JPA를 사용하면 개발 생산성을 크게 높일 수 있다

#### build.gradle 파일에 JPA, h2 데이터베이스 관련 라이브러리 추가

~~~java
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
~~~

#### 스프링 부트에 JPA 설정 추가 : resources/application.properties

~~~
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=none
~~~

>show-sql : JPA가 생성하는 SQL을 출력한다. 
>
>ddl-auto : JPA는 테이블을 자동으로 생성하는 기능을 제공하는데 none 를 사용하면 해당 기능을 끈다. create 를 사용하면 엔티티 정보를 바탕으로 테이블도 직접 생성해준다. 

#### JPA 엔티티 매핑

~~~java
@Entity
public class Member {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id; // 로그 id, 진짜 id 아님
    private String name;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
~~~

#### JPA 회원 리포지토리

~~~java
package hello.hellospring.repository;

import hello.hellospring.domain.Member;
import javax.persistence.EntityManager;
import java.util.List;
import java.util.Optional;
public class JpaMemberRepository implements MemberRepository {
    private final EntityManager em; // gradle에 jpa 라이브러리 등록하면 자동 생성됨
    public JpaMemberRepository(EntityManager em) {
        this.em = em;
    }
    public Member save(Member member) {
        em.persist(member); // 영구저장하다 persist
        return member;
    }
    public Optional<Member> findById(Long id) {
        Member member = em.find(Member.class, id); // id 조회
        return Optional.ofNullable(member);
    }
    public List<Member> findAll() {
        // Member 엔티티를 조회, Member as m
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }
    public Optional<Member> findByName(String name) {
        List<Member> result = em.createQuery("select m from Member m where m.name = :name", Member.class)
                .setParameter("name", name)
                .getResultList();
        return result.stream().findAny();
    }
}
~~~

#### 서비스 계층에 트랜잭션 추가

~~~java
import org.springframework.transaction.annotation.Transactional
    
@Transactional
public class MemberService {}
~~~

> org.springframework.transaction.annotation.Transactional 를 사용하자
>
> 스프링은 해당 클래스의 `메서드를 실행할 때 트랜잭션을 시작하고, 메서드가 정상 종료되면 트랜잭션을 커밋한다.` 
>
> 만약 런타임 예외가 발생하면 롤백한다. 
>
> **JPA를 통한 모든 데이터 변경은 트랜잭션 안에서 실행**해야 한다

#### JPA를 사용하도록 스프링 설정 변경

~~~java
@Configuration
public class SpringConfig {

    private final DataSource dataSource;
    private final EntityManager em;

    public SpringConfig(DataSource dataSource, EntityManager em) {
        this.dataSource = dataSource;
        this.em = em;
    }
~~~



# 6. 스프링 데이터 JPA

>  **스프링 부트와 JPA**만 사용해도 개발 생산성이 정말 많이 증가하고, 개발해야할 코드도 확연히 줄어든다.  여기에 **스프링 데이터 JPA**를 사용하면, 기존의 한계를 넘어 마치 마법처럼, `리포지토리에 구현 클래스 없이 인터페이스 만으로 개발`을 완료할 수 있다. 그리고 `반복 개발해온 기본 CRUD 기능도 스프링 데이터 JPA가 모두 제공`한다. 
>
> 스프링 부트와 JPA라는 기반 위에, 스프링 데이터 JPA라는 환상적인 프레임워크를 더하면 개발이 정말 쉬워진다. 지금까지 조금이라도 단순하고 반복이라 생각했던 개발 코드들이 확연하게 줄어든다.  따라서 개발자는 핵심 비즈니스 로직을 개발하는데, 집중할 수 있다. 실무에서 관계형 데이터베이스를 사용한다면 **스프링 데이터 JPA는 이제 선택이 아니라 필수다.**

#### 스프링 데이터 JPA 회원 리포지토리

~~~java
package hello.hellospring.repository;

import hello.hellospring.domain.Member;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface SpringDataJpaMemberRepository extends JpaRepository<Member, Long>,
        MemberRepository{ // JpaRepository<Member, Long> -> id
    // 상속받고 있으며, 자동으로 구현체를 생성, 스프링data jpa가 이것을 보고 빈에 자동 등록함
    Optional<Member> findByName(String name);
}
~~~

#### 스프링 데이터 JPA 회원 리포지토리를 사용하도록 스프링 설정 변경

~~~java
@Configuration
public class SpringConfig {

    private final MemberRepository memberRepository;

    @Autowired
    public SpringConfig(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository);
    }
}
~~~

#### 스프링 데이터 JPA 제공 클래스

<img src="https://user-images.githubusercontent.com/38436013/126629333-e2585a04-509e-4d30-b972-527a83ceb5e6.png" alt="image" style="zoom:80%;" />

#### 스프링 데이터 JPA 제공 기능

- 인터페이스를 통한 기본적인 CRUD 
- findByName() , findByEmail() 처럼 메서드 이름 만으로 조회 기능 제공 
- 페이징 기능 자동 제공

> `실무에서는 JPA와 스프링 데이터 JPA를 기본으로 사용하고, `  `복잡한 동적 쿼리는 Querydsl이라는 라이브러리` 를 사용하면 된다. Querydsl을 사용하면 쿼리도 자바 코드로 안전하게 작성할 수 있고, 동적 쿼리도 편리하게 작성할 수 있다. 이 조합으로 해결하기 `어려운 쿼리는 JPA가 제공하는 네이티브 쿼리`를 사용하거나, `앞서 학습한 스프링 JdbcTemplate를 사용`하면 된다.