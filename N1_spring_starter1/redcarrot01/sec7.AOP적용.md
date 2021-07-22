# 1. AOP가 필요한 상황

- 모든 메소드의 호출 시간을 측정하고 싶다면? 
- 공통 관심 사항(cross-cutting concern) vs 핵심 관심 사항(core concern) 
- 회원 가입 시간, 회원 조회 시간을 측정하고 싶다면

<img src="https://user-images.githubusercontent.com/38436013/126636918-04951095-58f1-4eaa-8b79-fdd3e06d7fbb.png" alt="image" style="zoom: 67%;" />

#### MemberService 회원 조회 시간 측정 추가

~~~java
@Transactional //  jpa를 사용하려면 트랜잭션 어노테이션 설정, 트랜잭션 안에서 데이터 저장변경
public class MemberService {


    private final MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }


    // 회원가입
    public Long join(Member member){

        long start = System.currentTimeMillis();
        try{
            // 중복 회원 검증하고, 통과하면 저장
            validateDuplicateMember(member);
            memberRepository.save(member);
            return member.getId();

        }
        finally{
            long finish = System.currentTimeMillis();
            long timeMs = finish - start;
            System.out.println("join " + timeMs + "ms");
        }

    }

    private void validateDuplicateMember(Member member) {
        memberRepository.findByName(member.getName())
                .ifPresent(m -> {
                    throw new IllegalStateException("이미 존재하는 회원입니다.");
                });
    }

    // 회원 조회
    public List<Member> findMembers(){
        long start = System.currentTimeMillis();
        try {
            return memberRepository.findAll();
        } finally {
            long finish = System.currentTimeMillis();
            long timeMs = finish - start;
            System.out.println("findMembers " + timeMs + "ms");
        }
    }

    public Optional<Member> findOne(Long memberId){
        return memberRepository.findById(memberId);
    }
}
~~~

문제 

- 회원가입, 회원 조회에 시간을 측정하는 기능은 핵심 관심 사항이 아니다. 

- 시간을 측정하는 로직은 공통 관심 사항이다. 

- 시간을 측정하는 로직과 핵심 비즈니스의 로직이 섞여서 유지보수가 어렵다. 

- 시간을 측정하는 로직을 별도의 공통 로직으로 만들기 매우 어렵다. 

- 시간을 측정하는 로직을 변경할 때 모든 로직을 찾아가면서 변경해야 한다

# 2. AOP 적용

AOP: Aspect Oriented Programming 

공통 관심 사항(cross-cutting concern) vs 핵심 관심 사항(core concern) 분리

시간 측정 로직을 한 곳에 모아두고, 원하는 곳 지정함 - AOP 적용

<img src="https://user-images.githubusercontent.com/38436013/126637189-9be9abc0-3042-4881-b5e6-b99235fcef3d.png" alt="image" style="zoom:67%;" />

#### 시간 측정 AOP 등록

~~~java
package hello.hellospring.aop;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Component
@Aspect
public class TimeTraceAop {

    @Around("execution(* hello.hellospring..*(..))")  // 원하는 조건 : 패키지 내에 적용
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable{

        long start = System.currentTimeMillis();

        System.out.println("START: " + joinPoint.toString());

        try {
            return joinPoint.proceed();
        } finally {
            long finish = System.currentTimeMillis();
            long timeMs = finish - start;
            System.out.println("END: " + joinPoint.toString()+ " " + timeMs +
                    "ms");
        }
    }
}

~~~

- 회원가입, 회원 조회등 핵심 관심사항과 시간을 측정하는 공통 관심 사항을 분리한다.

- 시간을 측정하는 로직을 별도의 공통 로직으로 만들었다. 
- 핵심 관심 사항을 깔끔하게 유지할 수 있다. 
- 변경이 필요하면 이 로직만 변경하면 된다. 
- 원하는 적용 대상을 선택할 수 있다.



# 3. 스프링의 AOP 동작 방식 설명

![image](https://user-images.githubusercontent.com/38436013/126639478-c29a8cd0-536d-469f-9c0a-8b9dccc5bd27.png)

> AOP를 적용하면 가짜 서비스(프록시)를 생성한다. 스프링 컨테이너는 빈 등록할 때, 가짜 스프링 빈을 먼저 세우고, 가짜 스프링 빈 끝나면 내부적으로 실제 빈을 호출
>
> **결론은 컨트롤러는 가짜 서비스(프록시)를 호출하는 것이다.**

#### AOP 적용 전체 그림

![image](https://user-images.githubusercontent.com/38436013/126639577-6e0607f5-0ae6-48de-9cb6-f69cb8347176.png)