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
 
 //inputStream 은 byte 코드를 반환
 ServletInputStream inputStream = request.getInputStream();
 String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
 System.out.println("messageBody = " + messageBody);
 response.getWriter().write("ok");
 }}
```

**JSON 타입의 HTTP 메세지 바디 조회**

JSON타입으로 온 데이터를 파싱해서 사용할 수 있다.
* Jackson, Gson 같은 JSON 라이브러리를 추가해서 사용해야한다. (스프링 부트는 Jackson 제공)

```java
@WebServlet(name = "requestBodyJsonServlet", urlPatterns = "/request-bodyjson")
public class RequestBodyJsonServlet extends HttpServlet {

 //Jackson라이브러리 objectMapper 객체 사용
   private ObjectMapper objectMapper = new ObjectMapper();

   @Override
   protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
   
     //Json형식 파싱 안하고 사용
     ServletInputStream inputStream = request.getInputStream();
     String messageBody = StreamUtils.copyToString(inputStream,StandardCharsets.UTF_8);
     System.out.println("messageBody = " + messageBody);
     
     //프로퍼티 설정되어있는 HelloData
     HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);
     System.out.println("helloData.username = " + helloData.getUsername());
     System.out.println("helloData.age = " + helloData.getAge());
     response.getWriter().write("ok");
  }
}
```

**참고**
객체를 JSON 형식으로 파싱하려면 objectMapper.writeValueAsString()를 사용하면 된다.

### HttpServletResponse - HTTP 응답 메세지를 파싱한 결과를 객체로 반환한다.
```java
//[status-line]
response.setStatus(HttpServletResponse.SC_OK); //200
//[response-headers]
response.setHeader("Content-Type", "text/plain;charset=utf-8");
//response.setContentType("text/plain");
//response.setCharacterEncodin("utf-8");
response.setHeader("Cache-Control", "no-cache, no-store, mustrevalidate");
response.setHeader("Pragma", "no-cache");
response.setHeader("my-header","hello");
response.sendRedirect("/basic/hello-form.html");

//[message body]
PrintWriter writer = response.getWriter();
writer.println("ok");
```

### Servlet 만을 사용해서 html 정보를 넘기기 - messege body에 html을 하나하나 넣어줘야 한다.

```java
@WebServlet(name = "responseHtmlServlet", urlPatterns = "/response-html")
public class ResponseHtmlServlet extends HttpServlet {
   @Override
   protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

     response.setContentType("text/html");
     response.setCharacterEncoding("utf-8");

     PrintWriter writer = response.getWriter();
     writer.println("<html>");
     writer.println("<body>");
     writer.println(" <div>안녕?</div>");
     writer.println("</body>");
     writer.println("</html>");
   }
}
```
