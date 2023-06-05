# index.js
---
### / 의 path로 접속했을 때 /login 으로 이동
이 기능에 대해서는 hint를 그대로 사용하여 /login으로만 이동하게끔 path를 지정해주었다.
```javascript
app.get("/", (req, res) =>
{
    // <Your codes here>
    res.redirect('/login');
});
```

### 로그인 상태일 때 /login path로 접속한다면 /profile로 이동

로그인이 되어있다면 session에 현재 로그인 되어있는 user의 정보가 있기 때문에 존재한다면(True 라면) /profile로 redirect를 해주었다.  

로그인이 되어있지 않다면 login-page.ejs를 렌더링 하도록 한다. 관련된 코드는 아래와 같다.

```javascript
if (req.session.user)
    {
        // <Your codes here>
       return res.redirect('/profile');
    }

    // Display the login page.
    // Hint: Use the render() function.

    // <Your codes here>
    res.render("login-page",{});
    return;
```

### /login 페이지에서 POST 요청이 들어온 경우 처리
우선 post 요청에 대한 처리를 해야하므로 다음 코드를 작성한다.
```javascript
app.post("/login",(req, res) =>
```
POST 로 받은 데이터를 userId: , password: 라는 새로운 변수로 받았다. 입력받은 userId를 토대로 testIds.js에 있는 user 정보와 비교하고 일치했을 때 그 user의 정보를 변수 user에 받았다. 따라서 아이디는 일치하지만 비밀번호가 틀렸을 때의 조건을 다음과 같이 작성할 수 있다.
```javascript
if(user.password !== password) {
    ...
}
```
  
  비밀번호를 잘못 입력했다면 경고창에 password failed를 띄우고 동시에 /login으로 다시 이동하도록 script를 다음과 같이 설정했다.
  ```javascript
  // Display a login failure pop-up and redirect to the "/login" page.
        res.writeHead(401, {"Content-Type": "text/html; charset=utf-8"});

        // <Your codes here> (For redirecting to the login page)
        res.write(`
        <script>
            window.onload = function() {
                alert("password failed!");
                window.location.href = "/login";
            };
        </script>`);

        res.end();
        return;
  ```

  입력된 아이디와 비밀번호 모두 일치한다면 session 에 로그인 한 사용자를 등록해줘야한다. 다음의 코드로 직접 session에 user를 등록해주었다.
  ```javascript 
  req.session.user = {
            userId: user.userId,
            password: user.password,
            userName: user.userName,
            classNumber: user.classNumber,
            photo: user.photo,
        };
        //It is quite long to save user info to session. So we have to guarantee the time to be saved.
        req.session.save(function(){
            res.redirect('/profile');
        })
  ```

  save 함수를 넣은 이유는 정말 높은 확률로 session에 user 를 등록하는 시간이 redirect보다 오래걸려 마치 로그인이 실패한 것 처럼 되기 때문이었다. 그래서 user 의 등록이 완료된 후 redirect 할 수 있도록 위의 req.session.save(...)를 작성했다.
    
### /logout 시 session 삭제
로그아웃을 하면 session에 있는 유저를 삭제해야하는데 이는 hint의 함수를 그대로 사용했다.
```javascript
    req.session.destroy();
    res.redirect('/login');
```

### /profile에서 GET 요청 받았을 때 처리
프로필에는 현재 session에 등록된 사용자의 정보를 가지고 렌더링을 해야하기 때문에 먼저 다음의 코드로 session의 user 정보를 가져온다.
```javascript
const user = req.session.user;
```

profile 페이지에서 필요한 사용자의 정보는 다음과 같이 구성했다.
```html
<!-- Your codes here -->
    <div class="profileImg">
        <img src= "<%=profileImg%>" /> 
    </div>
    <!-- the user information(name, student number) -->
    <!-- Hints: refer to the data of the testIds.js -->
    <div class="profile-info">
        <!-- Your codes here -->
        <div class="userName">
            NAME: <%=userName%>
        </div>
        <div class="studentId">
            STUDENT ID: <%=studentId%>
        </div>
        
    </div>
```
따라서 이름을 일치시켜 rendering에 필요한 데이터를 다음과 같이 구성했다.
```javascript
res.render("profile-page", {
        userName: user.userName,
        // <Your codes here>
        studentId: user.classNumber,
        profileImg: user.photo,
    });
```



