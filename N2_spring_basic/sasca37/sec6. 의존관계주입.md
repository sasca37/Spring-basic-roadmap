# 의존관계 주입 DI

-   생성자 주입
-   수정자 주입(setter)
-   필드 주입
-   일반 메서드 주입

## 생성자 주입

```
@Component
public class OrderServiceImpl implements OrderService{

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
... 
```

-   이름 그대로 생성자를 통해 의존 관계를 주입 받는 방식 (주로 사용하는 방식)
-   스프링이 실행되면서 컴포넌트 스캔을 통해 @Component들이 빈에 등록이 될때 생성자를 호출할 때, @Autowired 가 있으면 스프링 컨테이너에서 생성자의 매개변수에 속해 있는 것들을 꺼내와서 주입해준다.
-   생성자 호출 시점에 딱 1번만 호출하는 것을 보장
-   불변, 필수 의존관계에 사용
    -   setter 메서드를 만들지 않아서, 추후 다른사람이 외부에서 변경하여 버그가 발생하지 않도록 방지
-   생성자가 하나일 경우엔 @Autowired 생략이 가능하다. (추론 가능)

## 수정자 주입 (setter)

```
@Component
public class OrderServiceImpl implements OrderService{

    private  MemberRepository memberRepository;
    private  DiscountPolicy discountPolicy;
@Autowired
    public void setDiscountPolicy(DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
    }
@Autowired
    public void setMemberRepository(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
    ...
```

-   final 키워드를 지우고 생성자 대신 setter를 이용해 주입
-   @Autowired를 안해주면 자동 주입이 안된다. (오류 발생)
    -   @Autowired(required=false)로 지정시 주입할 대상이 없어도 동작하게 실행은 할 수있다.
-   선택, 변경 가능성이 있는 의존관계 사용 (중간에 변경할 일이 있을 경우.. 드물다)

## 필드 주입

```
@Component
public class OrderServiceImpl implements OrderService{

    @Autowired private MemberRepository memberRepository;
    @Autowired private DiscountPolicy discountPolicy;
    ...
    @Override
    public Order createOrder(Long memberId, String itemName, int itemPrice) {
        Member member = memberRepository.findById(memberId);
        int discountPrice = discountPolicy.discount(member, itemPrice);

        return new Order(memberId, itemName, itemPrice, discountPrice);
    }
```

```
 @Test
    void fieldInjectionTest() {
        OrderServiceImpl orderService = new OrderServiceImpl();
        // 값을 넣어 줄 수 없으므로 NullPointException 발생
        orderService.createOrder(1L, "itemA", 10000);
    }
```

-   의존관계를 필드에 바로 주입 (private 만 가능)
-   코드가 간결하지만 외부에서 변경이 불가능해서 테스트하기 어려운 단점이 생겨 권장하지 않는다.
    -   setter를 열어두고 , 테스트에서 값을 넣어줘야한다.
-   DI 프레임워크 없이 순수 자바코드로 아무것도 할 수 없다.
-   실제 애플리케이션 코드와 관계없는 테스트 단계안에서 사용한다. (@SpringBootTest)

## 일반 메서드 주입

```
@Component
public class OrderServiceImpl implements OrderService{

    private  MemberRepository memberRepository;
    private  DiscountPolicy discountPolicy;

    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }

    @Autowired
    public void init(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
```

-   일반 메서드에 의존 관계 주입
-   한번에 여러 필드를 주입 받을 수 있다.
-   생성자 , 수정자 주입 안에서 의존 관계를 해결하기 때문에 사용하지 않는다.

## 옵션 처리

```
@Test
    void AutowiredOption() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(TestBean.class);

    }

    static class TestBean {

        @Autowired(required = false)
        //스프링에 관리되는 빈이 아예 없는 상황
        public void setNoBean1(Member noBean1) {
            System.out.println("noBean1 = " + noBean1);
        }

        @Autowired
        //스프링에 관리되는 빈이 아예 없는 상황
        public void setNoBean2(@Nullable Member noBean2) {
            System.out.println("noBean2 = " + noBean2);
        }

        @Autowired
        public void setNoBean3(Optional<Member> noBean3){
            System.out.println("noBean3 = " + noBean3);
        }
        ...
        결과 :noBean2 = null
 noBean3 = Optional.empty
```

