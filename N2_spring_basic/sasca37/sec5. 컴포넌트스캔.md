## @ComponentScan

- 컴포넌트 스캔은 @Component가 붙은 클래스를 스캔해서 스프링 빈에 등록한다.
- @Configuration도 @Component가 포함되어 있어서 빈에 등록 된다. 

- Default 스캔 범위는 컴포넌트 스캔을 한 클래스의 패키지 이하 전부이다.

``` java
@Configuration
@ComponentScan(
    	//member 패키지 이하만 컴포넌트 스캔의 대상이 된다.
        basePackages = "hello.core.member",
        // 스캔을 안할 대상을 지정 (수동 설정 정보랑 출동 방지)
        excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Configuration.class)
)
public class AutoAppConfig {

}
```



``` java
@Component
public class MemberServiceImpl implements MemberService {

    private final MemberRepository memberRepository;

    @Autowired // 자동 의존관계 주입 
    public MemberServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
```

- @Component만 설정했을 경우 의존관계가 설정이 안된다. 
- @Autowired를 통해 자동으로 의존관계를 주입할 수 있도록 설정한다. 
- ac.getBean(MemberRepository.class) 와 비슷하게 동작을 한다. 
- @Bean으로 직접 설정하고 의존관계를 직접 명시했지만 @Autowired로 해결 가능 하다.



![sec5 캡처1](https://user-images.githubusercontent.com/81945553/127714860-05d435cf-a9d6-4004-8cc3-a21d2b56c310.png)


- 스프링 빈의 기본 이름은 클래스명을 사용하되, 앞 글자만 소문자로 사용

- 직접 이름을 부여하고 싶으면 @Component("memberService2") 와 같이 구현이 가능하다. 



![sec5 캡처2](https://user-images.githubusercontent.com/81945553/127714872-42f57bf4-92ed-4592-978f-91f66959701e.png)

- 생성자에 @Autowired를 지정하면, 스프링 컨테이너가 자동으로 해당 스프링 빈을 찾아서 주입한다.

- 이때 기본 조회 전략은 타입이 같은 빈을 찾아서 주입한다.

   - MemberRepository를 들어가서 MemoryMemberRepository를 가져와 등록한다.

   - 같은 타입이 여러 개일 경우? : 추후 정리

     


## @ComponentScan 기본 대상

- @Controller : 스프링 MVC 컨트롤러로 인식 
- @Repository : 스프링 데이터 접근 계층으로 인식하고, 데이터 계층의 예외를 스프링 예외로 변환해준다. 
- @Configuration : 앞서 보았듯이 스프링 설정 정보로 인식하고, 스프링 빈이 싱글톤을 유지하도록 추가 처 리를 한다. 
- @Service : 사실 @Service 는 특별한 처리를 하지 않는다. 대신 개발자들이 핵심 비즈니스 로직이 여기에 있겠구나 라고 비즈니스 계층을 인식하는데 도움이 된다. 
-  useDefaultFilters 옵션은 기본으로 켜져있는데, 이 옵션을 끄면 기본 스캔 대상들이 제외된다. 그 냥 이런 옵션이 있구나 정도 알고 넘어가자



## 필터 

- includeFilters : 컴포넌트 스캔 대상을 추가로 지정한다. 
- excludeFilters : 컴포넌트 스캔에서 제외할 대상을 지정한다.



```java
@Target(ElementType.TYPE) //TYPE : 클래스 LEVEL
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyExcludeComponent {
}
```

- 필터 확인을 위한 커스텀 어노테이션 생성
- MyIncludeComponent 도 동일하게 생성



``` java
public class ComponentFilterAppConfigTest {
    @Test
    void filterScan() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(ComponentFilterAppConfig.class);
        BeanA beanA = ac.getBean("beanA", BeanA.class);
        assertThat(beanA).isNotNull();

        BeanB beanB = ac.getBean("beanB", BeanB.class);
        assertThrows(NoSuchBeanDefinitionException.class,
                () ->ac.getBean("beanB", BeanB.class) );
    }


    @Configuration
    @ComponentScan(
            includeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = MyIncludeComponent.class),
            excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = MyExcludeComponent.class)
    )
    static class ComponentFilterAppConfig {

    }
}
```

- 테스트를 통해 필터 결과 확인 
- FilterType은 5가지 옵션이 있다.
   -	ANNOTATION: 기본값, 애노테이션을 인식해서 동작한다. ex) org.example.SomeAnnotation 
   -	ASSIGNABLE_TYPE: 지정한 타입과 자식 타입을 인식해서 동작한다. ex) org.example.SomeClass 
   -	AspectJ 패턴 사용 ex) org.example..*Service+ *
   -	*REGEX: 정규 표현식 ex) org\.example\.Default.* 
   -	CUSTOM: TypeFilter 이라는 인터페이스를 구현해서 처리 ex) org.example.MyTypeFilter
-  @Component면 충분하기 때문에 보통은 includeFilters는 사용할일이 없다.



## 중복 등록과 충돌

- 컴포넌트 스캔에서 같은 빈 이름을 등록하면 어떻게 될까?

- 자동 빈등록 vs 자동 빈 등록

  - 이름이 같은 경우 스프링은 오류를 발생시킨다. ConflictingBeanDefinitionException 예외 발생

- 수동 빈등록 vs 자동 빈 등록 

  -  해당 경우는 수동 빈 등록이 우선권을 가진다. (수동 빈이 자동 빈을 오버라이딩 해버린다.)
  -  최근 스프링 부트에서는 개발자의 의도라고 판단되지 않기 때문에 오류가 발생하도록 바꼈다
