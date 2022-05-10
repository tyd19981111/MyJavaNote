# Filter&Listener

# 1. ****Filter****

## 1.1. Filter概述

![Untitled](Filter&Listener%20ab8957ca4aa74a29a71c1c67c5daa5b8/Untitled.png)

- Filter 表示过滤器，是 JavaWeb 三大组件(Servlet、Filter、Listener)之一
- 过滤器可以把对资源的请求拦截下来，从而实现一些特殊的功能

## 1.2. ****Filter入门****

- Filter开发步骤
    - 定义类，实现 Filter接口，并重写其所有方法
    - 配置Filter拦截资源的路径：在类上定义 `@WebFilter`注解。而注解的 `value`属性值表示拦截所有的资源
    - 在doFilter方法中执行操作，并放行
- 案例演示
    - `pom.xml`配置文件内容如下：
        
        ```xml
        <dependencies>
                <dependency>
                    <groupId>javax.servlet</groupId>
                    <artifactId>javax.servlet-api</artifactId>
                    <version>3.1.0</version>
                    <scope>provided</scope>
                </dependency>
            </dependencies>
        ```
        
    - Webapp下创建`hello.jsp`页面，内容如下：
        
        ```html
        <%@ page contentType="text/html;charset=UTF-8" language="java" %>
        <html>
        <head>
            <title>Title</title>
        </head>
        <body>
            <h1>hello JSP~</h1>
        </body>
        </html>
        ```
        
    - 编写过滤器
        
        ```java
        @WebFilter("/*")
        public class FilterDemo implements Filter {
        
            @Override
            public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
                //执行操作
        				System.out.println("1.FilterDemo...");
                //放行
                chain.doFilter(request,response);
            }
        
            @Override
            public void init(FilterConfig filterConfig) throws ServletException {
            }
        
            @Override
            public void destroy() {
            }
        }
        ```
        
        <aside>
        💡 在doFilter中执行操作后，一般要放行：`chain.doFilter(request,response);`
        
        </aside>
        

## 1.3. ****Filter执行流程****

- 执行流程图
    
    ![Untitled](Filter&Listener%20ab8957ca4aa74a29a71c1c67c5daa5b8/Untitled%201.png)
    
- 总结：执行放行前逻辑 ==》 放行 ==》 访问资源 ==》 执行放行后逻辑
- 验证
    - 修改`doFilter()` 方法
        
        ```java
        //1．放行前，对request数据进行处理
        system.out.print1n("1.FilterDemo. . .");
        //放行
        chain.doFilter(request,response);
        //2．放行后，对Response数据进行处理
        system.out.println("3.FilterDemo. . .");
        ```
        
    - 在 `hello.jsp`页面加上输出语句，如下
        
        ```java
        <%
        system.out.println( "2.hello jsp");
        %>
        ```
        
        测试：执行访问该资源打印的顺序是按照我们标记的标号进行打印的话，说明我们上边总结出来的流程是没有问题的
        
    
    <aside>
    💡 以后我们可以将对请求进行处理的代码放在放行之前进行处理，而如果请求完资源后还要对响应的数据进行处理时可以在放行后进行逻辑处理
    
    </aside>
    

## 1.4. ****Filter拦截路径配置****

- 拦截具体的资源：/index.jsp：只有访问index.jsp时才会被拦截
- 目录拦截：/user/*：访问/user下的所有资源，都会被拦截
- 后缀名拦截：*.jsp：访问后缀名为jsp的资源，都会被拦截
- 拦截所有：/*：访问所有资源，都会被拦截

## 1.5. ****过滤器链****

- 概述
    
    ![Untitled](Filter&Listener%20ab8957ca4aa74a29a71c1c67c5daa5b8/Untitled%202.png)
    
    1. 执行 `Filter1` 的放行前逻辑代码
    2. 执行 `Filter1` 的放行代码
    3. 执行 `Filter2` 的放行前逻辑代码
    4. 执行 `Filter2` 的放行代码
    5. 访问到资源
    6. 执行 `Filter2` 的放行后逻辑代码
    7. 执行 `Filter1` 的放行后逻辑代码
- 案例演示
    - 编写第一个过滤器 `FilterDemo`，配置成拦截所有资源
        
        ```java
        @WebFilter("/*")
        public class FilterDemo implements Filter {
        
            @Override
            public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        
                //1. 放行前，对 request数据进行处理
                System.out.println("1.FilterDemo...");
                //放行
                chain.doFilter(request,response);
                //2. 放行后，对Response 数据进行处理
                System.out.println("3.FilterDemo...");
            }
        
            @Override
            public void init(FilterConfig filterConfig) throws ServletException {
            }
        
            @Override
            public void destroy() {
            }
        }
        ```
        
    - 编写第二个过滤器 `FilterDemo2` ，配置成拦截所有资源
        
        ```java
        @WebFilter("/*")
        public class FilterDemo2 implements Filter {
        
            @Override
            public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        
                //1. 放行前，对 request数据进行处理
                System.out.println("2.FilterDemo...");
                //放行
                chain.doFilter(request,response);
                //2. 放行后，对Response 数据进行处理
                System.out.println("4.FilterDemo...");
            }
        
            @Override
            public void init(FilterConfig filterConfig) throws ServletException {
            }
        
            @Override
            public void destroy() {
            }
        }
        ```
        
    - 测试：启动服务器，在浏览器输入 `http://localhost/filter-demo/hello.jsp`
    
    <aside>
    ❓ 上面代码中为什么是先执行 `FilterDemo`，后执行 `FilterDemo2`呢？
    
    - 我们现在使用的是注解配置Filter，而这种配置方式的优先级是按照过滤器类名(字符串)的自然排序。
    - 比如有如下两个名称的过滤器 ： `BFilterDemo`和 `AFilterDemo` 。那一定是 `AFilterDemo`过滤器先执行。
    </aside>
    

# 2. ****Listener****

## 2.1. ****概述****

- Listener 表示监听器，是 JavaWeb 三大组件(Servlet、Filter、Listener)之一
- 监听器可以监听就是在 `application`，`session`，`request`三个对象创建、销毁或者往其中添加修改删除属性时自动执行代码的功能组件
- request 和 session 我们学习过。而 `application`是 `ServletContext`类型的对象
- `ServletContext`代表整个web应用，在服务器启动的时候，tomcat会自动创建该对象。在服务器关闭时会自动销毁该对象

## 2.2. 分类

- JavaWeb 提供了8个监听器
    
    ![Untitled](Filter&Listener%20ab8957ca4aa74a29a71c1c67c5daa5b8/Untitled%203.png)
    

## 2.3. 代码演示

- 演示 `ServletContextListener`监听器
    - ServletContextListener接口中有以下两个方法
        - `void contextInitialized(ServletContextEvent sce)`：`ServletContext` 对象被创建了会自动执行的方法
        - `void contextDestroyed(ServletContextEvent sce)`：`ServletContext` 对象被销毁时会自动执行的方法
    - 代码
        
        ```java
        @WebListener
        public class ContextLoaderListener implements ServletContextListener {
            @Override
            public void contextInitialized(ServletContextEvent sce) {
                //加载资源
                System.out.println("ContextLoaderListener...");
            }
        
            @Override
            public void contextDestroyed(ServletContextEvent sce) {
                //释放资源
            }
        }
        ```
        
    - 测试
        - 启动服务器，就可以在启动的日志信息中看到 `contextInitialized()`方法输出的内容，同时也说明了 `ServletContext`对象在服务器启动的时候被创建了