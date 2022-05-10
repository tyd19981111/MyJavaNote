# Cookie

# 1. ****会话跟踪技术的概述****

### 1.1 会话

- 用户打开浏览器，访问web服务器的资源，会话建立，直到有一方断开连接，会话结束。
- 在一次会话中可以包含多次请求和响应

### 1.2 会话跟踪

- 一种维护浏览器状态的方法，服务器需要识别多次请求是否来自于同一浏览器
- 服务器识别浏览器后就可以在同一个会话中多次请求之间来共享数据
- 应用场景
    - 购物车：`加入购物车`和`去购物车结算`是两次请求，但是后面这次请求要想展示前一次请求所添加的商品，就需要用到数据共享
    - 页面展示用户登录信息：很多网站，登录后访问其他功能发送请求后，浏览器上都会有当前登录用户的信息[用户名]
    - 网站登录页面的`记住我`功能：当用户登录成功后，勾选`记住我`按钮后下次再登录的时候，网站就会自动填充用户名和密码，简化用户的登录操作
- 实现方式
    - 客户端会话跟踪技术：**Cookie**
    - 服务端会话跟踪技术：**Session**
    
    <aside>
    💡 这两个技术都可以实现会话跟踪，它们之间最大的区别：Cookie是存储在浏览器端而Session是存储在服务器端
    
    </aside>
    
    <aside>
    ❓ Http为什么不能实现请求间数据共享？HTTP协议是无状态的，每次浏览器向服务器请求时，服务器都会将该请求视为新的请求，每次请求之间相互独立
    
    </aside>
    

# 2. ****Cookie****

### 2.1 Cookie工作流程

- Cookie：客户端会话技术，将数据保存到客户端，以后每次请求都携带Cookie数据进行访问
- Cookie的工作流程
    
    ![Untitled](Cookie%20849cd773d5294afe9832d86d42307bca/Untitled.png)
    
    - 服务端提供了两个Servlet，分别是ServletA和ServletB
    - 浏览器发送HTTP请求1给服务端，服务端ServletA接收请求并进行业务处理
    - 服务端ServletA在处理的过程中可以创建一个Cookie对象并将`name=zs`的数据存入Cookie
    - 服务端ServletA在响应数据的时候，会把Cookie对象响应给浏览器
    - 浏览器接收到响应数据，会把Cookie对象中的数据存储在浏览器内存中，此时浏览器和服务端就==建立了一次会话==
    - 在同一次会话中浏览器再次发送HTTP请求2给服务端ServletB，浏览器会携带Cookie对象中的所有数据
    - ServletB接收到请求和数据后，就可以获取到存储在Cookie对象中的数据，这样同一个会话中的多次请求之间就实现了数据共享

### 2.2 Cookie基本使用

- 发送Cookie
    - 创建Cookie对象，并设置数据
        
        `Cookie cookie = new Cookie("key","value");`
        
    - 发送Cookie到客户端：使用==response==对象
        
        `response.addCookie(cookie);`
        
- 获取Cookie
    - 获取客户端携带的所有Cookie，使用request对象
        
        `Cookie[] cookies = request.getCookies();`
        
    - 使用Cookie对象方法获取数据
        
        `cookie.getName();
        cookie.getValue();`
        
