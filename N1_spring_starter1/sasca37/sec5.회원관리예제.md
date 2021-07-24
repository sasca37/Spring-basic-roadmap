## VIEW 우선순위 
- Request 요청이 오면 컨트롤러에 해당 정보가 있는지 먼저 찾고 없으면 static/index.html 파일을 찾는다. 
- localhost:8080 요청이 왔을 경우 Controller에 매핑이 되어 있으므로 template/home.html을 찾아간다. (타임리프 기준)

```Java
@Controller
public class HomeController {
    @GetMapping("/")
    public String home() {
        return "home";
    }
}
```


## VIEW FORM 태그 
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<div class="container">
  <form action="/members/new" method="post">
    <div class="form-group">
      <label for="name">이름</label>
      <input type="text" id="name" name="name" placeholder="이름을 입력하세요">
    </div>
    <button type="submit">등록</button>
  </form>
</div> <!-- /container -->
</body>
</html>
```
- form 태그 : 값을 전송하기 위한 태그 , method="post" 포스트 방식으로 전달 
- <input> type="text" : 텍스트 입력, name="name" 서버에 전달할 키값("name"), placeholder : 힌트 
- post 방식 : 데이터를 form 에 담아서 등록할 때(Insert 개념) 보통 사용 
- get 방식 : 데이터를 조회할 때 주로 사용 (Select 개념)
- URL이 같아도 Controller에서 GET, POST 매핑으로 다르게 처리할 수 있다. 


- Controller, MemberForm 코드 
```Java
@GetMapping("/members/new")
    public String createForm() {
        return "members/createMemberForm";
    }

    @PostMapping("/members/new")
    public String create(MemberForm form) {
        Member member = new Member();
        member.setName(form.getName());

        System.out.println("member : "+member.getName());

        memberService.join(member);

        return "redirect:/";
    }

    @GetMapping("/members")
    public String list(Model model) {
       List<Member> members = memberService.findMember();
       model.addAttribute("members", members);
       return "members/memberList";
    }
```
```Java
    public class MemberForm {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
- MemberForm 클래스에서 설정한 name을 @PostMapping에 매개변수로 담으면 form 태그에서 전달한 키값 : "name" 이 MemberForm 클래스의 name으로 매핑(Setter를 통해)된다. 

## 타임리프 문법 

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<div class="container">
  <div>
    <table>
      <thead>
      <tr>
        <th>#</th>
        <th>이름</th>
      </tr>
      </thead>
      <tbody>
      <tr th:each="member : ${members}">
        <td th:text="${member.id}"></td>
        <td th:text="${member.name}"></td>
      </tr>
      </tbody>
    </table>
  </div>
</div> <!-- /container -->
</body>
</html>
```
- Controller에서 model 로 전달한 값을 ${members}로 받아와서 th:each로 반복문을 실행한다. 
- <tr th:each="member : ${members}"> -> member : 사용할 변수명, ${members} : Controller에서 전달 받은 값
- ${member.id} : Getter 를 이용하여 값을 가져온다.
