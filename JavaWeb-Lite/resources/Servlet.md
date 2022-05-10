# Servlet

# 简介

![Untitled](Servlet%2009d3f64506ca49b4ae504907ce2c36ac/Untitled.png)

- Servlet是JavaWeb最为核心的内容，它是Java提供的一门**动态**的web资源开发技术
- Servlet是JavaEE规范之一，其实就是一个接口，将来我们需要定义Servlet类实现Servlet接口，并由web服务器运行Servlet

# ****快速入门****

<aside>
➡️ 需求分析: 编写一个Servlet类，并使用IDEA中Tomcat插件进行部署，最终通过浏览器访问所编写的Servlet程序

</aside>

- 创建Web项目`web-demo`，导入Servlet依赖坐标
    
    ```xml
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
        <!--
          此处为什么需要添加该标签?
          provided指的是在编译和测试过程中有效,最后生成的war包时不会加入
           因为Tomcat的lib目录中已经有servlet-api这个jar包，如果在生成war包的时候生效就会和Tomcat中的jar包冲突，导致报错
        -->
        <scope>provided</scope>
    </dependency>
    ```
    
- 定义一个类，实现Servlet接口，并重写接口中所有方法，并在service方法中输出一句话
    
    ```java
    package com.itheima.web;
    
    import javax.servlet.*;
    import java.io.IOException;
    
    public class ServletDemo1 implements Servlet {
    
        public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
            System.out.println("servlet hello world~");
        }
        public void init(ServletConfig servletConfig) throws ServletException {
    
        }
    
        public ServletConfig getServletConfig() {
            return null;
        }
    
        public String getServletInfo() {
            return null;
        }
    
        public void destroy() {
    
        }
    }
    ```
    
- 配置:在类上使用**@WebServlet**注解，配置该Servlet的访问路径
    
    ```java
    @WebServlet("/demo1")
    ```
    
- 访问:启动Tomcat,浏览器中输入URL地址(`http://localhost:8080/web-demo/demo1`)访问该Servlet
- 浏览器访问后，在控制台会打印`servlet hello world~`，说明servlet程序已经成功运行

# Servlet执行流程分析

<aside>
❓ Servlet程序已经能正常运行，但是我们需要思考个问题: 我们并没有创建ServletDemo1类的对象，也没有调用对象中的service方法，为什么在控制台就打印了`servlet hello world~`这句话呢?

**要想回答上述问题，我们就需要对Servlet的执行流程进行一个学习。**

</aside>

![Untitled](Servlet%2009d3f64506ca49b4ae504907ce2c36ac/Untitled%201.png)

- 浏览器发出`http://localhost:8080/web-demo/demo1`请求，从请求中可以解析出三部分内容，分别是`localhost:8080`、`web-demo`、`demo1`
    - 根据`localhost:8080`可以找到要访问的Tomcat Web服务器
    - 根据`web-demo`可以找到部署在Tomcat服务器上的web-demo项目
    - 根据`demo1`可以找到要访问的是项目中的哪个Servlet类，根据@WebServlet后面的值进行匹配
- 找到ServletDemo1这个类后，**Tomcat Web服务器就会为ServletDemo1这个类创建一个对象**，**然后调用对象中的service方法**
    - ServletDemo1实现了Servlet接口，所以类中必然会重写service方法供Tomcat Web服务器进行调用
    - service方法中有ServletRequest和ServletResponse两个参数，ServletRequest封装的是请求数据，ServletResponse封装的是响应数据，后期我们可以通过这两个参数实现前后端的数据交互

# Servlet生命周期

<aside>
📢 Servlet运行在Servlet容器(web服务器)中，其生命周期由容器来管理

</aside>

- 加载和实例化
    - 默认情况下，当Servlet第一次被访问时，由容器创建Servlet对象
    
    <aside>
    💡 默认情况，Servlet会在第一次访问被容器创建，但是如果创建Servlet比较耗时的话，那么第一个访问的人等待的时间就比较长，用户的体验就比较差，那么我们能不能把Servlet的创建放到服务器启动的时候来创建，具体如何来配置?
    
    ```java
    @WebServlet(urlPatterns = "/demo1",loadOnStartup = 1)
    //loadOnstartup的取值有两类情况
    //	（1）负整数:第一次访问时创建Servlet对象
    //	（2）0或正整数:服务器启动时创建Servlet对象，数字越小优先级越高
    ```
    
    </aside>
    