- 案例演示
    - 创建Maven项目cookie-demo，并在pom.xml添加依赖
        
        ```xml
        <dependencies>
            <!--servlet-->
            <dependency>
                <groupId>javax.servlet</groupId>
                <artifactId>javax.servlet-api</artifactId>
                <version>3.1.0</version>
                <scope>provided</scope>
            </dependency>
            <!--jsp-->
            <dependency>
                <groupId>javax.servlet.jsp</groupId>
                <artifactId>jsp-api</artifactId>
                <version>2.2</version>
                <scope>provided</scope>
            </dependency>
            <!--jstl-->
            <dependency>
                <groupId>jstl</groupId>
                <artifactId>jstl</artifactId>
                <version>1.2</version>
            </dependency>
            <dependency>
                <groupId>taglibs</groupId>
                <artifactId>standard</artifactId>
                <version>1.1.2</version>
            </dependency>
        </dependencies>
        ```
        
    - 编写Servlet类，名称为AServlet，用来发送Cookie
        
        ```java
        @WebServlet("/aServlet")
        public class AServlet extends HttpServlet {
            @Override
            protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                //发送Cookie
                //1. 创建Cookie对象
                Cookie cookie = new Cookie("username","zs");
                //2. 发送Cookie，response
                response.addCookie(cookie);
            }
        
            @Override
            protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                this.doGet(request, response);
            }
        }
        ```
        
        测试：浏览器访问`http://localhost:8080/cookie-demo/aServlet` ，使用chrome浏览器查看Cookie的值（两种方式）
        
    - 编写一个新Servlet类，名称为BServlet，用来获取Cookie
        
        ```java
        @WebServlet("/bServlet")
        public class BServlet extends HttpServlet {
            @Override
            protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                //获取Cookie
                //1. 获取Cookie数组
                Cookie[] cookies = request.getCookies();
                //2. 遍历数组
                for (Cookie cookie : cookies) {
                    //3. 获取数据
                    String name = cookie.getName();
                    if("username".equals(name)){
                        String value = cookie.getValue();
                        System.out.println(name+":"+value);
                        break;
                    }
                }
        
            }
        
            @Override
            protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                this.doGet(request, response);
            }
        }
        ```
        
        测试：访问`http://localhost:8080/cookie-demo/bServlet`
        
    
    <aside>
    ❓ 在访问AServlet和BServlet的中间把关闭浏览器,重启浏览器后访问BServlet能否获取到Cookie中的数据?
    
    </aside>
    

### 2.3 ****Cookie的原理分析****

- Cookie的实现原理是基于HTTP协议的,其中设计到HTTP协议中的两个请求头信息:
    - 响应头:set-cookie
    - 请求头: cookie
    
    ![Untitled](Cookie%20849cd773d5294afe9832d86d42307bca/Untitled%201.png)
    
- 验证
    - 访问AServlet对应的地址`http://localhost:8080/cookie-demo/aServlet`
    - 打开开发者工具查看响应头中的数据
    - 访问BServlet对应的地址`http://localhost:8080/cookie-demo/bServlet`
    - 打开开发者工具查看请求头中的数据

### 2.4 ****Cookie持久化存储****

- Cookie的存活时间
    - 默认情况下，Cookie存储在浏览器内存中，当浏览器关闭，内存释放，则Cookie被销毁
- Cookie持久化存储
    - 设置Cookie存活时间
        
        `setMaxAge(int seconds)`
        
        > 参数值取值：
        > 
        > 
        > 正数：将Cookie写入浏览器所在电脑的硬盘，持久化存储。到时间自动删除
        > 
        > 负数：默认值，Cookie在当前浏览器内存中，当浏览器关闭，则Cookie被销毁
        > 
        > 零：删除对应Cookie
        > 
    - 例如，在AServlet中去设置Cookie的存活时间为一周：`cookie.setMaxAge(60*60*24*7);`

### 2.5 ****Cookie存储中文****

- 方案：使用URL编码和解码
- 演示：修改Aservlet和Bservlet代码
    - Aservlet，设置URL编码
    
    ```java
    @WebServlet("/aServlet")
    public class AServlet extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            //发送Cookie
            String value = "张三";
            //对中文进行URL编码
            value = URLEncoder.encode(value, "UTF-8");
            System.out.println("存储数据："+value);
            //将编码后的值存入Cookie中
            Cookie cookie = new Cookie("username",value);
            //设置存活时间   ，1周 7天
            cookie.setMaxAge(60*60*24*7);
            //2. 发送Cookie，response
            response.addCookie(cookie);
        }
    
        @Override
        protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            this.doGet(request, response);
        }
    }
    ```
    
    - Bservlet，设置URL解码
    
    ```java
    @WebServlet("/bServlet")
    public class BServlet extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            //获取Cookie
            //1. 获取Cookie数组
            Cookie[] cookies = request.getCookies();
            //2. 遍历数组
            for (Cookie cookie : cookies) {
                //3. 获取数据
                String name = cookie.getName();
                if("username".equals(name)){
                    String value = cookie.getValue();//获取的是URL编码后的值 %E5%BC%A0%E4%B8%89
                    //URL解码
                    value = URLDecoder.decode(value,"UTF-8");
                    System.out.println(name+":"+value);//value解码后为 张三
                    break;
                }
            }
    
        }
    
        @Override
        protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            this.doGet(request, response);
        }
    }
    ```