## 빈 스코프

-   빈이 존재할 수 있는 범위
-   싱글톤 : 기본 스코프, 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프
-   프로토타입 : 컨테이너는 프로토타입 빈의 생성과 의존관계 주입까지만 관여하는 짧은 범위의 스코프
-   웹 관련 스코프
    -   request : 웹 요청이 들어오고 나갈 때까지 유지되는 스코프
    -   session : 웹 세션이 생성되고 종료될 때 까지 유지되는 스코프
    -   application : 웹의 서블릿 컨텍스트와 같은 범위로 유지되는 스코프

## 빈 요청

### 싱글톤 빈 요청

![sec9캡처1](https://user-images.githubusercontent.com/81945553/128006178-08ec6a07-712d-47f0-8a20-7b0d395eddef.png)

-   싱글톤 스코프의 빈을 스프링 컨테이너에 요청
-   스프링 컨테이너는 관리하는 스프링 빈을 반환
-   같은 요청이와도 같은 객체 인스턴스의 스프링 빈을 반환

### 프로토타입 빈 요청

-   프로토타입 빈은 스프링컨테이너가 빈을 생성, 의존관계 주입, 초기화 까지만 처리한다.
-   빈 관리 책임은 클라이언트한테 있다. (@PreDestroy 사용 못한다)

![sec9캡처2](https://user-images.githubusercontent.com/81945553/128006183-16e36d8f-d198-441d-be71-4c3fe9abce74.png)

-   프로토타입 스코프의 빈을 스프링 컨테이너에 요청
-   스프링 컨테이너는 프로토타입 빈을 생성하고, 의존관계 주입

![sec9캡처3](https://user-images.githubusercontent.com/81945553/128006184-019f5b0a-65bc-4d0c-b580-77feb966d1b7.png)

-   스프링 컨테이너는 생성한 프로토타입 빈을 클라이언트에 반환
-   같은 요청이오면 새로운 프로토타입 빈을 생성해서 반환

```
public class PrototypeTest {

    @Test
    void prototypeBeanFind() {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class);
        System.out.println("find proto1");
        PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);
        System.out.println("find proto2");
        PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);

        System.out.println("prototypeBean1 = " + prototypeBean1);
        System.out.println("prototypeBean2 = " + prototypeBean2);
        assertThat(prototypeBean1).isNotSameAs(prototypeBean2);

        ac.close(); //close 안됨 만들고 버리기 때문 
    }

    @Scope("prototype")
    static class PrototypeBean {
        @PostConstruct
        public void init() {
            System.out.println("prototype.init");
        }

        @PreDestroy
        public void destroy() {
            System.out.println("prototype.destroy");
        }
    }
}
-----------------------------------------------------------------
find proto1
prototype.init
find proto2
prototype.init
prototypeBean1 = hello.core.scope.PrototypeTest$PrototypeBean@413f69cc
prototypeBean2 = hello.core.scope.PrototypeTest$PrototypeBean@1f53a5dc
```

-   싱글톤 빈은 스프링 컨테이너 생성 시점에 초기화 메서드가 실행 되지만, 프로토타입 스코프의 빈은
-   스프링 컨테이너에서 빈을 조회할 때 생성되고, 초기화 메서드도 실행된다.
-   요청이 올때마다 빈이 새로 생성 되는 것을 알 수 있다.
-   스프링 컨테이너는 프로토타입 빈 생성 , 의존관계 주입, 초기화까지만 관여한다. (종료 메서드 호출X)

## 싱글톤, 프로토타입 빈을 함께 사용

### 문제점

![sec9캡처4](https://user-images.githubusercontent.com/81945553/128006186-413ab676-2b34-4282-9bff-e5e2cb139a7b.png)

```
public class SingletonWithPrototypeTesst1 {

    @Test
    void prototypeFind() {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class);
        PrototypeBean prototypeBean1= ac.getBean(PrototypeBean.class);
        prototypeBean1.addCount();
        assertThat(prototypeBean1.getCount()).isEqualTo(1);

        PrototypeBean prototypeBean2= ac.getBean(PrototypeBean.class);
        prototypeBean2.addCount();
        assertThat(prototypeBean2.getCount()).isEqualTo(1);
    }

    @Test
    void singletonClientUsePrototype() {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class, ClientBean.class);

        ClientBean clientBean1 = ac.getBean(ClientBean.class);
        int count1 = clientBean1.logic();
        assertThat(count1).isEqualTo(1);

        ClientBean clientBean2 = ac.getBean(ClientBean.class);
        int count2 = clientBean2.logic();
        assertThat(count2).isEqualTo(2);
    }

    @Scope("singleton") // 디폴트 값이지만 이해를 위해 명시
    static class ClientBean {
        private final PrototypeBean prototypeBean; // 생성시점에 주입

        @Autowired
        public ClientBean(PrototypeBean prototypeBean) {
            this.prototypeBean = prototypeBean;
        }

        public int logic() {
            prototypeBean.addCount(); // 몇번의 다른객체로 호출을 해도 prototype 객체는 그대로다.
            int count = prototypeBean.getCount();
            return count;
        }
    }

    @Scope("prototype")
    static class PrototypeBean {
        private int count = 0;

        public void addCount() {
            count++;
        }

        public int getCount() {
            return count;
        }

        @PostConstruct
        public void init() {
            System.out.println("init = " + this);
        }

        @PreDestroy
        public void destroy() {
            System.out.println("destroy");
        }

    }
}
```

-   clientBean은 항상 같은 객체를 반환한다.
-   안에 있는 프로토타입빈은 의존관계 까지 주입하고 종료되어 종료된 빈을 clientBean이 가지고 있게 된다.
-   프로토타입 빈 호출 시마다 새로운 객체를 반환받고 싶었으나, 이전 객체를 반환 받는다.

### Provider로 해결

```
static class ClientBean {
@Autowired // 좋지 않은 방법
private ApplicationContext ac;
public int logic() {
 PrototypeBean prototypeBean = ac.getBean(PrototypeBean.class);
 prototypeBean.addCount();
 int count = prototypeBean.getCount();
 return count;
}
```

-   싱글톤 빈이 프로토타입을 사용할 때 마다 스프링 컨테이너에 새로 요청하는 방법
-   의존 관계 주입(DI)가 아닌 의존관계 탐색(DL)의 예시로 코드가 종속적이고, 테스트가 어려워진다.
    -   DL : Defendency Lookup
-   이러한 DL의 기능을 제공해주는 것이 Provider이다.

```
 @Scope("singleton") 
    static class ClientBean {

        @Autowired
        private ObjectProvider<PrototypeBean> prototypeBeanProvider;

        public int logic() {
            PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
            prototypeBean.addCount(); 
            int count = prototypeBean.getCount();
            return count;
}
```

-   ObjectFactory 도 사용가능 (getObject 기능만 존재), ObjectProvider는 추가 기능 존재
-   prototypeBeanProvider.getObject()를 통해 항상 새로운 프로토타입 빈이 생성된다.
-   프로토타입 전용이라기 보단 대신 가져와주는 존재 정도로 이해하자. (DL 기능)
-   스프링에 의존적이다. (import org.springframework.beans.factory.ObjectProvider;)

```
import javax.inject.Provider;
@Scope("singleton") 
    static class ClientBean {

        @Autowired
        private Provider<PrototypeBean> prototypeBeanProvider; //Object

        public int logic() {
            PrototypeBean prototypeBean = prototypeBeanProvider.get();
            prototypeBean.addCount(); 
            int count = prototypeBean.getCount();
            return count;
        }
    }
```

-   JSR330 Provider (javax.inject.Provider)
-   implementation 'javax.inject:javax.inject:1' 을 gradle 에 추가
-   스프링에 의존적이지 않은 자바 표준 Provider 사용 (스프링이 아닌 다른 컨테이너에서도 사용 가능)

**스프링이 더 다양한 기능을 제공해주는 경우가 많아서 스프링을 쓰자**

## 웹스코프

-   웹 스코프는 웹 환경에서만 동작
-   스코프의 종료시점까지 관리 (종료 메서드 호출된다.)

### 웹스코프 종류

-   request: HTTP 요청 하나가 들어오고 나갈 때 까지 유지되는 스코프, 각각의 HTTP 요청마다 별도의 빈 인스턴스가 생성되고, 관리된다.
-   session: HTTP Session과 동일한 생명주기를 가지는 스코프
-   application: ServletContext와 동일한 생명주기를 가지는 스코프
-   websocket: 웹 소켓과 동일한 생명주기를 가지는 스코프

```
//web 라이브러리 추가, 포트 변경 시 server.port=포트번호 (application.properties)
implementation 'org.springframework.boot:spring-boot-starter-web'
```

### request 스코프

![sec9캡처5](https://user-images.githubusercontent.com/81945553/128006193-bfeea379-11b5-4375-a37b-b0b7fcfc9fea.png)

-   Http request 요청 당 각각 할당되는 request 스코프

#### request 스코프 예제

![sec9캡처6](https://user-images.githubusercontent.com/81945553/128006194-426bcdf7-5f57-4507-adf4-0a5d24a7c9c2.png)

-   해당 로그가 남도록 request 스코프를 활용해서 추가 기능 개발
-   UUID를 사용해서 HTTP 요청을 구분하자.
-   requestURL 정보도 추가로 넣어서 어떤 URL을 요청해서 남은 로그인지 확인하자.

```
@Component
@Scope(value = "request")
public class MyLogger {

    private String uuid;
    private String requestURL;

    public void setRequestURL(String requestURL) {
        this.requestURL = requestURL;
    }

    public void log(String message) {
        System.out.println("[" + uuid + "]" +"[" + requestURL + "] " + message);
    }

    @PostConstruct
    public void init() {
        uuid = UUID.randomUUID().toString();
        System.out.println("[" + uuid + "] request scope bean create: " + this);
    }

    @PreDestroy
    public void close() {
        System.out.println("[" + uuid + "] request scope bean close: " + this);
    }
}
```

-   로그를 출력하기 위한 MyLogger 클래스
-   request 스코프 지정으로 HTTP 요청당 빈이 하나씩 생성되고 HTTP 요청이 끝나는 시점에 소멸된다.
-   requestURL은 빈이 생성되는 시점에는 알 수 없으므로 외부에서 setter로 입력 받는다.

```
@Controller
@RequiredArgsConstructor
public class LogDemoController {

    private final LogDemoService logDemoService;
    //Dependency Lookup 기능 사용, 사용안하면 request 스코프 실행 오류 발생
    private final ObjectProvider<MyLogger> myLoggerProvider;

    @RequestMapping("log-demo")
    @ResponseBody //데이터 반환
    public String logDemo(HttpServletRequest request) throws InterruptedException {
        MyLogger myLogger = myLoggerProvider.getObject();
        String requestURL = request.getRequestURL().toString();
        myLogger.setRequestURL(requestURL);

        myLogger.log("controller test");
        Thread.sleep(1000); // 요청 시간 확인 테스트
        logDemoService.logic("testId");
        return "OK";
    }
}
-------------------------------------------------------------------------------
@Service
@RequiredArgsConstructor
public class LogDemoService {
    //Dependency Lookup 기능 사용, 사용안하면 request 스코프 실행 오류 발생
    private final ObjectProvider<MyLogger> myLoggerProvider;

    public void logic(String id) {
        // 기존 MyLogger를 DL의 getObject로 가져온다.
        MyLogger myLogger = myLoggerProvider.getObject();
        myLogger.log("service id = " + id);
    }
}  
```

-   ObjectProvider를 사용하여 getObject를 호출하는 시점까지 빈의 생성을 지연
    -   호출하는 시점에는 HTTP 요청이 진행중이므로 빈의 생성이 정상 처리 (아니면 오류 발생 )
-   MyLogger를 저장하는 부분은 Controller 보다는 스프링 인터셉터나 서블릿 필터에서 활용하는 것이 좋다.
    -   예제의 단순화를 위해 Controller 사용
    -   request scope를 사용하지 않고 정보를 서비스 계층에 넘긴다면 유지보수가 어렵다.

### 웹스코프와 프록시

```
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS) //값이 2개 이상 쓰려면 value 사용
public class MyLogger {
...
```

-   proxyMode = ScopedProxyMode.TARGET\_CLASS 추가 하나로 Provider를 대신할 수 있다.
    -   적용 대상이 클래스면 TARGET\_CLASS
    -   인터페이스면 INTERFACES 선택
    -   MyLogger의 가짜 프록시 클래스를 만들어두고 HTTP request와 상관없이 가짜 프록시 클래스를 다른 빈에 미리 주입해 둘 수 있다.
-   Controller에서 로그를 찍어보면 MyLogger$$EnhancerBySpringCGLIB$$e3e8d27f를 확인 할 수 있다.

#### 동작 원리

![sec9캡처7](https://user-images.githubusercontent.com/81945553/128006199-689c7392-bfd8-4464-99b3-7ed26ff61b28.png)

-   CGLIB 라이브러리를 이용해 바이트코드를 조작하여 MyLogger를 상속받은 가짜 프록시 객체 생성
-   스프링 컨테이너에 myLogger 이름으로 가짜 프록시 객체 등록
    -   ac.getBean("myLogger", MyLogger.class)로 조회 가능
    -   의존 관계 주입도 가짜 프록시가 주입
    -   가짜 프록시 객체는 내부에 진짜 myLogger를 찾는 법을 알고 있다.
    -   클라이언트 logic() 호출 -> 가짜 프록시 logic() 동작 -> 원본 logic() 호출
    -   클라이언트 입장에서는 원본인지 아닌지 모르게 동작 (다형성)
    -   가짜 프록시 객체는 실제 request scope와 아무런 관계가 없고, 단순 위임 로직만 있으며,
    -   싱글톤 처럼 동작한다.

**어노테이션 설정만으로 원본 객체를 프록시로 대체할 수있는 것이 다형성과 DI 컨테이너의 큰 장점이다.**

**무분별하게 사용하면 유지보수가 어려워질 수 있으니 최소한으로 사용하자**