- 初始化
    - 在Servlet实例化之后，容器将调用Servlet的**`init()`**方法初始化这个对象，完成一些如加载配置文件、创建连接等初始化的工作。该方法只**调用一次**
- 请求处理
    - 每次请求Servlet时，Servlet容器都会调用Servlet的**`service()`**方法对请求进行处理
- 服务终止
    - 当需要释放内存或者容器关闭时，容器就会调用Servlet实例的**`destroy()`**方法完成资源的释放。在destroy()方法调用之后，容器会释放这个Servlet实例，该实例随后会被Java的垃圾收集器所回收

**通过案例演示下上述的生命周期**

```java
package com.itheima.web;

import javax.servlet.*;
import javax.servlet.annotation.WebServlet;
import java.io.IOException;
/**
* Servlet生命周期方法
*/
@WebServlet(urlPatterns = "/demo2",loadOnStartup = 1)
public class ServletDemo2 implements Servlet {

    /**
     *  初始化方法
     *  1.调用时机：默认情况下，Servlet被第一次访问时，调用
     *      * loadOnStartup: 默认为-1，修改为0或者正整数，则会在服务器启动的时候，调用
     *  2.调用次数: 1次
     * @param config
     * @throws ServletException
     */
    public void init(ServletConfig config) throws ServletException {
        System.out.println("init...");
    }

    /**
     * 提供服务
     * 1.调用时机:每一次Servlet被访问时，调用
     * 2.调用次数: 多次
     * @param req
     * @param res
     * @throws ServletException
     * @throws IOException
     */
    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        System.out.println("servlet hello world~");
    }

    /**
     * 销毁方法
     * 1.调用时机：内存释放或者服务器关闭的时候，Servlet对象会被销毁，调用
     * 2.调用次数: 1次
     */
    public void destroy() {
        System.out.println("destroy...");
    }
    public ServletConfig getServletConfig() {
        return null;
    }

    public String getServletInfo() {
        return null;
    }

}
```

<aside>
💡 destroy方法只有Serclet停止（如Tomcat关闭）时才会执行

</aside>

# Servlet****方法介绍****

- 初始化方法，在Servlet被创建时执行，只执行一次
    
    ```java
    void init(ServletConfig config)
    ```
    
- 提供服务方法， 每次Servlet被访问，都会调用该方法
    
    ```java
    void service(ServletRequest req,ServletResponse res)
    ```
    
- 销毁方法，当Servlet被销毁时，调用该方法。在内存释放或服务器关闭时销毁Servlet
    
    ```java
    void destroy()
    ```
    
- 获取Servlet信息
    
    ```java
    String getServletInfo() 
    //该方法用来返回Servlet的相关信息，没有什么太大的用处，一般我们返回一个空字符串即可
    public String getServletInfo() {
        return "";
    }
    ```
    
- 获取ServletConfig对象
    - **`ServletConfig** **getServletConfig**()`
    - ServletConfig对象，在init方法的参数中有，而Tomcat Web服务器在创建Servlet对象的时候会调用init方法，必定会传入一个ServletConfig对象，我们只需要将服务器传过来的ServletConfig进行返回即可
    
    ```java
    @WebServlet(urlPatterns = "/demo3",loadOnStartup = 1)
    public class ServletDemo3 implements Servlet {
    
        private ServletConfig servletConfig;
     
        public void init(ServletConfig config) throws ServletException {
            this.servletConfig = config;
            System.out.println("init...");
        }
        public ServletConfig getServletConfig() {
            return servletConfig;
        }
    }
    ```
    
    <aside>
    📢 getServletInfo()和getServletConfig()这两个方法使用的不是很多，了解即可
    
    </aside>
    

# HttpServlet

