# IntelliJ 단축키 모음 (Window 버전)

+ ctrl + b (클래스) : 연관된 클래스 목록보기
+ alt + enter : 클래스에서 테스트 만들기
+ alt + insert : 메서드 테스트 만들기
+ ctrl + alt + m : DI 생성자 따로 바깥으로 리팩토링
+ ctrl + e : 클래스 히스토리 내역을 볼 수 있음
+ ctrl + alt + v : 반환값 자동 생성
+ ctrl + alt + shift + t : 리팩토링
+ shift + f6 : 한번에 같은 단어 변경
+ ctrl + shift + t : 자동 테스트 패키지, 클래스, 메서드 생성
+ shift + f10 : 이전에 실행한걸 그대로 실행
+ psvm : main 메소드 자동생성
+ ctrl + d : 라인복제
+ ctrl (or) alt + shift + ↑ / ↓ : 라인 이동 (구문 안에서만, 구문무시)
+ ctrl + alt + f10 : 마지막 실행 재실행
+ ctrl + alt + <- / -> : 이전 커서가 있던 화면으로 이동


## 개발 환경 
JAVA 11, IntelliJ, Gradle, Spring Boot 2.5.2

## 설정 
Gradle 통해 실행하는 것 보다 자바로 실행하는 것이 속도가 빠르다. 
File -> Setting
Build, Execution, Deployment -> Build Tools -> Gradle 
<img src="https://user-images.githubusercontent.com/81945553/126821863-d68d984a-6df0-46dd-9a56-09eca3527cb3.png"  width="700" height="400">


## 라이브러리 
- spring-boot-starter-web
    - spring-boot-starter-tomcat: 톰캣 (웹서버)
    - spring-webmvc: 스프링 웹 MVC

- spring-boot-starter-thymeleaf: 타임리프 템플릿 엔진(View)
- spring-boot-starter(공통): 스프링 부트 + 스프링 코어 + 로깅
    - spring-boot
        - spring-core
    - spring-boot-starter-logging
        - logback, slf4j
- spring-boot-starter-test
    - junit: 테스트 프레임워크
    - mockito: 목 라이브러리
    - assertj: 테스트 코드를 좀 더 편하게 작성하게 도와주는 라이브러리
    - spring-test: 스프링 통합 테스트 지원

## VIEW 
static/index.html을 올려두면 Welcome page 기능을 제공한다. 

## 동작 과정 

```java
@Controller
public class HelloController {
 @GetMapping("hello")
 public String hello(Model model) {
 model.addAttribute("data", "hello!!");
 return "hello";
 }
}
```

```html
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
```

![캡쳐1](https://user-images.githubusercontent.com/81945553/126821474-4e440e40-a628-4469-b044-41f79a614fd2.png)

