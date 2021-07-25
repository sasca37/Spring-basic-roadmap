## AOP란? 
- Spring은 Spring Triangle이라고 부르는 세 가지 개념을 제공해준다. 각각 IoC, AOP, PSA를 일컫는다.
- AOP는 Aspect Oriented Programming의 약자로 '측면/양상 지향적인 프로그래밍'이라는 의미이다.
- ※ PSA (Portable Service Abstraction) : 추상화 계층을 사용하여 어떤 기술을 내부에 숨기고 개발자에게 편의성을 제공해주는 것

## AOP 개념 
- Aspect : 흩어진 관심사를 모듈화
- Target : Aspect를 적용하는 곳 (클래스, 메서드 등 )
- Advice : 실질적으로 어떤 일을 해야할 지에 대한 것, 실질적인 부가기능을 담은 구현체
- JointPoint : Advice가 적용될 위치, 끼어들 수 있는 지점. 메서드 진입 지점, 생성자 호출 시점, 필드에서 값을 꺼내올 때 등 다양한 시점에 적용가능
- PointCut : JointPoint의 상세한 스펙을 정의한 것. 'A란 메서드의 진입 시점에 호출할 것'과 같이 더욱 구체적으로 Advice가 실행될 지점을 정할 수 있음

### AOP가 필요한 예시 상황 (시간 측정 기능)
```Java
public List<Member> findMember() {
        long start = System.currentTimeMillis(); // 시간측정 로직
        try {
            return memberRepository.findAll(); //핵심 로직 
        } finally {
            long finish = System.currentTimeMillis(); // 시간측정 로직
            long timeMs = finish - start; // 시간측정 로직
            System.out.println("timeMs = " + timeMs + "ms"); 
        }

    }
   ```
- 시간 측정 로직은 공통 관심 사항이다.
- 시간 측정 기능은 핵심 관심 사항이 아니다.
- 시간 측정 로직과 핵심 비즈니스 로직이 섞여 유지보수가 어렵다.
- 별도의 공통 로직으로 만들기 매우 어렵다.
- 모든 로직을 찾아가면서 변경해야 한다.
- 공통 관심 사항(cross-cutting concern) :시간 측정 로직 vs 핵심 관심 사항(core concern) 분리 필요 

## AOP 적용 
![sec7캡처1](https://user-images.githubusercontent.com/81945553/126908044-3fd3b791-5da7-417f-9846-633a5737eb0b.png)

### AOP 적용 코드 
``` Java
package hello.hellospring.aop;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Component //컴포넌트 스캔을 사용하거나 (빈등록), Configuration 에 직접 빈등록 (이 방법을 주로 선호) 
@Aspect //AOP 적용 어노테이션 
public class TimeTraceAop {
    //hellospring 패키지의 하위 모두에 적용 시키겠다.
    @Around("execution(* hello.hellospring..*(..))")
    // 가짜 Bean 통해 프록시를 생성 후 JoinPoint로 진짜 Bean으로 연결
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        System.out.println("START: " + joinPoint.toString());
        try {
            return joinPoint.proceed(); //인라인으로 합침 (Object result = joinPoint.proceed(); + return result;)
        } finally {
            long finish = System.currentTimeMillis();
            long timeMs = finish - start;
            System.out.println("END: " + joinPoint.toString()+ " " + timeMs + "ms");
        }
    }
}
```
- 공통 관심 사항(cross-cutting concern) :시간 측정 로직 vs 핵심 관심 사항(core concern) 분리
- 유지보수 간단 
- @Around 에서 적용하지 않을 곳도 추가 지정이 가능하다.   
EX) @Around("execution(* hello.hellospring.service..*(..))") : service 하위만 적용 

## AOP 동작 방식 (의존 관계) 
- 프록시 방식의 AOP (DI가 가능하니까 프록시 방식이 가능 (전달할 대상이 누군지 상관 없음)), 실제 자바코드를 넣는 AOP 방식도 있다.
- 프록시 : 프록시 객체는 원래 객체를 감싸고 있는 가짜 객체이다. 
- 프록시패턴 : 가짜 객체가 원래 객체를 감싸서 client의 요청을 처리하게 하는 패턴이다.

※ 프록시 패턴을 쓰는 이유는 접근을 제어하고 싶거나, 부가 기능을 추가하고 싶을 때 사용한다.  
※ .getClass()를 이용해 로그 출력해보면 $$EnhancerBySpringCGLIB$$~ 통해 프록시 적용을 확인할 수 있다. 

![sec7캡처2](https://user-images.githubusercontent.com/81945553/126908498-4c8ab5fd-efff-4cfa-a101-68947d66937d.png)  

- 전체 의존 관계 

![sec7캡처3](https://user-images.githubusercontent.com/81945553/126908529-150486a1-7c68-4722-bdd0-c2ceced3dbee.png)  


