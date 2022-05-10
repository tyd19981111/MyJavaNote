# Response

# 1. Reponse继承体系

- 如下图所示
    
    ![Untitled](Response%20f09a626594644b1f8ca9f1f16f25c0e4/Untitled.png)
    

# 2. ****Response设置响应数据功能介绍****

### 2.1. 响应行

![Untitled](Response%20f09a626594644b1f8ca9f1f16f25c0e4/Untitled%201.png)

- 对于响应头，比较常用的就是设置响应状态码:
    
    `void setStatus(int sc);`
    

### 2.2. 响应头

![Untitled](Response%20f09a626594644b1f8ca9f1f16f25c0e4/Untitled%202.png)

- 设置响应头键值对：
    
    `void setHeader(String name,String value);`
    

### 2.3. 响应体

![Untitled](Response%20f09a626594644b1f8ca9f1f16f25c0e4/Untitled%203.png)

- 对于响应体，是通过字符、字节输出流的方式往浏览器写
    - 获取字符输出流：`PrintWriter getWriter();`
    - 获取字节输出：`ServletOutputStream getOutputStream();`

# 3. ****Respones请求重定向****

- Response重定向(redirect)：一种资源跳转方式
    
    ![Untitled](Response%20f09a626594644b1f8ca9f1f16f25c0e4/Untitled%204.png)
    
    - 浏览器发送请求给服务器，服务器中对应的资源A接收到请求
    - 资源A现在无法处理该请求，就会给浏览器响应一个302的状态码+location的一个访问资源B的路径
    - 浏览器接收到响应状态码为302就会重新发送请求到location对应的访问地址去访问资源B
    - 资源B接收到请求后进行处理并最终给浏览器响应结果，这整个过程就叫重定向
- 重定向的实现方式
    
    ```java
    resp.setStatus(302);
    resp.setHeader("location","资源B的访问路径");
    ```
    
- 案例
    - 创建ResponseDemo1类
        
        ```java
        @WebServlet("/resp1")
        public class ResponseDemo1 extends HttpServlet {
            @Override
            protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                System.out.println("resp1....");
                //重定向
                //1.设置响应状态码 302
                response.setStatus(302);
                //2. 设置响应头 Location
                response.setHeader("Location","/request-demo/resp2");
            }
        
            @Override
            protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                this.doGet(request, response);
            }
        }
        ```
        
        <aside>
        💡 简化：`resposne.sendRedirect("/request-demo/resp2")`
        
        </aside>
        
    - 创建ResponseDemo2类
        
        ```java
        @WebServlet("/resp2")
        public class ResponseDemo2 extends HttpServlet {
            @Override
            protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                System.out.println("resp2....");
            }
        
            @Override
            protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                this.doGet(request, response);
            }
        }
        ```
        
    - 测试：访问`http://localhost:8080/request-demo/resp1`
- 重定向特点
    - 浏览器地址栏路径发生变化
    - 可以重定向到任何位置的资源(服务内部、外部均可)
    - 两次请求，不能在多个资源使用request共享数据

![Untitled](Response%20f09a626594644b1f8ca9f1f16f25c0e4/Untitled%205.png)

# 4. ****路径问题****

- Q&A
    - 转发的时候路径上没有加项目路径`/request-demo`而重定向加了，什么时候需要加，什么时候不需要加呢?
        
        ![Untitled](Response%20f09a626594644b1f8ca9f1f16f25c0e4/Untitled%206.png)
        
        - Ans：浏览器使用需要加虚拟目录(项目访问路径)；服务端使用不需要加虚拟目录
        
        <aside>
        💡 对于转发来说，因为是在服务端进行的，所以不需要加虚拟目录
        
        对于重定向来说，路径最终是由浏览器来发送请求，就需要添加虚拟目录
        
        </aside>
        
