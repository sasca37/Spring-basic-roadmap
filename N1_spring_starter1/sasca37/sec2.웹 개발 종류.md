# 웹 개발 종류 
## 정적 컨텐츠
- 단순 파일 그대로를 전달
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
Hello
<a href="/hello">hello</a>
</body>
</html>
``` 
![sec2캡처1](https://user-images.githubusercontent.com/81945553/126855430-62df584c-e614-412e-9e53-ec021f3f0642.png)

## MVC와 템플릿 엔진(Jsp, Thymeleaf 등) 
- HTML에 템플릿 엔진을 통해 동적 페이지 구성

![sec2캡처1](https://user-images.githubusercontent.com/81945553/126855449-1b9ec740-0189-49e4-b6b5-ab4b6a1743e0.png)

## API
- JSON 데이터만 클라이언트에 전달하여 구성 (서버 간 통신 등 ), @ResponsBody 사용하여 객체를 반환하면 JSON으로 변환됨 

```Java
@GetMapping("hello-mvc")
    public String helloMvc(@RequestParam("name") String name, Model model) {
        model.addAttribute("name", name);
        return "hello-template";
    }
    //ResponseBody 
    //http body 부분에 직접 넣어주겠다를 명시하는 애노테이션
    // 문자면 String 값, 객체면 제이슨 값 ( viewResolver 대신에 HttpMessageConverter가 동작)
    // 문자 처리 : StringHttpMessageConverter
    // 객체 처리 : MappingJackson2HttpMessageConverter

    @ResponseBody
    @GetMapping("hello-string")
    public String helloString(@RequestParam("name") String name) {
        return "hello" +name;
    }

    //json 방식 (객체를 만들어서 날렸을 때 json으로 출력됨 )
    @GetMapping("hello-api")
    @ResponseBody
    public Hello helloApi(@RequestParam("name") String name ){
        Hello hello = new Hello(); //c shift enter 시 자동완성
        hello.setName(name);
        return hello;
    }
``` 

![sec2캡쳐3](https://user-images.githubusercontent.com/81945553/126855575-a68bc00e-6f3d-45a4-9ee2-437d1ec8a289.png)
