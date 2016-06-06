# Web Applicaties IV - Samenvatting

## Servlets

### Servlets First Example

#### Servlets/HelloServlet.java

```java
@Override // Example get method
protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        String firstname = request.getParameter("firstname");
        // ...
    }
@Override
protected void doPost(HttpServletRequest request, HttpServletResponse response)
                throws ServletException, IOException {
        processRequest(request, response);
      }
```

#### web/welcomeForm.html

```html
<!-- Form to post / get -->
<form action = "welcome1" method ="post">
    <p><label>Click the button to invoke the servlet</label>
    <input type ="text" name="firstname"/>
    <input type="submit" value="Get HTNL Document"/>
</form>
```

### Redirect Servlet

#### web/redirectForm.html

```html
<p>
   <a href = "redirect?page=oracle">
      ORACLE</a><br>
   <a href = "redirect?page=welcome">
      Welcome servlet</a>
</p>
```

#### Servlet/RedirectServlet.java

```java
@Override
protected void doGet(HttpServletRequest request,
        HttpServletResponse response)
        throws ServletException, IOException {
    String location = request.getParameter("page");
    if (location != null) {
        if (location.equals("oracle")) {
            response.sendRedirect("http://www.oracle.com/us/index.htm");
        } else {
            if (location.equals("welcome")) {
                response.sendRedirect("welcome1");
            }
        }
    }
}
```

--------------------------------------------------------------------------------

## Spring MVC

### Structuur

### Hello Application

### web/WEB-INF/jsp/helloView.jsp

```xml
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>Hello</title>
    </head>
    <body>
        <h1>${helloMessage}</h1>
    </body>
</html>
```

### web/WEB-INF/jsp/nameForm.jsp

```xml
<%@page contentType="text/html" pageEncoding="UTF-8"%>
<%@taglib prefix = "form" uri="http://www.springframework.org/tags/form"%>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>Enter your name</title>
    </head>
    <body>
        <h1>Enter your name</h1>
        <form:form method="POST" action="hello.htm" modelAttribute="name">
            Name:
            <form:input path="value" size="15"/>
            <input type="submit" value="OK"/>
        </form:form>
    </body>
</html>
```

### web/WEB-INF/web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="3.1" xmlns="http://xmlns.jcp.org/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd">
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/applicationContext.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>2</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>*.htm</url-pattern>
    </servlet-mapping>
    <session-config>
        <session-timeout>
            30
        </session-timeout>
    </session-config>
    <welcome-file-list>
        <welcome-file>redirect.jsp</welcome-file>
    </welcome-file-list>
</web-app>
```

### web/WEB-INF/applicationContext.xml

```xml
<?xml version='1.0' encoding='UTF-8' ?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop-4.0.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx-4.0.xsd">
    <bean id="helloService" class="domain.HelloServiceImpl"/>
</beans>
```

### java/domain/HelloService.java

```java
public interface HelloService {
    public String sayHello(String name);
}
```

### java/domain/HelloServiceImpl.java

```java
public class HelloServiceImpl implements HelloService{
    @Override
    public String sayHello(String name) {
        return String.format("Hello %s", name != null?name:"");
    }
}
```

### java/controller/HelloController.java

```java
@Controller
public class HelloController {
    @Autowired
    private HelloService helloService;

    @RequestMapping(value = {"hello"}, method = RequestMethod.GET)
    public String showHomePage(Model model){
        model.addAttribute("name", new Name());
        return "nameForm";
    }

    @RequestMapping(value = {"hello"}, method = RequestMethod.POST)
    public String onSubmit(@ModelAttribute Name name, Model model){
        model.addAttribute("helloMessage", helloService.sayHello(name.getValue()));
        return "helloView";
    }
}
```

### test/controller/HelloControllerMockTest

```java
public class HelloControllerMockTest {
    private HelloController controller;
    private MockMvc mockMvc;

    @Mock
    HelloService mock;

    @Before
    public void setUp() {
        MockitoAnnotations.initMocks(this);
        controller = new HelloController();
        mockMvc = standaloneSetup(controller).build();
    }

    @Test
    public void testHelloPost() throws Exception{
        String expResult ="Hello testMock";
        Mockito.when(mock.sayHello("test")).thenReturn(expResult);
        ReflectionTestUtils.setField(controller, "helloService", mock);

        mockMvc.perform(post("/hello").param("value", "test"))
                .andExpect(view().name("helloView"))
                .andExpect(model().attributeExists("helloMessage"))
                .andExpect(model().attribute("helloMessage", expResult));
    }
}
```

### test/controller/HelloControllerTest.java

```java
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration({
  "file:web/WEB-INF/applicationContext.xml",
  "file:web/WEB-INF/dispatcher-servlet.xml"
})
public class HelloControllerTest {
    @Autowired
    private WebApplicationContext wac;
    private MockMvc mockMvc;

    @Before
    public void setUp() {
        mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
    }

    @Test
    public void testHelloGet() throws Exception{
        mockMvc.perform(get("/hello"))
                .andExpect(view().name("nameForm"))
                .andExpect(model().attributeExists("name"));
    }

    @Test
    public void testHelloPost() throws Exception{
        mockMvc.perform(post("/hello")
                .param("value", "test"))
                .andExpect(status().isOk())
                .andExpect(view().name("helloView"))
                .andExpect(model().attributeExists("helloMessage"))
                .andExpect(model().attribute("helloMessage", "Hello test"));
    }
}
```
