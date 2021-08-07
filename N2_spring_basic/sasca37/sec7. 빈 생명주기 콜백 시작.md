## 빈 생명주기 콜백 시작

-   DB 커넥션 풀이나, 네트워크 소켓처럼 애플리케이션 시작 시점에 필요한 연결을 미리 해두고,
-   종료 시점에 모두 종료하는 작업을 진행하려면, 객체의 초기화와 종료 작업이 필요하다.

```
public class NetworkClient {

    private String url;

    public NetworkClient() {
        System.out.println("생성자 호출, url = " + url);
        connect();
        call("초기화 연결 메시지");    }

    public void setUrl(String url) {
        this.url = url;
    }

    //서비스 시작시 호출
    public void connect() {
        System.out.println("connect" + url);
    }

    public void call(String message) {
        System.out.println("call: " + url + " message = " +message);
    }

    //서비스 종료시 호출
    public void disconnect() {
        System.out.println("close " + url);
    }
}
----------------------------------------------------------------------------------
@Test
    public void lifeCycleTest() {
        // 해당 구문 생성하는 동안 컨테이너 생성~초기화 콜백 까지 동작한다.
        ConfigurableApplicationContext ac = new AnnotationConfigApplicationContext(LifeCycleConfig.class);
        NetworkClient client = ac.getBean(NetworkClient.class);
        ac.close();
    }

    @Configuration // 수동 빈등록을 위한 설정정보 어노테이션 (@Component 포함)
    static class LifeCycleConfig {
        @Bean // 빈등록 
        public NetworkClient networkClient() {
            NetworkClient networkClient = new NetworkClient();
            networkClient.setUrl("http://aa.com");
            return networkClient;
        }
    }
```

-   해당 값이 null 이 나온다.
-   스프링 빈 라이프 사이클 (싱글톤)
    -   컨테이너 생성 - > 빈 생성 -> 의존 관계 주입 -> 초기화 콜백 -> 실제 동작 -> 소멸전 콜백 -> 종료
    -   초기화 작업은 의존 관계 주입이 모두 완료되고 난 다음에 호출해야 한다.
    -   스프링은 완료된 시점은 콜백 메서드를 통해서 알려주는 기능이 있다.
    -   스프링 컨테이너 종료되기 직전에 (싱글톤) 소멸 콜백을 준다.
-   객체의 생성과 초기화를 분리하자.
    -   생성에서 생성자는 필수 정보(파라미터)를 받고, 메모리를 할당해서 객체를 생성하는 책임
    -   초기화는 생성된 값들을 활용해서 외부 커넥션을 연결하는 등 무거운 동작을 수행
    -   이 둘을 분리하는 것이 유지보수 관점에서 좋다.



## InitializingBean, DisposableBean 인터페이스

```
public class NetworkClient implements InitializingBean, DisposableBean {
    ...
        @Override
    // 의존관계 주입이 끝나면 알아서 호출해주겠다. (InitializingBean)
    public void afterPropertiesSet() throws Exception {
        connect();
        call("초기화 연결 메시지");
    }

    @Override
    // 소멸 지원(DisposableBean)
    public void destroy() throws Exception {
        disconnect();
    }
```

-   이 인터페이스는 스프링 전용 인터페이스이므로 스프링에 의존한다.
-   초기화, 소멸 메서드명을 변경할 수없으며, 외부 라이브러리에 적용할 수 없다.

## 빈 등록 초기화, 소멸 메서드

```
public class NetworkClient {
    ...
 public void init() throws Exception {
        connect();
        call("초기화 연결 메시지");
    }


 public void close() throws Exception {
        disconnect();
 }
---------------------------------------
@Configuration
    static class LifeCycleConfig {
        @Bean(initMethod = "init", destroyMethod = "close") 
        public NetworkClient networkClient() {
            NetworkClient networkClient = new NetworkClient();
            networkClient.setUrl("http://aa.com");
            return networkClient;
        }
}
```

-   설정 정보 사용으로 스프링 코드에 의존하지 않고, 외부 라이브러리에도 적용이 가능하다.
-   destroyMethod 는 Default 값이 추론이다. close, shutdown 을 추론해준다.
    -   destroyMethod="" 설정시 추론기능을 적용하지 않을 수 있다.

## @PostConstruct, @PreDestroy 어노테이션

```
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;
public class NetworkClient {
    ...
     @PostConstruct
    public void init() throws Exception {
        connect();
        call("초기화 연결 메시지");
    }

    @PreDestroy
    public void close() throws Exception {
        disconnect();
 }
---------------------------------------
@Configuration
    static class LifeCycleConfig {
        @Bean
        public NetworkClient networkClient() {
            NetworkClient networkClient = new NetworkClient();
            networkClient.setUrl("http://aa.com");
            return networkClient;
        }
}
```

-   최신 스프링에서 권장하는 방법
-   컴포넌트 스캔과 잘어울리며 JSR-250이라는 표준 라이브러리다.
-   유일한 단점은 외부 라이브러리 적용하지 못한다는 것이다.
    -   외부 라이브러리르 초기화, 종료해야 하면 @PostConstruct, @PreDestroy를 사용하자.