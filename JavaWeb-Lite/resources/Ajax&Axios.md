# Ajax&Axios

# 1. Ajax

## 1.1 概述

- `AJAX` (Asynchronous JavaScript And XML)：异步的 JavaScript 和 XML
- 概念中的 `JavaScript` 和 `XML`，`JavaScript` 表明该技术和前端相关；`XML` 是指以此进行数据交换

## 1.2 Ajax作用

- 与服务器进行数据交换
    
    ![Untitled](Ajax&Axios%208a29e2ae62e34053b708c559e6922999/Untitled.png)
    
    - 浏览器发送请求servlet，servlet 调用完业务逻辑层后将数据直接响应回给浏览器页面，页面使用 HTML 来进行数据展示
    - Ajax+Html的方式替代了Jsp
- 异步交互
    - 可以在不重新加载整个页面的情况下，与服务器交换数据并更新部分网页
- 补充：同步和异步
    - 同步发送请求过程如下
        
        ![Untitled](Ajax&Axios%208a29e2ae62e34053b708c559e6922999/Untitled%201.png)
        
        - 浏览器页面在发送请求给服务器，在服务器处理请求的过程中，浏览器页面不能做其他的操作。只能等到服务器响应结束后才能，浏览器页面才能继续做其他的操作
    - 异步发送请求过程如下
        
        ![Untitled](Ajax&Axios%208a29e2ae62e34053b708c559e6922999/Untitled%202.png)
        
        - 浏览器页面发送请求给服务器，在服务器处理请求的过程中，浏览器页面还可以做其他的操作

## 1.3 快速入门

- 服务端实现
    - 创建AjaxServlet类
        
        ```java
        @WebServlet("/ajaxServlet")
        public class AjaxServlet extends HttpServlet {
            @Override
            protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                //1. 响应数据
                response.getWriter().write("hello ajax~");
            }
        
            @Override
            protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
                this.doGet(request, response);
            }
        }
        ```
        
    - 在 `webapp`下创建名为 `01-ajax-demo1.html`的页面，在该页面书写 `ajax`代码
        
        ```jsx
        <!DOCTYPE html>
        <html lang="en">
        <head>
            <meta charset="UTF-8">
            <title>Title</title>
        </head>
        <body>
        
        <script>
            //1. 创建核心对象
            var xhttp;
            if (window.XMLHttpRequest) {
                xhttp = new XMLHttpRequest();
            } else {
                // code for IE6, IE5
                xhttp = new ActiveXObject("Microsoft.XMLHTTP");
            }
            //2. 发送请求
            xhttp.open("GET", "http://localhost:8080/ajax-demo/ajaxServlet");
            xhttp.send();
        
            //3. 获取响应
            xhttp.onreadystatechange = function() {
                if (this.readyState == 4 && this.status == 200) {
                    alert(this.responseText);
                }
            };
        </script>
        </body>
        </html>
        ```
        
    - 测试：在浏览器地址栏输入 `http://localhost:8080/ajax-demo/01-ajax-demo1.html`
    - 使用开发者工具查看network一栏，类型为xhr的即为Ajax请求

# 2. Axios

Axios 是对原生的AJAX进行封装，简化书写。

Axios官网是：`https://www.axios-http.cn`

## 2.1 ****基本使用****

- 引入 axios 的 js 文件
    
    ```jsx
    <script src="js/axios-0.18.0.js"></script>
    ```
    
- 使用axios 发送请求，并获取响应结果
    - 发送 get 请求
        
        ```jsx
        axios({    
        	//用来设置请求方式的
        	method:"get",    
        	//用来书写请求的资源路径
        	url:"http://localhost:8080/ajax-demo1/aJAXDemo1?username=zhangsan"
        }).then(function (resp){    
        		alert(resp.data);
        })
        ```
        
    - 发送 post 请求
        
        ```jsx
        axios({    
        	method:"post",    
        	url:"http://localhost:8080/ajax-demo1/aJAXDemo1",    
        	//设置请求体
        	data:"username=zhangsan"
        }).then(**function** (resp){    
        		alert(resp.data);
        });
        ```
        
    
    <aside>
    💡 将 `then()`中传递的匿名函数称为**回调函数**，意思是该匿名函数在发送请求时不会被调用，而是在成功响应后调用的函数。而该回调函数中的 `resp`参数是对响应的数据进行封装的对象，通过 `resp.data`可以获取到响应的数据。
    
    </aside>
    

## 2.2 ****快速入门****

- 后端实现
    
    ```jsx
    @WebServlet("/axiosServlet")
    public class AxiosServlet extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            System.out.println("get...");
            //1. 接收请求参数
            String username = request.getParameter("username");
            System.out.println(username);
            //2. 响应数据
            response.getWriter().write("hello Axios~");
        }
    
        @Override
        protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            System.out.println("post...");
            this.doGet(request, response);
        }
    }
    ```
    
- 前端实现
    
    ```jsx
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
    </head>
    <body>
    
    <script src="js/axios-0.18.0.js"></script>
    <script>
        //1. get
       axios({
            method:"get",
            url:"http://localhost:8080/ajax-demo/axiosServlet?username=zhangsan"
        }).then(function (resp) {
            alert(resp.data);
        })
    
        //2. post  在js中{} 表示一个js对象，而这个js对象中有三个属性
        axios({
            method:"post",
            url:"http://localhost:8080/ajax-demo/axiosServlet",
            data:"username=zhangsan"
        }).then(function (resp) {
            alert(resp.data);
        })
    </script>
    </body>
    </html>
    ```
    

## 2.3 ****请求方法别名****

- 为了方便起见， Axios 已经为所有支持的请求方法提供了别名。如下：
    - `get` 请求 ： `axios.get(url[,config])`
    - `delete` 请求 ： `axios.delete(url[,config])`
    - `head` 请求 ： `axios.head(url[,config])`
    - `options` 请求 ： `axios.option(url[,config])`
    - `post` 请求：`axios.post(url[,data[,config])`
    - `put` 请求：`axios.put(url[,data[,config])`
    - `patch` 请求：`axios.patch(url[,data[,config])`
- 入门案例中的 `get` 请求代码可以改为如下：
    
    ```jsx
    axios.get("http://localhost:8080/ajax-demo/axiosServlet?username=zhangsan").then(function (resp) {alert(resp.data);});
    ```
    
- 入门案例中的 `post` 请求代码可以改为如下：
    
    ```jsx
    axios.post("http://localhost:8080/ajax-demo/axiosServlet","username=zhangsan").then(**function** (resp) {alert(resp.data);})
    ```