- Servlet体系结构
    
    ![Untitled](Servlet%2009d3f64506ca49b4ae504907ce2c36ac/Untitled%202.png)
    
    <aside>
    💡 因为我们将来开发B/S架构的web项目，都是针对HTTP协议，所以我们自定义Servlet一般都会通过继承HttpServlet来实现
    
    </aside>
    
- Servlet实现doGet和doPost方法
    
    <aside>
    📢 前端发送GET和POST请求的时候，参数的位置不一致，GET请求参数在请求行中，POST请求参数在请求体中，为了能处理不同的请求方式，我们得在service方法中进行判断，然后写不同的业务处理
    
    </aside>
    
    ```java
    public class MyHttpServlet implements Servlet {
        public void init(ServletConfig config) throws ServletException {
    
        }
    
        public ServletConfig getServletConfig() {
            return null;
        }
    
        public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
            HttpServletRequest request = (HttpServletRequest)req;
            //1. 获取请求方式
            String method = request.getMethod();
            //2. 判断
            if("GET".equals(method)){
                // get方式的处理逻辑
                doGet(req,res);
            }else if("POST".equals(method)){
                // post方式的处理逻辑
                doPost(req,res);
            }
        }
        protected void doPost(ServletRequest req, ServletResponse res) {
        }
    
        protected void doGet(ServletRequest req, ServletResponse res) {
        }
    
     
    }
    ```
    
    有了MyHttpServlet这个类，以后我们再编写Servlet类的时候，只需要继承MyHttpServlet，重写父类中的doGet和doPost方法，就可以用来处理GET和POST请求的业务逻辑
    
- HttpServlet
    
    <aside>
    📢 类似MyHttpServlet这样的类Servlet中已经为我们提供好了，就是HttpServlet
    
    </aside>
    
    ```java
    @WebServlet("/demo4")
    public class ServletDemo4 extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            //TODO GET 请求方式处理逻辑
            System.out.println("get...");
        }
        @Override
        protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            //TODO Post 请求方式处理逻辑
            System.out.println("post...");
        }
    }
    ```
    
    翻开HttpServlet源码，大家可以搜索`service()`方法，你会发现HttpServlet做的事更多，不仅可以处理GET和POST还可以处理其他五种请求方式
    

# ****urlPattern配置****

- 精确匹配
    - `@WebServlet(urlPatterns = "/user/select")`
    - 访问路径：localhost:8080/web-demo/user/select
- 目录匹配
    - `@WebServlet(urlPatterns = "/user/*")`
    - 访问路径http://localhost:8080/web-demo/user/任意
- 扩展名匹配
    - `@WebServlet(urlPatterns = "*.do")`
    - 访问路径http://localhost:8080/web-demo/任意.do
- 任意匹配
    - `@WebServlet(urlPatterns = "/*")`
    - 访问路径[http://localhost:8080/demo-web/任意](https://gitee.com/link?target=http%3A%2F%2Flocalhost%3A8080%2Fdemo-web%2F%25E4%25BB%25BB%25E6%2584%258F)

# Servlet的Xml配置

- Servlet从3.0版本后开始支持注解配置，3.0版本前只支持XML配置文件的配置方法
- XML的配置步骤有两步
    - 编写Servlet类
        
        ```java
        public class ServletDemo13 extends MyHttpServlet {
        
            @Override
            protected void doGet(ServletRequest req, ServletResponse res) {
        
                System.out.println("demo13 get...");
            }
            @Override
            protected void doPost(ServletRequest req, ServletResponse res) {
            }
        }
        ```
        
    - 在web.xml中配置该Servlet
        
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
                 version="4.0">
            <!-- 
                Servlet 全类名
            -->
            <servlet>
                <!-- servlet的名称，名字任意-->
                <servlet-name>demo13</servlet-name>
                <!--servlet的类全名-->
                <servlet-class>com.itheima.web.ServletDemo13</servlet-class>
            </servlet>
        
            <!-- 
                Servlet 访问路径
            -->
            <servlet-mapping>
                <!-- servlet的名称，要和上面的名称一致-->
                <servlet-name>demo13</servlet-name>
                <!-- servlet的访问路径-->
                <url-pattern>/demo13</url-pattern>
            </servlet-mapping>
        </web-app>
        
        ```