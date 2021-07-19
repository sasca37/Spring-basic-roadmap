# 1.1 프로젝트 

### 1) 프로젝트 생성

#### 설치

java11.0.11

intelliJ

#### 프로젝트 생성

**start.spring.io 를 이용하여 프로젝트 생성**

1. project 

   - maven vs gradle -> 빌드 도와주는 툴

2. SpringBoot ver 

   - snapshot -> 만들고 있는 버전
   - m1 -> 출시된 버전 아님

3. Project Metadata

   - group : 기업 도메인 명
   - artifact : 빌드 결과물

4. dependencies

   - 사용할 라이브러리

   - spring web, thymleaf(템플릿 엔진)

### 2) 프로젝트 구조

src 아래에 main과 test가 있다.  main 아래에 java, resource는 실제 소스(자바 파일)랑 설정 파일다.  

build.gradle은 mavenCentral() 을 통해 dependencies를 받아오고 빌드하기 쉽게 설정해준다.

![image-20210719144744493](C:\Users\yujin\AppData\Roaming\Typora\typora-user-images\image-20210719144744493.png)

### 3) 프로젝트 실행

환경 셋팅이 다 된 상태에서 main application을 실행하면 스프링부트가 실행된다. 또한, http 8080 포트로 톰캣 서버가 내장되어 있는 것을 확인할 수 있다. 

스프링부트 소스 라이브러리가 톰캣서버를 내장하고 있다.

<img src="C:\Users\yujin\AppData\Roaming\Typora\typora-user-images\image-20210719144710342.png" alt="image-20210719144710342" style="zoom: 67%;" />

- preference 단축키 : ctrl + alt+s
- gradle 거치지 않고 인텔리제이 실행 : preference -> gradle -> intelliJ 로 바꾸기

# 1.2 라이브러리 

>  Gradle은 의존 관계가 있는 라이브러리를 함께 다운로드 한다.

### 1) 스프링 부트 라이브러리

- spring-boot-starter-web
  - spring-boot-starter-tomcat : 톰캣(웹서버)
  - spring-webmvc : 스프링 웹 MVC
- spring-boot-starter-thymeleaf : 타임리프 템플릿 엔진(View)
- spring-boot-starter(공통) : 스프링 부트  + 스프링 코어 + 로깅
  - spring-boot 
    - spring-core
  - spring-boot-starter-logging
    - logback, slf4j

### 2) 테스트 라이브러리

- spring-boot-starter-test
  - junit : 테스트 프레임워크로 5시리즈를 주로 사용
  - mockito : 목 라이브러리
  - assertj : 테스트 코드를 좀 더 편하게 작성하게 도와주는 라이브러리
  - spring-test : 스프링 통합 테스트 지원 (junit 을 스프링과 통합하여 테스트를 도와줌)

# 1.3 View 환경 설정

### 1) 정적 페이지 동작

#### WelcomePage 만들기

- resources/static/index.html 

  ~~~html
  <!DOCTYPE HTML>
  <html>
  <head>
   <title>Hello</title>
   <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
  </head>
  <body>
  Hello
  <a href="/hello">hello</a>
  </body>
  </html>
  ~~~

- 스프링부트가 제공하는 WelcomePage 기능
  - static/index.html 을 올려두면 Welcome page 기능을 제공한다.
  - https://docs.spring.io/spring-boot/docs/2.3.1.RELEASE/reference/html/spring-boot-features.html#boot-features-spring-mvc-welcome-page

### 2) Thymeleaf 페이지 동작

- thymeleaf 공식 사이트: https://www.thymeleaf.org/ 
- 스프링 공식 튜토리얼: https://spring.io/guides/gs/serving-web-content/ 
- 스프링부트 메뉴얼: https://docs.spring.io/spring-boot/docs/2.3.1.RELEASE/reference/ html/spring-boot-features.html#boot-features-spring-mvc-template-engines

- cotroller/HelloController.class

  ~~~java
  @Controller
  public class HelloController {
   @GetMapping("hello")
   public String hello(Model model) {
   model.addAttribute("data", "hello!!");
   return "hello";
   }
  }
  /*
  스프링의 웹의 진입점은 Controller다. 
  어노테이션 GetMapping을 통해 "/hello" url을 연결하고, Model 메서드를 선언한다.
  이 메서드는 "data"라는 키를 통해 "hello!"를 전달하며, return 값은 "hello.html"로 리턴하라는 의미다.
  */
  ~~~

- resources/templates/hello.html

  ~~~html
  <!DOCTYPE HTML>
  <html xmlns:th="http://www.thymeleaf.org">
  <head>
   <title>Hello</title>
   <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
  </head>
  <body>
  <p th:text="'안녕하세요. ' + ${data}" >안녕하세요. 손님</p>
  </body>
  </html>
  <!-- 태그 p에서 ${"data"}는 coctroller 모델에서 전달받은 키이며, 키에 대한 값을 리턴한다.-->
  ~~~

#### 동작 환경

![image-20210719171832558](C:\Users\yujin\AppData\Roaming\Typora\typora-user-images\image-20210719171832558.png)

> 웹에 url을 검색 ->  톰캣이 먼저 받고->  /hello과 매핑된 컨트롤러에 전달 -> 컨트롤러는 model의 data 키 값을 html에 리턴 ->  html에서 화면 처리한다.



# 1.4 빌드와 실행하기

[윈도우 버전]

서버 배포할 때 jar파일을 복사하고  자바 실행시키면 서버에서 동작됨

1. gradlew build  : 빌드 하기   
2. cd build/lib 
3. java -jar hello-spring-0.0.1-SNAPSHOT.jar  
4. 실행 확인
5. gradlew clean build   : 빌드 지우기

> cmd에서 리눅스 명령어 사용하기
>
> 1. ll을 사용하고 싶다.  -> doskey ll = dir