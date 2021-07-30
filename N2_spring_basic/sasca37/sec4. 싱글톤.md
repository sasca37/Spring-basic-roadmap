## 순수 DI 컨테이너

![sec4캡처1](https://user-images.githubusercontent.com/81945553/127704760-224cc899-3b7b-4ee5-9f80-558c780b607a.png)

-   요청이 올때마다 객체가 새로 생성한다.
-   또한 이 객체가 DI 관계가 있다면 연관된 객체마저도 새로 생성됨으로 메모리 낭비가 심해진다.
-   해결 방안은 객체를 1번만 생성하고, 공유하도록 설계 해야 한다. -> 싱글톤 패턴

## 싱글톤 패턴

-   클래스 인스턴스가 1개만 생성되는 것을 보장하는 디자인 패턴
-   객체 인스턴스를 2개 이상 생성하지 못하도록 막아야 한다.
    -   즉, private 생성자를 통해 외부에서 임의로 new 키워드를 사용하지 못하도록 막아야 한다.

```
  public class SingletonService {

      //1. 자기 자신을 static영역으로 가지면 객체가 하나만 존재한다.
      private static final SingletonService instance = new SingletonService();

      //2. public 으로 객체 인스턴스가 필요하면 이 static 메서드를 통해서만 조회되도록 한다.
      public static SingletonService getInstance() {
          return instance;
      }

      //3. 외부에서 new 연산자로 객체 생성하는 것을 막는다.
      private SingletonService(){}

      public void logic () {
          System.out.println("싱글톤 객체 호출");
      }

  }
```

![sec4캡처2](https://user-images.githubusercontent.com/81945553/127704798-1c624bde-acff-4764-84d8-d748ee1e73cc.png)

-   외부에서 객체 생성시 오류 발생

```
@Test
    @DisplayName("싱글톤 패턴 적용한 객체 사용 ")
    void singletonServiceTest() {
        SingletonService singletonService1 = SingletonService.getInstance();
        SingletonService singletonService2 = SingletonService.getInstance();

        //isEqualTo : equals , isSameAs : ==
        assertThat(singletonService1).isSameAs(singletonService2);

    }
```

-   동일한 객체인지 테스트 확인 (테스트 하는 습관을 들이자.)

## 싱글톤 패턴 문제점

-   싱글톤 패턴을 구현하는 코드가 많이 들어간다.
    -   logic() 메서드 하나 쓰려고하는데 여러 코드가 들어감.
-   의존관계상 클라이언트가 구체 클래스에 의존하게됨. -> DIP, OCP 위반
-   테스트하기 어렵다.
-   내부 속성을 변경하거나 초기화 하기 어렵다.
-   private 생성자로 자식 클래스를 만들기 어렵다.
-   유연성이 떨어지게 되고(DI 적용이 어려워짐), 안티패턴으로 불리기도 한다.

## 스프링 컨테이너

-   스프링 컨테이너는 자동으로 싱글톤 패턴으로 객체를 생성 및 관리(싱글톤 레지스트리)한다.
    -   성능 향상
    -   빈 이름(메서드 이름), 빈 객체로 저장 (EX : memberService, MemberService@x01)
-   기존의 싱글톤 패턴의 문제점을 모두 해결해준다. (DIP, OCP, 테스트, private 등 해결)

```
    @Test
    @DisplayName("스프링 컨테이너와 싱글톤")
    void springContainer() {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

        MemberService memberService1 = ac.getBean("memberService", MemberService.class);
        MemberService memberService2 = ac.getBean("memberService", MemberService.class);

        // 참조 값이 같은 것을 확인
        System.out.println("memberService1 = " + memberService1);
        System.out.println("memberService2 = " + memberService2);

        assertThat(memberService1).isSameAs(memberService2);
    }
```

-   클라이언트의 요청이 올때마다, 이미 만들어진 객체를 재사용 할 수 있다.
-   기존에 싱글톤패턴의 설정 없이 스프링 컨테이너(AnnotationConfigApplicationContext) 사용만으로
-   싱글톤 패턴 적용이 가능하다.
-   요청할 때마다 새로운 객체를 생성해서 반환하는 방법도 제공하긴 한다.
    -   Bean 스코프 (거의 쓰지 않음), httpSession 등 특정 경우에만 사용 한다.

## 싱글톤 패턴 주의점

-   하나의 같은 객체의 인스턴스를 공유하기 때문에 싱글톤 객체는 상태를 유지(stateful)하게 설계하면 안된다.
-   무상태(stateless)로 설계해야 한다.
    -   특정 클라이언트에 의존적인 필드가 있으면 안된다.
    -   특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다.
    -   가급적 읽기만 가능해야 한다.
    -   필드 대신에 공유되지 않는 지역변수, 파라미터, ThreadLocal 등을 사용해야 한다.
-   스프링 빈의 필드에 공유 값을 설정하면 큰 장애가 발생할 수있다.

```
public class StatefulService {

    private int price; //상태 유지 필드

    public void order(String name, int price) {
        System.out.println("name = " + name + "price = " +price);
        this.price = price; //여기가 문제!
    }

    public int getPrice() {
        return price;
    }
}
```

-   상태를 유지하는 필드를 생성

```
class StatefulServiceTest {

    @Test
    void statefulServiceSingleton() {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
        StatefulService statefulService1 = ac.getBean(StatefulService.class);
        StatefulService statefulService2 = ac.getBean(StatefulService.class);

        //Thread A : A 사용자 10000원 주문
        statefulService1.order("userA", 10000);
        //Thread B : B 사용자 20000원 주문
        statefulService2.order("userA", 20000);

        //사용자 A가 주문 금액 조회
        int price = statefulService1.getPrice();
        System.out.println("A : price = " + price);
    }

    static class TestConfig {

        @Bean
        public StatefulService statefulService() {
            return new StatefulService();
        }
    }
}
```

-   A 사용자가 주문 후 조회하는 사이에 B사용자가 주문 시 A 사용자의 주문 조회 결과가 B사용자의 주문 조회로
-   출력 된다. (10000원이 나와야하는데 20000원이 나오는 문제 발생)
-   이런 장애는 찾기도 어렵기 때문에 정말 중요하다.
-   공유 필드는 정말 조심해야한다. 스프링 빈은 항상 무상태(stateless)로 설계하자.

```
public class StatefulService {

//    private int price; //상태 유지 필드

    public int order(String name, int price) {
        System.out.println("name = " + name + "price = " +price);
//        this.price = price; //여기가 문제!
        return price;
    }

}
```

-   공유 필드를 없애고 파라미터로 해결한다.

```
class StatefulServiceTest {

    @Test
    void statefulServiceSingleton() {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
        StatefulService statefulService1 = ac.getBean(StatefulService.class);
        StatefulService statefulService2 = ac.getBean(StatefulService.class);

        //Thread A : A 사용자 10000원 주문
        int userAPrice = statefulService1.order("userA", 10000);
        //Thread B : B 사용자 20000원 주문
        int userBPrice = statefulService2.order("userB", 20000);

        //사용자 A가 주문 금액 조회
//        int price = statefulService1.getPrice();
        System.out.println("A : price = " + userAPrice);

    }

    static class TestConfig {

        @Bean
        public StatefulService statefulService() {
            return new StatefulService();
        }
    }
}
```
