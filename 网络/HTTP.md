# HTTP

- 概念：Hyper Text Transfer Protocol 超文本传输协议
  - 传输协议：定义了，客户端和服务器端通信时，发送数据的格式
  - 特点：
    1. 基于TCP/IP的高级协议
    2. 默认端口号:80
    3. 基于请求/响应模型的:一次请求对应一次响应
    4. 无状态的：每次请求之间相互独立，不能交互数据
  - 历史版本：
    - 1.0：每一次请求响应都会建立新的连接
    - 1.1：复用连接

---

##  请求消息数据格式

1. 请求行

   `请求方式 请求url 请求协议/版本
   GET /login.html	HTTP/1.1`

   - 请求方式：

     -  HTTP协议有7中请求方式，常用的有2种

       1.  GET：

          > 请求参数在请求行中，在url后。
          >
          > 请求的url长度有限制的
          >
          > 不太安全

       2. POST：

          > 请求参数在请求体中
          >
          > 请求的url长度没有限制的
          >
          > 相对安全

2. 请求头：客户端浏览器告诉服务器一些信息

   请求头名称: 请求头值

   - 常见的请求头：

     1. User-Agent：浏览器告诉服务器，我访问你使用的浏览器版本信息

        - 可以在服务器端获取该头的信息，解决浏览器的兼容性问题

     2. Referer：http://localhost/login.html

        - 告诉服务器，我(当前请求)从哪里来？
          -  作用：
            1.  防盗链：
            2. 统计工作：

     3. 请求空行

        - 空行，就是用于分割POST请求的请求头，和请求体的。

     4. 请求体(正文)：

        - 封装POST请求消息的请求参数的

     5. 字符串格式

        ```
        		POST /login.html	HTTP/1.1
        		Host: localhost
        		User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:60.0) Gecko/20100101 Firefox/60.0
        		Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
        		Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
        		Accept-Encoding: gzip, deflate
        		Referer: http://localhost/login.html
        		Connection: keep-alive
        		Upgrade-Insecure-Requests: 1
        		
        		username=zhangsan
        ```



---

## 响应消息数据格式

服务器端发送给客户端的数据

- 数据格式：

  1. 响应行

     1. 组成：协议/版本 响应状态码 状态码描述

     2. 响应状态码：服务器告诉客户端浏览器本次请求和响应的一个状态。

        1. 状态码都是3位数字 

        2. 分类：

           > 1xx：服务器接收客户端消息，但没有接受完成，等待一段时间后，发送1xx多状态码						
           >
           > 2xx：成功。代表：200
           > 3xx：重定向。代表：302(重定向)，304(访问缓存)
           > 4xx：客户端错误。
           > 	 404（请求路径没有对应的资源） 
           > 	 405：请求方式没有对应的doXxx方法
           > 5xx：服务器端错误。代表：500(服务器内部出现异常)

  2. 响应头：

     1. 格式：头名称： 值

     2. 常见的响应头：

        1. Content-Type：服务器告诉客户端本次响应体数据格式以及编码格式

        2.  Content-disposition：服务器告诉客户端以什么格式打开响应体数据

           - 值：
             - in-line:默认值,在当前页面内打开
             - attachment;filename=xxx：以附件形式打开响应体。文件下载

        3.  响应空行

        4.  响应体:传输的数据

           ```html
           响应字符串格式
           		HTTP/1.1 200 OK
           		Content-Type: text/html;charset=UTF-8
           		Content-Length: 101
           		Date: Wed, 06 Jun 2018 07:08:42 GMT
           
           		<html>
           		  <head>
           		    <title>$Title$</title>
           		  </head>
           		  <body>
           		  hello , response
           		  </body>
           		</html>
           ```

           