- 动态获取项目路径
    - pom中配置
        
        ```xml
        <build>
        <plugins>
        	<plugin>
        		<groupId>org.apache.tomcat.maven</groupId>
        		<artifactId>tomcat7-maven-plugin</artifactId>
        		<version>2.2</version>
        		<configuration>
        			<!--配置项目的访问地址-->
        			<path>/request-demo</path>
        		</configuration>-->
        	</plugin>
        </plugins>
        </build>
        ```
        
    - 使用request对象中的getContextPath()方法获取动态路径
        
        ```java
        @WebServlet("/resp1")
        public class ResponseDemo1 extends HttpServlet {
            @Override
            protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                System.out.println("resp1....");
        
                //简化方式完成重定向
                //动态获取虚拟目录
                String contextPath = request.getContextPath();
                response.sendRedirect(contextPath+"/resp2");
            }
        
            @Override
            protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                this.doGet(request, response);
            }
        }
        ```
        

# 5. ****Response响应字符数据****

- 要想将字符数据写回到浏览器，我们需要两个步骤:
    - 通过Response对象获取字符输出流： PrintWriter writer = resp.getWriter();
    - 通过字符输出流写数据: writer.write("aaa");
- 案例
    - 返回一个简单的字符串`aaa`
        
        ```java
        /**
         * 响应字符数据：设置字符数据的响应体
         */
        @WebServlet("/resp3")
        public class ResponseDemo3 extends HttpServlet {
            @Override
            protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                response.setContentType("text/html;charset=utf-8");
                //1. 获取字符输出流
                PrintWriter writer = response.getWriter();
        		 writer.write("aaa");
            }
            @Override
            protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                this.doGet(request, response);
            }
        }
        ```
        
    - 返回一串html字符串，并且能被浏览器解析
        
        ```java
        PrintWriter writer = response.getWriter();
        //content-type，告诉浏览器返回的数据类型是HTML类型数据，这样浏览器才会解析HTML标签
        response.setHeader("content-type","text/html");
        writer.write("<h1>aaa</h1>");
        ```
        
    - 返回一个中文的字符串`你好`，需要注意设置响应数据的编码为`utf-8`
        
        ```java
        //设置响应的数据格式及数据的编码
        response.setContentType("text/html;charset=utf-8");
        writer.write("你好");
        ```
        
    
    <aside>
    💡 一次请求响应结束后，response对象就会被销毁掉，所以不用手动关闭流
    
    </aside>
    

# 6. ****Response响应字节数据****

- 要想将字节数据写回到浏览器，我们需要两个步骤:
    - 通过Response对象获取字节输出流：ServletOutputStream outputStream = resp.getOutputStream();
    - 通过字节输出流写数据: outputStream.write(字节数据);
- 案例
    - 返回一个图片文件到浏览器
        
        ```java
        /**
         * 响应字节数据：设置字节数据的响应体
         */
        @WebServlet("/resp4")
        public class ResponseDemo4 extends HttpServlet {
            @Override
            protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                //1. 读取文件
                FileInputStream fis = new FileInputStream("d://a.jpg");
                //2. 获取response字节输出流
                ServletOutputStream os = response.getOutputStream();
                //3. 完成流的copy
                byte[] buff = new byte[1024];
                int len = 0;
                while ((len = fis.read(buff))!= -1){
                    os.write(buff,0,len);
                }
                fis.close();
            }
        
            @Override
            protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                this.doGet(request, response);
            }
        }
        ```
        
    
    <aside>
    💡 上述代码中，对于流的copy的代码还是比较复杂的，所以我们可以使用别人提供好的方法来简化代码的开发
    
    </aside>
    
    - pom.xml添加依赖
        
        ```xml
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.6</version>
        </dependency>
        ```
        
    - 调用工具类方法
        
        ```java
        //fis:输入流
        //os:输出流
        IOUtils.copy(fis,os);
        ```
        
    - 优化后的代码
        
        ```java
        /**
         * 响应字节数据：设置字节数据的响应体
         */
        @WebServlet("/resp4")
        public class ResponseDemo4 extends HttpServlet {
            @Override
            protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                //1. 读取文件
                FileInputStream fis = new FileInputStream("d://a.jpg");
                //2. 获取response字节输出流
                ServletOutputStream os = response.getOutputStream();
                //3. 完成流的copy
              	IOUtils.copy(fis,os);
                fis.close();
            }
        
            @Override
            protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                this.doGet(request, response);
            }
        }
        ```