# Request

# 1. 概述

<aside>
💡 Request是请求对象，Response是响应对象，在servlet的service方法中作为输入参数使用

</aside>

- request:获取请求数据
    - 浏览器会发送HTTP请求到后台服务器[Tomcat]
    - HTTP的请求中会包含很多请求数据[请求行+请求头+请求体]
    - 后台服务器[Tomcat]会对HTTP请求中的数据进行解析并把解析结果存入到一个对象中
    - 所存入的对象即为request对象，所以我们可以从request对象中获取请求的相关参数
    - 获取到数据后就可以继续后续的业务，比如获取用户名和密码就可以实现登录操作的相关业务
- response:设置响应数据
    - 业务处理完后，后台就需要给前端返回业务处理的结果即响应数据
    - 把响应数据封装到response对象中
    - 后台服务器[Tomcat]会解析response对象,按照[响应行+响应头+响应体]格式拼接结果
    - 浏览器最终解析结果，把内容展示在浏览器给用户浏览
- 使用案例
    
    ```java
    @WebServlet("/demo3")
    public class ServletDemo3 extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            //使用request对象 获取请求数据
            String name = request.getParameter("name");//url?name=zhangsan
    
            //使用response对象 设置响应数据
            response.setHeader("content-type","text/html;charset=utf-8");
            response.getWriter().write("<h1>"+name+",欢迎您！</h1>"); //响应页面显示---zhangsan，欢迎您
        }
    
        @Override
        protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            System.out.println("Post...");
        }
    }
    ```
    
    启动成功后就可以通过浏览器来访问（`localhost:8080/request-demo/demo3**?name=zhangsan**`），根据传入参数的不同就可以在页面上展示不同的内容
    

# 2. ****Request对象****

### 2.1 ****Request继承体系****

- 继承体系如下图
    
    ![Untitled](Request%207b0df1dc9e274741b8774936c5c8f30b/Untitled.png)
    
- ServletRequest和HttpServletRequest是继承关系，并且两个都是**接口，接口是无法创建对象，这个时候就引发了下面这个问题:**
    
    ![Untitled](Request%207b0df1dc9e274741b8774936c5c8f30b/Untitled%201.png)
    
    - 事实上，Request继承体系中的`RequestFacade`类实现了HttpServletRequest接口，也间接实现ServletRequest接口
    - Servlet类中的service方法、doGet方法或者是doPost方法最终都是由Web服务器[Tomcat]来调用的，所以Tomcat提供了方法参数接口的具体实现类，并完成了对象的创建

### 2.2 ****Request获取请求数据****

**获取请求行数据**

请求行包含三块内容，分别是`请求方式`、`请求资源路径`、`HTTP协议及版本`

![Untitled](Request%207b0df1dc9e274741b8774936c5c8f30b/Untitled%202.png)

- 获取请求方式: `GET`
    - String getMethod()
- 获取虚拟目录(项目访问路径): `/request-demo`
    - String getContextPath()
- 获取URL(统一资源定位符): `http://localhost:8080/request-demo/req1`
    - StringBuffer getRequestURL()
- 获取URI(统一资源标识符): `/request-demo/req1`
    - String getRequestURI()
- 获取请求参数(GET方式): `username=zhangsan&password=123`
    - String getQueryString()

使用案例

```java
@WebServlet("/req1")
public class RequestDemo1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // String getMethod()：获取请求方式： GET
        String method = req.getMethod();
        System.out.println(method);//GET
        // String getContextPath()：获取虚拟目录(项目访问路径)：/request-demo
        String contextPath = req.getContextPath();
        System.out.println(contextPath);
        // StringBuffer getRequestURL(): 获取URL(统一资源定位符)：http://localhost:8080/request-demo/req1
        StringBuffer url = req.getRequestURL();
        System.out.println(url.toString());
        // String getRequestURI()：获取URI(统一资源标识符)： /request-demo/req1
        String uri = req.getRequestURI();
        System.out.println(uri);
        // String getQueryString()：获取请求参数（GET方式）： username=zhangsan
        String queryString = req.getQueryString();
        System.out.println(queryString);
    }
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    }
}
```