-   @Autowired(required=false) : 자동 주입할 대상이 없으면 메서드 자체가 호출이 안됨
-   @Nullable : 자동 주입할 대상이 없으면 null 입력
-   Optional<> : 자동 주입할 대상이 없으면 Optional.empty 입력

## 생성자 주입을 선택하는 이유

- 대부분 애플리케이션은 종료시점까지 의존관계를 변경할 일이 없다. (불변)

-   수정자 주입을 하려면 setXXX 메서드를 public으로 열어두어야 한다.
    -   누군가 실수로 변겨할 수 있기 때문에 좋은 설계가 아니다.
    
- 객체 생성할 때 한번만 호출 되므로 불변하게 설계할 수 있다.

- final 키워드를 이용해 생성자를 통해서만 값을 지정함으로 변경을 자동 방지해준다.

- 가끔 필수값이 아닌 옵션이 발생하면 수정자 주입을 권장한다. 필드 주입 사용 X


## 조회 빈이 2개 이상일 경우

```
  Caused by: org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'hello.core.discount.DiscountPolicy' available: expected single matching bean but found 2: fixDiscountPolicy,rateDiscountPolicy
```

-   같은 인터페이스에 여러 구현체가 @Component로 등록 되어 있을 경우 해당 오류 발생한다.
-   스프링 빈을 수동으로 등록, 또는 의존 관게 자동 주입에서 해결 가능
-   @Autowired 필드명 매칭
-   @Qualifier -> @Qualifier끼리 매칭 -> 빈 이름 매칭
-   @Primary 사용

### @Autowired 필드명 매칭

```
@Autowired
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy rateDiscountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = rateDiscountPolicy;
    }
```

-   DiscountPolicy 에 RateDiscountPolicy, FixDiscountPolicy 2개 모두 빈 등록 되어 있는 경우
-   필드명, 파라미터명을 rateDiscountPolicy로 지정해주면 @Autowired가 해당 빈으로 인식 한다.
    -   클래스 레벨이 빈 등록될 때 클래스명에 첫글자는 소문자로 등록이 된다.
    -   ex ) RateDiscountPolicy 는 rateDiscountPolicy라는 빈이름으로 등록

### @Qualifier 사용

```
  @Component
  @Qualifier("mainDiscountPolicy")
  public class RateDiscountPolicy implements DiscountPolicy{
  ...
  @Component
  @Qualifier("fixDiscountPolicy")
  public class FixDiscountPolicy implements DiscountPolicy{
  ...
  @Component
  public class OrderServiceImpl implements OrderService{

      private  final MemberRepository memberRepository;
      private  final DiscountPolicy discountPolicy;

      @Autowired
      public OrderServiceImpl(MemberRepository memberRepository, @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
          this.memberRepository = memberRepository;
          this.discountPolicy = discountPolicy;
      }    
  ...
```

-   빈 이름을 변경하는 것이 아닌, @Qualifier는 추가 구분자를 붙여주는 방법이다.
-   @Qualifier를 못찾으면 명시한 이름으로 스프링 빈으로 찾는다. (그래도 없으면 예외 발생)
-   @Qualifier는 찾는 용도로만 사용하자. (헷갈림 방지)

```
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@Qualifier("mainDiscountPolicy")
public @interface MainDiscountPolicy { //커스텀 어노테이션 
} 
---------------------------------------------------------------------------------------
@Component
@MainDiscountPolicy // 커스텀 어노테이션 지정 
public class RateDiscountPolicy implements DiscountPolicy{
...
---------------------------------------------------------------------------------------   
@Component
public class OrderServiceImpl implements OrderService{

    private  final MemberRepository memberRepository;
    private  final DiscountPolicy discountPolicy;

    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, @MainDiscountPolicy DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
...
```

-   @Qualifier에 지정한 문자는 타입 체크가 안된다. (오타시 컴파일 오류가 안나옴)
-   그래서 커스텀 어노테이션을 사용하는 경우가 많다. (오타시 컴파일 오류)

### @Primary 사용

