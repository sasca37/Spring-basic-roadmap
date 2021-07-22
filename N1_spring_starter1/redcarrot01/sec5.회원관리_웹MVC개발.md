### 목표

- 회원 웹 기능 - 홈 화면 추가
- 회원 웹 기능 - 등록
- 회원 웹 기능 - 조회

# 1. 회원 웹 기능 - 홈 화면 추가

#### 홈 컨트롤러 추가

~~~java
package hello.hellospring.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {

    @GetMapping("/")
    public String home(){
        return "home";
    }
}

~~~



#### 회원 관리형 홈

~~~html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<div class="container">
    <div>
        <h1>Hello Spring</h1>
        <p>회원 기능</p>
        <p>
            <a href="/members/new">회원 가입</a>
            <a href="/members">회원 목록</a>
        </p>
    </div></div> <!-- /container -->
</body>
</html>
~~~

> 컨트롤러가 정적 파일보다 우선순위가 높다. 



# 2. 회원 웹 기능 - 등록

### 회원 등록 폼 개발

#### 회원 등록 폼 컨트롤러

~~~java
@Controller
public class MemberController {
    
    private final MemberService memberService;

   
    @Autowired
    public MemberController(MemberService memberService){

        this.memberService = memberService;
    }

    @GetMapping(value="members/new")
    public String CreateForm() {
        return "members/createMemberForm";
    }
~~~



#### 회원 등록 폼 HTML L ( resources/templates/members/createMemberForm )

~~~html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org"><body>
<div class="container">
    <form action="/members/new" method="post">
        <div class="form-group">
            <label for="name">이름</label>
            <input type="text" id="name" name="name" placeholder="이름을
입력하세요">
        </div>
        <button type="submit">등록</button>
    </form>
</div> <!-- /container -->
</body>
</html>
~~~



### 회원 등록 컨트롤러

#### 웹 등록 화면에서 데이터를 전달 받을 폼 객체

~~~java
package hello.hellospring.controller;
public class MemberForm {
    private String name;

    public String getName(){
        return name;
    }
    public void setName(String name){
        this.name = name;
    }
}
~~~



#### 회원 컨트롤러에서 회원을 실제 등록하는 기능

~~~~java
@PostMapping(value="members/new")
    public String create(MemberForm form){
        Member member = new Member();
        member.setName(form.getName());
        memberService.join(member);
        return "redirect:/";
}
~~~~



# 3. 회원 웹 기능 - 조회

#### 회원 컨트롤러에서 조회 기능

~~~java
public String list(Model model){
        List<Member> members = memberService.findMembers();
        model.addAttribute("members", members);
        return "members/membersList";
    }
~~~



#### 회원 리스트 HTML

~~~html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<div class="container">
    <div>
        <table>
            <thead> <tr>
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
~~~

> **html 보기** ->  ${members} 은 모델 안의 값을 가져오는 것으로, members는 위 컨트롤러 코드에서 확인할 수 있다. 
>
> **컨트롤러 보기** -> 컨트롤러에서 members는 members라는 키의 members 값으로, 이것은 리스트로 모든 회원을 다 가져온다.
>
> 다시 **html 보기** -> th: each는 반복문 같은 것으로, 루프를 돌면서 members에서 회원 정보를 하나씩 꺼내온다. 여기서 id와 이름을 getId(), getName()으로 가져와 출력한다.

