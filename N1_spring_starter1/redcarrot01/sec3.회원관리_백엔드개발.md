- 비즈니스 요구사항 정리 
- 회원 도메인과 리포지토리 만들기 

- 회원 리포지토리 테스트 케이스 작성 
- 회원 서비스 개발 
- 회원 서비스 테스트 

# 1. 비즈니스 요구사항 정리

- 데이터 : 회원ID, 이름
- 기능 : 회원 등록, 조회
- DB : 아직 선정 X

![image](https://user-images.githubusercontent.com/38436013/126443873-fef9e7ae-f825-4c8f-bd05-426c7c73c20e.png)

- 컨트롤러 : 웹 MVC
- 서비스 : 비지니스 로직
- 리포지토리 : DB접근, 도메인 객체를 DB에 저장하고 관리
- 도메인 : 비지니스 도메인 객체 예) 회원, 주문 , 쿠폰 등 DB에 저장하고 관리됨

![image](https://user-images.githubusercontent.com/38436013/126443900-bd83b7af-2e3b-45f2-9ce9-19a90977b951.png)

- 아직 DB 선정 못해서, 인터페이스로 구현 클래스를 변경할 수 있도록 설계
- DB는 고민 중
- 초기 개발 단계에서는 구현체로 가벼운 메모리 기반의 데이터 저장소 사용



# 2. 회원 도에인과 리포지토리 만들기

#### 회원 객체

~~~java
package hello.hellospring.domain;

public class Member {

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



#### 회원 리포지토리 클래스

~~~java
package hello.hellospring.repository;


import hello.hellospring.domain.Member;
import java.util.List;
import java.util.Optional;

public interface MemberRepository {
    Member save(Member member); // 회원 저장
    Optional<Member> findById(Long id); // 저장소에서 id 찾음
    Optional<Member> findByName(String name);
    List<Member> findAll();

}

~~~



#### 회원 리포지토리 메모리 구현체

~~~~java
package hello.hellospring.repository;

import hello.hellospring.domain.Member;

import java.util.*;

/*
 동시성 문제가 고려되어 있지 않기 때문에, 실무에서는 ConcurrentHashMap, AtomicLong 사용 고려
*/

public class MemoryMemberRepository implements  MemberRepository{

    private static Map<Long, Member> store = new HashMap<>(); // 저장소를 HashMap으로 선언
    private static long sequence = 0L;

    @Override
    public Member save(Member member) {
        member.setId(++sequence); // id는 시스템에서 정해주는 것이므로 여기서 정함
        store.put(member.getId(), member); // 저장소에 id 저장
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        // null이 반환될 가능성 -> Optional 감싸서 반환해줌
        return Optional.ofNullable(store.get(id));
    }

    @Override
    public Optional<Member> findByName(String name) {
        // 람다 사용 , filer는 탐색
        // member.getName()이 파라미터로 넘어온 name과 같다면 하나라도 반환
        return store.values().stream()
                .filter(member -> member.getName().equals(name))
                .findAny();
    }

    @Override
    public List<Member> findAll() {
        return new ArrayList<>(store.values());
    }

    public void clearStore() {
        store.clear();
    }
    // 이 로직이 동작하는 것을 확인하기 위해 앞으로 테스트 케이스 작성해보기.
}

~~~~



# 3. 회원 레포지토리 테스트 케이스 작성

#### 회원 리포지토리 메모리 구현체 테스트

src/test/java 하위 폴더에 생성한다.

~~~java
package hello.hellospring.repository;

import hello.hellospring.domain.Member;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;

import java.util.List;

import static org.assertj.core.api.Assertions.*;

public class MemoryMemberRepositoryTest {
    MemoryMemberRepository repository = new MemoryMemberRepository();

    // 전체 테스트를 실행하면, 테스트는 순서를 보장하지 않기 때문에 오류 발생.
    // 각 테스트 메서드마다 실행 끝나면 메모리 clear 해주는 메서드를 불러오는 콜백함수
    @AfterEach
    public void afterEach() {
        repository.clearStore();
    }

    @Test
    public void save(){
        // 저장이 잘 되었는가?, 값을 리포지에 저장,
        // 레포에서 findById로 Id 값을 꺼내 result에 저장
        // result랑 member랑 비교
        Member member = new Member();
        member.setName("spring");
        repository.save(member);
        Member result=repository.findById(member.getId()).get();
        //Assertions.assertEquals(member, result); // junit.jupiter.api.Assertions;
        assertThat(member).isEqualTo(result); // org.assertj.core.api.Assertions.*;
    }

    @Test
    public void findByName(){
        // 멤버 2명 저장
        Member member1 = new Member();
        member1.setName("Spring1");
        repository.save(member1);

        Member member2 = new Member();
        member2.setName("Spring2");
        repository.save(member2);

        Member result = repository.findByName("Spring1").get();
        //Member result = repository.findByName("Spring2").get(); // 오류

        assertThat(result).isEqualTo(member1);

    }
    @Test
    public void findAll(){
        Member member1 = new Member();
        member1.setName("spring1");
        repository.save(member1);

        Member member2 = new Member();
        member2.setName("spring2");
        repository.save(member2);

        List<Member> result = repository.findAll();

        assertThat(result.size()).isEqualTo(2);
    }

}
~~~



# 4. 회원 서비스 개발

#### 회원 서비스 MemberService

~~~java
package hello.hellospring.service;

import hello.hellospring.domain.Member;
import hello.hellospring.repository.MemberRepository;
import hello.hellospring.repository.MemoryMemberRepository;

import java.util.List;
import java.util.Optional;

public class MemberService {

    private final MemberRepository memberRepository = new MemoryMemberRepository();

    // 회원가입
    public Long join(Member member){
        // 중복 회원 검증하고, 통과하면 저장
        validateDuplicateMember(member);
        memberRepository.save(member);
        return member.getId();
    }

    private void validateDuplicateMember(Member member) {
        memberRepository.findByName(member.getName())
                .ifPresent(m -> {
                    throw new IllegalStateException("이미 존재하는 회원입니다.");
                });
    }

    // 회원 조회
    public List<Member> findMembers(){
        return memberRepository.findAll();
    }

    public Optional<Member> findOne(Long memberId){
        return memberRepository.findById(memberId);
    }
}

~~~

# 5. 회원 서비스 테스트

#### 기존에는 회원 서비스가 메모리 회원 리포지토리를 직접 생성하게 했다.

~~~java
public class MemberService {
 private final MemberRepository memberRepository = 
 new MemoryMemberRepository();
}
~~~

#### *회원 리포지토리의 코드가 회원 서비스 코드를 DI 가능하게 변경한다.

~~~java
public class MemberService {
 private final MemberRepository memberRepository;
 public MemberService(MemberRepository memberRepository) {
 this.memberRepository = memberRepository;
 }
 ...
}
~~~

#### 회원 서비스 테스트 

~~~java
package hello.hellospring.service;

import hello.hellospring.domain.Member;

import hello.hellospring.repository.MemoryMemberRepository;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.junit.jupiter.api.Assertions.fail;


class MemberServiceTest {

    MemberService  memberService ;
    MemoryMemberRepository memberRepository;

    @BeforeEach
    public void beforeEach() {
        memberRepository = new MemoryMemberRepository();
        memberService = new MemberService(memberRepository);
    }

    @AfterEach
    public void afterEach() {
        memberRepository.clearStore();
    }

    @Test
    void join() {
        // 레포지에 저장한 member가 맞니? 단순 등록 확인
        // given
        Member member = new Member();
        member.setName("Spring");

        // when
        Long saveId = memberService.join(member);

        // then
        Member findMember = memberService.findOne(saveId).get();
        assertThat(member.getName()).isEqualTo(findMember.getName());
    }

    @Test
    public void 중복_회원_예외() throws Exception{
        // 단순 등록 말고, 예외 처리 매우 중요
        // given
        Member member1 = new Member();
        member1.setName("spring");

        Member member2 = new Member();
        member2.setName("spring");

        // when
        memberService.join(member1);
        // 람다 이용해서 에러 처리 구문
        IllegalStateException e = assertThrows(IllegalStateException.class,
                () -> memberService.join(member2));//예외가 발생해야 한다.
        assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");

        // then
        // 단순 에러 처리
//       try {
//            memberService.join(member2);
//            fail();
//        }catch(IllegalStateException e){
//            assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다");
//        }
    }

//......
}
~~~

@BeforeEach : 각 테스트 실행 전에 호출된다, 테스트가 서로 영향 없도록 항상 새로운 객체 생성하고 의존관계도 새로 맺어준다.