# servlet-mvc
스프링 MVC 1편

## Web Server vs WAS

* Web Server - 정적 리소스 처리
* WAS - 정적 리소스 처리

크게보면 이렇지만 Web Server도 어플리케이션 로직을 처리하면서 경계가 애매해졌다. WAS가 어플리케이션 로직 처리에 특화 되었다고 생각하자.

WAS로만 웹구성시 WAS가 너무 많은 일을 한다.(정적 리소스 처리, 동적 리소스 처리 등등) -> WAS앞에 Web Server을 두어 정적 리소스는 웹서버가 처리하게 구성한다.

이렇게 구성하면 WAS는 어플리케이션 로직만을 수행하며 과부하를 줄이고, 상대적으로 죽을 확률이 높은 WAS가 죽었을 때, 화면이 아예 노출이 안되는 불상사를 막을 수 있다.

## 서블릿
* 소켓 연결이나 HTTP 메세지 파싱, 멀티 스레드와 같은 반복적이고 어려운 작업을 대신 해준다.
* 동시 요청을 위한 멀티쓰레드 처리 지원

멀티 쓰레드 풀을 이용해 관리.

1. @ServletComponentScan
2. @WebServlet 사용과 service 메서드 정의
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

content-type: application/json

보통 객체형식으로 전송한다.

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

cookie(response);

response.sendRedirect("/basic/hello-form.html");

//[message body]
PrintWriter writer = response.getWriter();
writer.println("ok");

private void cookie(HttpServletResponse response){
   //Set-Cookie: myCooke=good; Max-Age=600;
   //response.setHeader("Set-Cooke", "myCooke=good; Max-Age=600");
   Cookie cookie = new Cookie("myCookie", "good");
   cookie.setMaxAge(600);
   response.addCookie(cookie);
}
```

### Servlet 만 사용

https://github.com/playLeo/servlet-mvc/tree/main/src/main/java/hello/servlet/web/servlet

### JSP + Servlet 사용 + view 분리

* Http messege body에 html을 직접 전달해서 동적 페이지를 만드는 힘든작업을 JSP 도입으로 해결하고, MVC 모델을 도입해 비지니스 로직과 뷰를 분리해 유지보수성을 높인다. 

https://github.com/playLeo/servlet-mvc/tree/main/src/main/java/hello/servlet/web/servletmvc

* JSP 파일이 분리되어 화면을 구성할 때, 필요한 데이터를 받아야 한다. -> JSP는 HttpServletRequest를 Model로 사용할 수 있다.
* 비지니스 로직에서 View에 필요한 데이터를 request.setAttribute로 저장
* View 로직에서 필요한 데이터를 request.getAttribute로 꺼내 사용할 수 있다. -> JSP 문법 ${}로 편하게 사용 가능
* request.getRequestDisptcher(viewPath).forward(request, response)  - 서블릿이나 JSP로 이동할 수 있는 기능(버서 내부에서 다시 호출)
* /WEB_INF 경로 안에 있는 JSP는 외부에서 직접 호출 불가능하고, 컨트롤러를 통해서만 호출 가능하다.


### Front Controller 도입 - V1

* 스프링 MVC는 DispatcherServlet이 Front Controller 패턴으로 구현되어 있다.
* 프론트 컨트롤러를 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아도 된다.

https://github.com/playLeo/servlet-mvc/tree/main/src/main/java/hello/servlet/web/frontcontroller/v1

![Front Controller IMG](https://velog.velcdn.com/images%2Fbins1225%2Fpost%2F16bbc561-cb66-4265-b050-2ef90aba07d8%2Fimage.png)

### Front Controller를 통한 포워드 작업 일괄처리 - V2

https://github.com/playLeo/servlet-mvc/tree/main/src/main/java/hello/servlet/web/frontcontroller/v2

![Front Controller view 분리 IMG](https://velog.velcdn.com/images%2Fbins1225%2Fpost%2F11ece184-dcf7-45d9-a540-22623855d813%2Fimage.png)


### Model 객체를 만들어 서블릿 종속성 제거, 뷰 이름 중복 제거 - V3

https://github.com/playLeo/servlet-mvc/tree/main/src/main/java/hello/servlet/web/frontcontroller/v3

![IMG](https://velog.velcdn.com/images%2Fbins1225%2Fpost%2F726201c3-e08b-48d3-9510-902454a88237%2Fimage.png)

### 객체반환 -> 문자열 반환 - V4

https://github.com/playLeo/servlet-mvc/tree/main/src/main/java/hello/servlet/web/frontcontroller/v4

![IMG](https://velog.velcdn.com/images%2Fbins1225%2Fpost%2Fc8908627-6f42-4744-9703-1701ed308975%2Fimage.png)


### 어댑터 패턴 적용 - V5

https://github.com/playLeo/servlet-mvc/tree/main/src/main/java/hello/servlet/web/frontcontroller/v5

![IMG](https://velog.velcdn.com/images%2Fbins1225%2Fpost%2Fceb796d6-5e20-47eb-92cd-ce5ce867ee7c%2Fimage.png)











