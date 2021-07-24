## 스프링 빈과 의존관계

### 스프링 빈이란? 
Spring IoC 컨테이너가 관리하는 자바 객체를 빈(Bean)이라는 용어로 부른다.  
우리가 new 연산자로 어떤 객체를 생성했을 때 그 객체는 빈이 아니다.  
ApplicationContext.getBean()으로 얻어질 수 있는 객체는 빈이다.  
즉, Spring에서의 빈은 ApplicationContext가 만들어서 그 안에 담고있는 객체를 의미한다. 

※ 스프링은 스프링 컨테이넌에 빈을 등록할 때 기본으로 싱글톤으로 등록한다. (하나의 객체만 등록하여 관리)  
   설정으로 싱글톤이 아니게 설정할 수 있지만, 대부분 싱글톤 사용 





![sec2캡쳐2](https://user-images.githubusercontent.com/81945553/126857978-f0855a3f-55f1-41c6-a06c-e0c3b64a87de.png)
- 스프링이 처음 실행될 때 스프링 컨테이너가 생성되고 스프링 컨테이너에서 @Controller, @Service, @Repository 등 어노테이션이 붙은 객체를 생성하여 관리한다.

### 스프링 빈을 등록하는 방법 
- 컴포넌트 스캔과 자동 의존관계 설정
- 자바 코드로 직접 스프링 빈 등록 

### 컴포넌트 스캔 원리 
- @Component 어노테이션이 있으면 스프링 빈으로 자동 등록 된다.
- @Controller, @Service, @Repository 등은 @Component를 포함하는 어노테이션으로 자동 등록이 된다.

```Java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller {

	/**
	 * The value may indicate a suggestion for a logical component name,
	 * to be turned into a Spring bean in case of an autodetected component.
	 * @return the suggested component name, if any (or empty String otherwise)
	 */
	@AliasFor(annotation = Component.class)
	String value() default "";

}
```

## 의존관계 설정 

![sec4캡쳐1](https://user-images.githubusercontent.com/81945553/126858220-c6e9d8ad-cbf9-4404-bc2f-a7a5df89fa7e.png)

- Controller는 Service를 , Service는 Repository를 의존한다. 이를 의존 관계라고 말한다. 
- Controller, Service 의존 관계 설정 예시 
```Java
private final MemberService memberService = new MemberService(); 
```
- new 연산자 사용시 해당 컨트롤러가 아닌 다른 컨트롤러에서도 가져다 쓸 수 있는 문제점 발생 (사용하지 않는다)

```Java
@Controller
public class MemberController {
    private final MemberService memberService; // 해당 클래스에 @Service 등록 필요

    @Autowired // 의존관계 주입 DI 중 생성자 주입
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }
 /*
    // 의존관계 DI 중 세터주입
    @Autowired
    public void setMemberService(MemberService memberService) {
        this.memberService = memberService;
    }
    //의존관계 주입 DI 중 필드 주입
    @Autowired  private  MemberService memberService;
*/
```
- final 키워드는 무조건 초기화가 이루어져야한다. 생성자를 통해 초기화를 해주거나 바로 초기화를 해줘야한다. (실수 예방)
- @Autowired 를 통해 의존 관계 주입 (Dependency Injection), 생성자, 세터, 필드 DI 존재  
 ※ @Autowired는 스프링 빈에 등록이 된 객체에서만 동작한다.
- DI에서 의존관계가 실행중에 동적으로 변하는 경우가 거의 없으므로 생성자 주입을 권장한다. 

## 자바 코드로 직접 스프링 빈 등록 
- 주로 정형화 되지 않거나, 상황에 따라 구현 클래스를 변경해야 하면 설정을 통해 스프링 빈으로 등록한다. 

```Java
package hello.hellospring;
import hello.hellospring.repository.MemberRepository;
import hello.hellospring.repository.MemoryMemberRepository;
import hello.hellospring.service.MemberService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SpringConfig {

 @Bean
 public MemberService memberService() {
 return new MemberService(memberRepository());
 }
 
 @Bean
 public MemberRepository memberRepository() {
return new MemoryMemberRepository();
 }
}
```
- @Service, @Repository, @Autowired 등 어노테이션 사용하지 않고 @Configuration 과 @Bean을  
  사용하여 직접 스프링 빈에 등록한다. 