- 测试：启动服务器，访问`http://localhost:8080/request-demo/req1?username=zhangsan&passwrod=123`

****获取请求头数据****

对于请求头的数据，格式为`key: value`如下:

![Untitled](Request%207b0df1dc9e274741b8774936c5c8f30b/Untitled%203.png)

- 根据请求头名称获取对应值：
    - String getHeader(String name)
- 使用案例
    
    ```java
    @WebServlet("/req1")
    public class RequestDemo1 extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            //获取请求头: user-agent: 浏览器的版本信息
            String agent = req.getHeader("user-agent");
    		System.out.println(agent);
        }
        @Override
        protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        }
    }
    ```
    
    - 测试：启动服务器后，访问`http://localhost:8080/request-demo/req1?username=zhangsan&passwrod=123`

****获取请求体数据****

浏览器在发送GET请求的时候是没有请求体的，所以需要把请求方式变更为POST，请求体中的数据格式如下:

![Untitled](Request%207b0df1dc9e274741b8774936c5c8f30b/Untitled%204.png)

- Request对象提供了如下两种方式来获取请全体数据
    - 获取字节输入流，如果前端发送的是字节数据，比如传递的是文件数据，则使用该方法
        
        `ServletInputStream getInputStream()`
        
    - 获取字符输入流，如果前端发送的是纯文本数据，则使用该方法
        
        `BufferedReader getReader()`
        
- 使用案例
    - 在项目的webapp目录下添加一个html页面，名称为：`req.html`
        
        ```html
        <!DOCTYPE html>
        <html lang="en">
        <head>
            <meta charset="UTF-8">
            <title>Title</title>
        </head>
        <body>
        <!-- 
            action:form表单提交的请求地址
            method:请求方式，指定为post
        -->
        <form action="/request-demo/req1" method="post">
            <input type="text" name="username">
            <input type="password" name="password">
            <input type="submit">
        </form>
        </body>
        </html>
        ```
        
    - 在Servlet的doPost方法中获取数据，因为目前前端传递的是纯文本数据，所以我们采用getReader()方法来获取
        
        ```java
        @WebServlet("/req1")
        public class RequestDemo1 extends HttpServlet {
            @Override
            protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            }
            @Override
            protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
                 //获取post 请求体：请求参数
                //1. 获取字符输入流
                BufferedReader br = req.getReader();
                //2. 读取数据
                String line = br.readLine();
                System.out.println(line);
            }
        }
        ```
        
        <aside>
        💡 BufferedReader流是通过request对象来获取的，当请求完成后request对象就会被销毁，request对象被销毁后，BufferedReader流就会自动关闭，所以此处就不需要手动关闭流了
        
        </aside>
        
    - 测试：启动服务器，通过浏览器访问`http://localhost:8080/request-demo/req.html`

****获取请求参数的通用方式****

- GET请求，请求参数在请求行中；POST请求，请求参数一般在请求体中
- 请求参数的获取
    - GET方式：`String getQueryString()`
    - POST方式：`BufferedReader getReader();`
