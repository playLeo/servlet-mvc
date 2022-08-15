# servlet-mvc
스프링 MVC 1편

## 서블릿
* 소켓 연결이나 HTTP 메세지 파싱, 멀티 스레드와 같은 반복적이고 어려운 작업을 대신 해준다.
```java
@WebServlet(name = "helloServlet", urlPatterns = "/hello") 
public class HelloServlet extends HttpServlet {
 @Override 
 protected void service(HttpServletRequest request, HttpServletResponse response){
 // 애플리케이션 로직 } 
 }
```
![서블릿 컨테이너](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcF89Gz%2FbtricDOaWSN%2FgUdbGrpu7T1sOF3OcquWj0%2Fimg.png)


### HttpServletRequset - HTTP 요청 메세지를 파싱한 결과를 객체로 반환해준다.

요청은 주로 3가지 방법을 사용한다.
1. GET - 쿼리 파라미터
2. POST - HTML FORM
    * content-type:application/x-www-form-urlencoded
    * 메세지 바디에 쿼리 파라미터 형식으로 전달된다.  
3. HTTP 메세지 바디
    * HTTP API에서 주로 사용, JSON,XML,TEXT
    * POST(HTML FORM형식이 아닌), PUT, PATCH
  
  
* GET - 쿼리 파라미터 조회

서버에서 쿼리 파라미터를 HttpServletRequest가 제공하는 메서드로 편하게 조회할 수 있다.
```java
String username = request.getParameter("username"); //단일 파라미터 조회
Enumeration<String> parameterNames = request.getParameterNames(); //파라미터 이름들모두 조회
Map<String, String[]> parameterMap = request.getParameterMap(); //파라미터를 Map으로 조회
String[] usernames = request.getParameterValues("username"); //복수 파라미터 조회
```

* POST - 쿼리 파리미터 조회 - GET 파라미터 조회와 같이 메서드로 조회가 가능하다.

데이터가 메세지 바디에서 오지만, 쿼리 파라미터 형식으로 오기 때문에 메서드로 조회 가능하다.

* HTTP 메세지 바디 조회
```java
@WebServlet(name = "requestBodyStringServlet", urlPatterns = "/request-bodystring")
public class RequestBodyStringServlet extends HttpServlet {

 @Override
 protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
 
 
 ServletInputStream inputStream = request.getInputStream();
 String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
 System.out.println("messageBody = " + messageBody);
 response.getWriter().write("ok");
 }}
```
