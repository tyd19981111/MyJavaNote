# Session

# 1. ****Session的基本使用****

### 1.1 概念

Session：服务端会话跟踪技术：将数据保存到服务端

- Session是存储在服务端而Cookie是存储在客户端
- 存储在客户端的数据容易被窃取和截获，存在很多不安全的因素
- 存储在服务端的数据相比于客户端来说就更安全

### 1.2 **Session的工作流程**

![Untitled](Session%2092dde0d2515f4e3abb19f4cd929db332/Untitled.png)

- 在服务端的**AServlet**获取一个Session对象，把数据存入其中；在服务端的**BServlet**获取到相同的Session对象，从中取出数据，就可以实现一次会话中多次请求之间的数据共享了

<aside>
❓ 问题：如何保证AServlet和BServlet使用的是同一个Session对象(在原理分析会讲解)?

</aside>

### 1.3 **Session的基本使用**

在JavaEE中提供了HttpSession接口，来实现一次会话的多次请求之间数据共享功能

- 获取Session对象,使用的是request对象：
    
    `HttpSession session = request.getSession();`
    
- Session对象提供的功能:
    - 存储数据到 session 域中
        
        `void setAttribute(String name, Object o)`
        
    - 根据 key，获取值
        
        `Object getAttribute(String name)`
        
    - 根据 key，删除该键值对
        
        `void removeAttribute(String name)`
        
- 案例演示
    - 创建名为SessionDemo1的Servlet类（获取Session对象，存储数据）
        
        ```java
        @WebServlet("/demo1")
        public class SessionDemo1 extends HttpServlet {
            @Override
            protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            	//存储到Session中
                //1. 获取Session对象
                HttpSession session = request.getSession();
                //2. 存储数据
                session.setAttribute("username","zs");
            }
        
            @Override
            protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                this.doGet(request, response);
            }
        }
        ```
        
    - 创建名为SessionDemo2的Servlet类（获取Session对象，获取数据）
        
        ```java
        @WebServlet("/demo2")
        public class SessionDemo2 extends HttpServlet {
            @Override
            protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                //获取数据，从session中
                //1. 获取Session对象
                HttpSession session = request.getSession();
                //2. 获取数据
                Object username = session.getAttribute("username");
                System.out.println(username);
            }
        
            @Override
            protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                this.doGet(request, response);
            }
        }
        ```
        
    - 启动测试
        - 先访问`http://localhost:8080/cookie-demo/demo1`,将数据存入Session
        - 在访问`http://localhost:8080/cookie-demo/demo2`,从Session中获取数据
        
        <aside>
        💡 通过案例的效果，能看到Session是能够在一次会话中两次请求之间共享数据
        
        </aside>
        

# 2. Session原理分析

### 2.1 Q&A

- Session要想实现一次会话多次请求之间的数据共享，就必须要保证多次请求获取Session的对象是同一个。上一个案例中的两个Servlet，他们的Session对象是同一对象吗？
    - 验证：分别打印在控制台查看`System.out.println(session);`
    - 访问demo1，demo2后发现确实是同一对象
- 问题又来了，如果新开一个浏览器，访问demo1或者demo2,打印在控制台的Session还是同一个对象么?
    - 如果是不同浏览器或者重新打开浏览器后，打印的Session就不一样了
- Session是如何保证在一次会话中获取的Session对象是同一个呢?（见原理分析）

### 2.2 原理分析

<aside>
💡 Session是基于Cookie实现的

</aside>

![Untitled](Session%2092dde0d2515f4e3abb19f4cd929db332/Untitled%201.png)

- demo1在第一次获取session对象的时候，session对象会有一个唯一的标识，假如是`id:10`
- demo1在session中存入其他数据并处理完成所有业务后，需要通过Tomcat服务器响应结果给浏览器
- Tomcat服务器发现业务处理中使用了session对象，就会把session的唯一标识`id:10`当做一个cookie，添加`Set-Cookie:JESSIONID=10`到响应头中，并响应给浏览器
- 浏览器接收到响应结果后，会把响应头中的coookie数据存储到浏览器的内存中
- 浏览器在同一会话中访问demo2的时候，会把cookie中的数据按照`cookie: JESSIONID=10`的格式添加到请求头中并发送给服务器Tomcat
- demo2获取到请求后，从请求头中就读取cookie中的JSESSIONID值为10，然后就会到服务器内存中寻找`id:10`的session对象，如果找到了，就直接返回该对象，如果没有则新创建一个session对象
- 关闭打开浏览器后，因为浏览器的cookie已被销毁，所以就没有JESSIONID的数据，服务端获取到的session就是一个全新的session对象

<aside>
➡️ 验证：在上一案例中，访问demo1，demo2时打开开发者工具

</aside>

# 3. ****Session钝化与活化****

### 3.1 Q&A

- 服务器重启后，Session中的数据是否还在?
    - 分析一个案例：
    
    <aside>
    💡 用户把需要购买的商品添加到购物车，假设把数据存入Session对象中；用户正要付钱的时候接到一个电话，付钱的动作就搁浅了；此时购物网站因为某些原因需要重启，重启后session数据被销毁，购物车中的商品信息也就会随之而消失；用户想再次发起支付，就会出为问题
    
    </aside>
    
    - 所以说对于session的数据，我们应该做到就算服务器重启了，也应该能把数据保存下来才对

### 3.2 Session钝化与活化

- 钝化：在服务器正常关闭后，Tomcat会自动将Session数据写入硬盘的文件中
    - 钝化的数据路径为:`项目目录\target\tomcat\work\Tomcat\localhost\项目名称\SESSIONS.ser`
- 活化：再次启动服务器后，从文件中加载数据到Session中
    - 数据加载到Session中后，路径中的`SESSIONS.ser`文件会被删除掉

<aside>
💡 验证：使用之前的案例验证，关闭Tomcat后重启，查看session是否丢失

</aside>

### 3.3 ****Session销毁****

- 默认情况下，无操作，30分钟自动销毁
    - 对于这个失效时间，是可以通过配置进行修改的，在项目的web.xml中配置
        
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
                 version="3.1">
        
            <session-config>
                <session-timeout>100</session-timeout>
            </session-config>
        </web-app>
        ```
        
- 调用Session对象的invalidate()进行销毁
    - 在SessionDemo2类中添加session销毁的方法
        
        ```java
        @WebServlet("/demo2")
        public class SessionDemo2 extends HttpServlet {
            @Override
            protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                //获取数据，从session中
        
                //1. 获取Session对象
                HttpSession session = request.getSession();
                System.out.println(session);
        
                // 销毁
                session.invalidate();
                //2. 获取数据
                Object username = session.getAttribute("username");
                System.out.println(username);
            }
        
            @Override
            protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                this.doGet(request, response);
            }
        }
        ```
        
    - 启动访问测试，先访问demo1将数据存入到session，再次访问demo2从session中获取数据，浏览器显示session失效