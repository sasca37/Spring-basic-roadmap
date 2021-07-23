### 목표

- **정적컨텐츠** : 파일 그냥 웹 브라우저에 전달, 서버 관여 x
- **MVC와 템플릿 엔진** : 서버의 관여, HTML 변환 후 전달
- **API** : 데이터포맷(JSON)으로 클라이언트에게 전달하는 방식

# 1. 정적컨텐츠

 resources/static/hello-static.html 생성

![image](https://user-images.githubusercontent.com/38436013/126434128-6fae20a7-ad63-417f-bbdb-b37a71b4794c.png)

> 웹 브라우저에서 위 url로 검색-> 내장 톰캣서버가 요청 받음 -> 서버가 요청을 스프링 컨테이너에게 넘김 -> 1)컨트롤러 없으므로, 2)html 파일을 찾음 -> html 파일을 웹에 반환



# 2. MVC와 템플릿 엔진

MVC : Model, View, Controller

#### Controller 

비지니스 로직 담당

~~~java
@Controller
public class HelloController {
 @GetMapping("hello-mvc")
 public String helloMvc(@RequestParam("name") String name, Model model) {
 model.addAttribute("name", name);
 return "hello-template";
 }
}
~~~

#### view 

화면 담당

~~~html
<html xmlns:th="http://www.thymeleaf.org">
<body>
<p th:text="'hello ' + ${name}">hello! empty</p>
</body></html>
~~~

#### 실행 화면 

![image](https://user-images.githubusercontent.com/38436013/126435825-786ea041-53dd-461f-b74c-456a184b6821.png)

![image](https://user-images.githubusercontent.com/38436013/126435999-7bf06354-e7cb-4258-89ae-ef10c3e43b57.png)

> 2.1과 다른 점은, viewResolver는 컨트롤러에서 리턴된 html을 찾아서 **템플릿 엔진 처리 후 변환된 html을 웹 브라우저에 리턴**한다.

# 3. API

#### @ResponseBody 문자 반환 : Controller 

뷰 리졸버를 사용하지 않는다.

~~~java
@GetMapping("hello-string")
    @ResponseBody // http의 body부분에 데이터를 넣어준다.
    public String helloStringg(@RequestParam("name") String name) {
        return "hello " + name;
    }
~~~

#### 실행 화면

![image](https://user-images.githubusercontent.com/38436013/126436645-b9584343-b532-43ba-881a-cb165e927a56.png)

`return "hello " + name;` 이 부분을 그대로 html에 띄운다. **템플릿 엔진을 거치지 않는다.**

#### @ResponseBody 객체 반환 : Controller 

@ResponseBody 를 사용하고, 객체를 반환하면 객체가 JSON으로 변환됨

~~~java
@GetMapping("hello-api")
    @ResponseBody
    public Hello helloApi(@RequestParam("name") String name) {
        Hello hello = new Hello();
        hello.setName(name);
        return hello; // 클래스 객체를 전달, 객체는 json 형태로 리턴함
    }
    static class Hello {
        private String name;
        public String getName() {
            return name;
        }
        public void setName(String name) {
            this.name = name;
        }
    }
~~~

 #### 실행 화면

![image](https://user-images.githubusercontent.com/38436013/126440204-e64110ad-6759-495f-a275-898392af9e08.png)

![image](https://user-images.githubusercontent.com/38436013/126440324-efa4ce69-d1be-4831-9059-6f90459cffeb.png)