```
@Primary
public class RateDiscountPolicy implements DiscountPolicy{
...
```

-   @Primary가 등록되면 여러 빈중에 최상위 우선순위를 갖게 된다.
    -   우선 순위는 항상 자동 < 수동 , 넓은 선택 < 좁은 선택이다.
    -   @Primary 와 @Qualifier가 모두 적용 되어있을 경우 @Qualifier가 우선 순위가 높다.
    -   보통 2개의 빈을 모두 사용할 때 메인 구현체에 @Primary, 서브에 @Qualifier를 지정하여 사용한다.
-   @Qualifier 보다 간결하여 주로 많이 사용하는 방식이다.

## 조회한 빈이 모두 필요할때 (List, Map)

-   의도적으로 스프링 빈이 다 필요한 경우
-   ex) 할인 서비스에 클라이언트가 여러 할인을 선택할 수 있는 상황
-   스프링에서 소위 말하는 전략 패턴을 매우 간단하게 구현할 수 있다.

```
import hello.core.AutoAppConfig;
import hello.core.discount.DiscountPolicy;
import hello.core.member.Grade;
import hello.core.member.Member;
import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import java.util.List;
import java.util.Map;
import static org.assertj.core.api.Assertions.*;

public class AllBeanTest {

    @Test
    void findAllBean() {
        //AutoAppConfig : 컴포넌트 스캔을 위한 Configuration, DiscountService 빈 등록 
        ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class, DiscountService.class);
        // 등록된 DiscountService 빈을 꺼내옴 
        DiscountService discountService = ac.getBean(DiscountService.class);
        // 테스트 더미 데이터 
        Member member = new Member(1L, "userA", Grade.VIP);
        // discount 메서드를 통해 fix 정책을 적용
        int discountPrice = discountService.discount(member, 10000, "fixDiscountPolicy");

        assertThat(discountService).isInstanceOf(DiscountService.class);
        assertThat(discountPrice).isEqualTo(1000);
        // discount 메서드를 통해 rate 정책을 적용
        int rateDiscountPrice = discountService.discount(member, 10000, "rateDiscountPolicy");
        assertThat(rateDiscountPrice).isEqualTo(2000);
    }

static class DiscountService {
        private final Map<String, DiscountPolicy> policyMap;
        private final List<DiscountPolicy> policies;

        public DiscountService(Map<String, DiscountPolicy> policyMap, List<DiscountPolicy> policies) {
            this.policyMap = policyMap;
            this.policies = policies;
            System.out.println("policyMap = " + policyMap);
            System.out.println("policies = " + policies);
        }

        public int discount(Member member, int price, String discountCode) {
            DiscountPolicy discountPolicy = policyMap.get(discountCode);
            return discountPolicy.discount(member, price);
        }
    }
}
```

-   DiscountService는 Map으로 모든 DiscountPolicy 를 주입받는다. 이때 fixDiscountPolicy , rateDiscountPolicy 가 주입된다. (빈이름의 Default는 첫글자 소문자, 이하 동일 )
-   discount로 넘어온 값으로 스프링 빈을 찾아서 실행한다.
-   List 에 DiscountPolicy 타입으로 조회한 모든 스프링 빈을 담는다.
-   해당 타입의 빈이 없으면 빈컬렉션이나 Map을 주입한다.

## 자동, 수동 의존 관계의 기준

-   어플리케이션 : 업무로직빈, 기술 지원 빈으로 크게 나눌 수 있다.
-   업무로직 빈 (비즈니스 요구사항을 개발 할 때 )
    -   자동 기능을 기본으로 사용하자 (최근 스프링부트는 @Component를 기본으로 갖추고 있음)
    -   자동 빈 등록을 해도 OCP, DIP를 지킬 수 있다.
-   기술 지원 빈 (기술적인 문제나 , 공통 관심사(AOP))을 처리 할 때 주로 사용된다.
    -   애플리케이션에 광범위하게 영향을 미치는 경우
    -   로직 적용 확인 파악하기 어려운 경우
    -   가급적 수동 빈 등록을 사용하여 명확하게 들어내는 것이 좋다. (주로 Root 계층에 지정)
    -   다형성을 활용하여 타인이 파악하기 어려운 경우

````