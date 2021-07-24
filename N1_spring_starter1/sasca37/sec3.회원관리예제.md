# 회원 관리 예제 - 백엔드
- 비즈니스 요구사항 정리
- 회원 도메인과 리포지토리 만들기
- 회원 리포지토리 테스트 케이스 작성
- 회원 서비스 개발
- 회원 서비스 테스트 

## 1. 비즈니스 요구사항 정리
- 데이터: 회원ID, 이름 : Member 
- 기능: 회원 등록, 조회
- 아직 데이터 저장소가 선정되지 않음(가상의 시나리오)

![sec3캡쳐1](https://user-images.githubusercontent.com/81945553/126855740-8bdc47e5-7ae8-46dc-9550-0ff8e5125788.png)
- 컨트롤러: 웹 MVC의 컨트롤러 역할
- 서비스: 핵심 비즈니스 로직 구현
- 리포지토리: 데이터베이스에 접근, 도메인 객체를 DB에 저장하고 관리
- 도메인: 비즈니스 도메인 객체, 예) 회원, 주문, 쿠폰 등등 주로 데이터베이스에 저장하고 관리됨

아직 데이터가 선정되지 않아서 인터페이스로 구현 클래스를 변경할 수 있도록 설계 MemberService -> MemberRepository(인터페이스) <- MemoryMemberRepository
Optional : 값이 널이여도 감쌀수 있도록 해줌

## 2. 회원 도메인과 리포지토리 만들기 

- 회원 도메인
```Java
@Entity
public class Member {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id; //시스템이 정해주는 id

    //@Column(name = "username")
    private String name; //사용자가 적는 이름

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
```
- 회원 리포지토리 메모리 구현체 

``` Java
public class MemoryMemberRepository implements MemberRepository{

    private static Map<Long, Member> store = new HashMap<>();
    private static long sequence = 0L; //키값을 생성해주는 시퀸스

    @Override
    public Member save(Member member) {
        member.setId(++sequence);
        store.put(member.getId(), member);
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        //해당 값이 널이여도 반환할 수 있도록 Nullable에 값을 감싼다.
        return Optional.ofNullable(store.get(id));
    }

    @Override
    public Optional<Member> findByName(String name) {
        //값을 찾는 순간 그값을 반환
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
}
```

## 3. 회원 리포지토리 테스트 케이스 작성 
- 테스트 3단계 given : 뭔가 주어졌을 때 when : 이걸 실행했을 때 then : 결과가 이게 나와야 돼
- @AfterEach : 각 테스트가 종료 될때마다 실행 (테스트 간 의존관계를 없애기위한 clear 작업 처리 ),테스트는 순서와 상관없이 독립적으로 실행되어야한다. 
(한번에 여러테스트 실행시 메모리 DB에 직전 테스트 결과가 남아있을 수 있다)
- @BeforeEach : 테스트 실행전에 설정
- Assertions 사용으로 junit 테스트 import static org.assertj.core.api.Assertions.*; assertThat(member).isEqualTo(result);
서비스 입장에서는 외부에서 memberRepository를 넣어줌 -> 이것을 DI 라고함 memberService = new MemberService(memberRepository); 
DI (필드, 세터, 생성자 3가지가 있지만 동적으로 변하는 경우가 거의 없기 때문에 생성자 주입을 권장)

```Java
class MemoryMemberRepositoryTest {

    MemoryMemberRepository repository = new MemoryMemberRepository();

    // 테스트 하나 수행할때마다 레포지토리를 수정해줌 **중요 (테스트간 의존관계 없도록)
    @AfterEach
    public void afterEach(){
        repository.clearStore();
    }

    @Test
    public void save(){
        Member member = new Member();
        member.setName("spring");

        repository.save(member);
        // optional 값을 get 사용해서 꺼내옴
        Member result = repository.findById(member.getId()).get();
        //System.out.println("result = "+ (result == member));
        //테스트 실행시 일치하지 않으면 빨간불들어옴
        //Assertions.assertEquals(member, result);
        assertThat(member).isEqualTo(result);
    }

    @Test
    public void findByName(){
        Member member1 = new Member();
        member1.setName("spring1");
        repository.save(member1);

        // 쉬프트 f6 누르면 한꺼번에 같은 내용 변경 가능
        Member member2 = new Member();
        member2.setName("spring2");
        repository.save(member2);

        Member result = repository.findByName("spring1").get();

        assertThat(result).isEqualTo(member1);
    }

    @Test
    public void findAll() {
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
```
## 4. 회원 서비스 개발 
``` Java
package hello.hellospring.service;

import hello.hellospring.domain.Member;
import hello.hellospring.repository.MemberRepository;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.Optional;

// 스프링이 올라올 때 얘를 서비스라고 인식하고 등록하고 생성자를 호출
@Transactional
public class MemberService {

    private final MemberRepository memberRepository;

    // MemberRepository도 Autowired 때문에 주입됨, 의존관계설정해줌
    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    /*
          회원 가입
    */
    public Long join(Member member){
        //같은 이름이 있는 중복 회원X
        // Optional 안에 감쌌기 때문에 중복일경우 밑에 메서드와 같이 해결이 가능하다.
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

    /*
        전체 회원 조회
     */
    public List<Member> findMember() {
        return memberRepository.findAll();
    }

    public Optional<Member> findOne(Long memberId) {
        return memberRepository.findById(memberId);
    }

}

```

## 5. 회원 서비스 테스트 
```Java
package hello.hellospring.service;

import hello.hellospring.domain.Member;
import hello.hellospring.repository.MemoryMemberRepository;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import static org.assertj.core.api.Assertions.*;
import static org.junit.jupiter.api.Assertions.*;
// 컨트롤 쉬프트 t : 자동 테스트 패키지, 클래스 , 메서드 생성
// 쉬프트 f10 : 이전에 실행 한걸 그대로 실행해줌
class MemberServiceTest {

    MemberService memberService;
    MemoryMemberRepository memberRepository;

    // 테스트 동작하기 전에
    @BeforeEach
    public void beforeEach() {
        memberRepository = new MemoryMemberRepository();
        // 서비스 입장에서는 외부에서 memberRepository를 넣어줌 -> 이것을 DI 라고함
        memberService = new MemberService(memberRepository);
    }

    @AfterEach
    public void afterEach() {
        memberRepository.clearStore();
    }
    // 리포지토리를 꺼내기위해서

    @Test
    void 회원가입() {
        //given 뭔가 주어졌을때
        Member member = new Member();
        member.setName("spring");

        //when 이걸 실행했을때
        Long saveId = memberService.join(member);

        //then 결과가 이게 나와야해
        Member findMember = memberService.findOne(saveId).get();
        assertThat(member.getName()).isEqualTo(findMember.getName());
    }

    @Test
    public void 중복_회원_예외() {
        //given
        Member  member1 = new Member();
        member1.setName("spring");

        Member  member2 = new Member();
        member2.setName("spring");
        //when
        memberService.join(member1);
        IllegalStateException e = assertThrows(IllegalStateException.class, () -> memberService.join(member2));
        assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");
       /* try {
            memberService.join(member2);
            fail();
        }
        catch (IllegalStateException e) {
            assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");
        }*/

        //then
    }

   @Test
    void findMember() {
    }

    @Test
    void findOne() {
    }
}
```
