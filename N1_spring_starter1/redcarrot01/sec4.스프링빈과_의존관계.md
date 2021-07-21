### 목표 : 스프링 빈을 등록하는 2가지 방법

- 컴포넌트 스캔과 자동 의존 관계 설정
- 자바 코드로 직접 스프링 빈 등록하기

# 1. 컴포넌트 스캔과 자동 의존관계 설정

회원 컨트롤러가 회원서비스와 회원 리포지토리를 사용할 수 있게 의존관계를 준비하자.

#### 회원 컨트롤러에 의존관계 추가 

~~~java
package hello.hellospring.controller;

import hello.hellospring.service.MemberService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;


/*
@Controller -> 컴포넌트를 사용하게 되면 스프링에서
스프링 컨테이너는 MemberController 객체를 생성해서 넣어둠 & 스프링이 맡아서 관리
=> 스프링 빈이 관리된다고 표현
* */
@Controller
public class MemberController {
    // new 객체를 생성하지 않고, 스프링 컨테이너에 등록하여 받아서 쓰는 형태로 구현함
    private final MemberService memberService;

    // @Autowired : 스프링이 스프링 컨테이너에 있는 MemberService를 연결시킴
    @Autowired
    public MemberController(MemberService memberService){
        this.memberService = memberService;
    }
}

~~~

- 생성자에 `@Autowired`가 있으면 스프링이 연관된 객체를 스프링 컨테이너에서 찾아서 넣어준다. 이렇게 객체 의존관계를 외부에서 넣어주는 것을 DI (Dependency Injection), 의존성 주입이라 한다.

- 이전 테스트에서는 개발자가 직접 주입했고, 여기서는  @Autowired에 의해 스프링이 주입해준다.

#### 오류 발생

~~~
Consider defining a bean of type 'hello.hellospring.service.MemberService' in your configuration.
~~~

#### memberService가 스프링 빈으로 등록되어 있지 않다.

<img src="https://user-images.githubusercontent.com/38436013/126494877-4cc12b96-59b5-4469-9f06-64e9f7d560e2.png" alt="image" style="zoom:50%;" />

>  해결 방법은 아래를 통해 알아보자!

#### 컴포넌트 스캔 원리

- `@Component` 애노테이션이 있으면 스프링 빈으로 자동 등록된다. 
- `@Controller` 컨트롤러가 스프링 빈으로 자동 등록된 이유도 컴포넌트 스캔 때문이다.

- `@Component` 를 포함하는 다음 애노테이션도 스프링 빈으로 자동 등록된다.
  - `@Controller,  @Service,  @Repository`

#### 회원 서비스 스프링 빈 등록

~~~java
// @Service로 이 클래스가 스프링 연관 객체임을 나타냄
@Service
public class MemberService {

    //private final MemberRepository memberRepository = new MemoryMemberRepository();
    // MemberService와 MemberServiceTest의 MemoryMemberRepository 객체는 서로 다르다.
    // 다른 것을 쓸 필요가 없으므로 수정 -> di
    private final MemberRepository memberRepository;
    @Autowired
    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
~~~

> 생성자에 @Autowired 를 사용하면 객체 생성 시점에 스프링 컨테이너에서 해당 스프링 빈을 찾아서 주입한다. 생성자가 1개만 있으면 @Autowired 는 생략할 수 있다.

#### 회원 리포지토리 스프링 빈 등록

~~~java
@Repository
public class MemoryMemberRepository implements MemberRepository {}
~~~

<img src="https://user-images.githubusercontent.com/38436013/126497100-10cb7114-2820-4a11-8d58-10a9a1a887f8.png" alt="image" style="zoom:67%;" />

> 스프링은 스프링 컨테이너에 스프링 빈을 등록할 때, **기본으로 싱글톤**으로 등록한다(유일하게 하나만 등록해서 공유한다) 따라서 같은 스프링 빈이면 모두 같은 인스턴스다. 

> **정리**
>
> 컨트롤러 통해서 외부 요청 받고, 서비스 통해 비지니스 로직 처리하고, 리포지토리 통해 데이터 저장하는 일련의 과정을 자세히 설명하면 아래와 같다.
>
> 스프링 뜰 때 컨트롤러, 서비스, 리포지토리를 가져온다. 컨트롤러와 서비스 연결을 위해 Autowired를 생성한다. Autowired를 생성자에 붙이면, 컨트롤러가 생성될 때, 스프링 빈에 등록된 서비스 객체를 가져와 스프링 컨테이너에 주입한다. 이것이 바로 의존성 주입이다. 
>
> 서비스와 리포지토리 연결을 위해 Autowired를 생성한다. 서비스 생성자에 Autowired를 붙이면, 스프링이 서비스를 생성할 때,  생성자를 호출하는데 이때, Autowired가 있는 리포지토리를 확인하여 스프링 컨테이너에 리포지토리를 주입한다.



# 2. 자바 코드로 직접 스프링 빈 등록하기

회원 서비스와 회원 리포지토리의 @Service, @Repository, @Autowired 애노테이션을 제거하고 진행한다.

~~~java
package hello.hellospring;

import hello.hellospring.repository.MemberRepository;
import hello.hellospring.repository.MemoryMemberRepository;
import hello.hellospring.service.MemberService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SpringConfig {
    
    @Bean
    public MemberService memberService(){
        return new MemberService(memberRepository());
    }
    
    @Bean
    public MemberRepository memberRepository(){
        return new MemoryMemberRepository();
    }
}
~~~

- @Bean를 통해 memberService, memberRepository를 스프링 빈에 올려준다.
- MemberService(memberRepository()) 는 Autowired처럼 스프링 빈에 등록된 리포지토리를 넣어준다.

#### 여기서 컴포넌트 스캔 방식 대신 자바 코드로 스프링 빈 사용한 이유는 향후 메모리 리포지토리를 다른 리포지토리로 변경할 예정이기 때문이다.