- 统一获取参数方式
    - 方案一
        
        ```java
        @WebServlet("/req1")
        public class RequestDemo1 extends HttpServlet {
            @Override
            protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
                //获取请求方式
                String method = req.getMethod();
                //获取请求参数
                String params = "";
                if("GET".equals(method)){
                    params = req.getQueryString();
                }else if("POST".equals(method)){
                    BufferedReader reader = req.getReader();
                    params = reader.readLine();
                }
                //将请求参数进行打印控制台
                System.out.println(params);
              
            }
            @Override
            protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
                this.doGet(req,resp);
            }
        }
        ```
        
        这种方案以后每个Servlet都需要这样写代码，实现起来比较麻烦，这种方案我们不采用
        
    - 方案二
        - request对象已经将上述获取请求参数的方法进行了封装
        - request对象为我们提供了如下方法:
            - 获取所有参数Map集合：`Map<String,String[]> getParameterMap()`
            - 根据名称获取参数值(数组)：`String[] getParameterValues(String name)`
            - 根据名称获取参数值(单个值)：`String getParameter(String name)`
        - 案例
            - 修改req.html页面
                
                ```html
                <!DOCTYPE html>
                <html lang="en">
                <head>
                    <meta charset="UTF-8">
                    <title>Title</title>
                </head>
                <body>
                <form action="/request-demo/req2" method="get">
                    <input type="text" name="username"><br>
                    <input type="password" name="password"><br>
                    <input type="checkbox" name="hobby" value="1"> 游泳
                    <input type="checkbox" name="hobby" value="2"> 爬山 <br>
                    <input type="submit">
                
                </form>
                </body>
                </html>
                ```
                
            - 在Servlet代码中获取页面传递GET请求的参数值
                
                ```java
                @WebServlet("/req2")
                public class RequestDemo2 extends HttpServlet {
                    @Override
                    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
                        //GET请求逻辑
                        System.out.println("get....");
                        **//1. 获取所有参数的Map集合**
                        Map<String, String[]> map = req.getParameterMap();
                        for (String key : map.keySet()) {
                            // username:zhangsan lisi
                            System.out.print(key+":");
                            //获取值
                            String[] values = map.get(key);
                            for (String value : values) {
                                System.out.print(value + " ");
                            }
                            System.out.println();
                        }
                				**//2. 获取GET请求参数中的爱好，结果是数组值**
                				System.out.println("------------");
                        String[] hobbies = req.getParameterValues("hobby");
                        for (String hobby : hobbies) {
                            System.out.println(hobby);
                        }
                				**//3. 获取GET请求参数中的用户名和密码，结果是单个值**
                				String username = req.getParameter("username");
                        String password = req.getParameter("password");
                        System.out.println(username);
                        System.out.println(password);
                    }
                
                    @Override
                    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
                			this.doGet(req,resp);
                    }
                }
                ```
                
                <aside>
                💡 此时，即使表单页面用post方法请求，依然不用改变代码
                
                </aside>
                

### 2.3 ****Request请求转发****

- 请求转发(forward)：一种在服务器内部的资源跳转方式
    
    ![Untitled](Request%207b0df1dc9e274741b8774936c5c8f30b/Untitled%205.png)
    
    - 浏览器发送请求给服务器，服务器中对应的资源A接收到请求
    - 资源A处理完请求后将请求发给资源B
    - 资源B处理完后将结果响应给浏览器
    - 请求从资源A到资源B的过程就叫请求转发
- 请求转发的实现方式：`req.getRequestDispatcher("资源B路径").forward(req,resp);`
- 请求转发资源间共享数据:使用Request对象
    - 存储数据到request域[范围,数据是存储在request对象]中
        
        `void setAttribute(String name,Object o);`
        
    - 根据key获取值
        
        `Object getAttribute(String name);`
        
    - 根据key删除该键值对
        
        `void removeAttribute(String name);`
        
- 案例
    - 创建RequestDemo5类
        
        ```java
        @WebServlet("/req5")
        public class RequestDemo5 extends HttpServlet {
            @Override
            protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                System.out.println("demo5...");
                //存储数据
                request.setAttribute("msg","hello");
                //请求转发
                request.getRequestDispatcher("/req6").forward(request,response);
        
            }
        
            @Override
            protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                this.doGet(request, response);
            }
        }
        ```
        
    - 创建RequestDemo6类
        
        ```java
        @WebServlet("/req6")
        public class RequestDemo6 extends HttpServlet {
            @Override
            protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                System.out.println("demo6...");
                //获取数据
                Object msg = request.getAttribute("msg");
                System.out.println(msg);
        
            }
        
            @Override
            protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                this.doGet(request, response);
            }
        }
        ```
        
        测试：访问`http://localhost:8080/request-demo/req5`
        
- 请求转发的特点
    - 浏览器地址栏路径不发生变化：虽然后台从`/req5`转发到`/req6`,但是浏览器的地址一直是`/req5`,未发生变化
    - 只能转发到当前服务器的内部资源：不能从一个服务器通过转发访问另一台服务器
    - 一次请求，可以在转发资源间使用request共享数据（虽然后台从`/req5`转发到`/req6`，但是这个只有一次请求）