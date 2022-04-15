先来看一下**WSGI的介绍**：

全称Python Web Server Gateway Interface，指定了web服务器和Python web应用或web框架之间的标准接口，以提高web应用在一系列web服务器间的移植性。 具体可查看 [官方文档](https://www.python.org/dev/peps/pep-0333/)

从以上介绍我们可以看出：

1. WSGI是一套接口标准协议/规范；
2. 通信（作用）区间是Web服务器和Python Web应用程序之间；
3. 目的是制定标准，以保证不同Web服务器可以和不同的Python程序之间相互通信

你可能会问，**为什么需要WSGI？**

首先，我们明确一下web应用处理请求的具体流程：

1. 用户操作操作浏览器发送请求；
2. 请求转发至对应的web服务器
3. web服务器将请求转交给web应用程序，web应用程序处理请求
4. web应用将请求结果返回给web服务器，由web服务器返回用户响应结果
5. 浏览器收到响应，向用户展示

可以看到，请求时Web服务器需要和web应用程序进行通信，但是web服务器有很多种啊，Python web应用开发框架也对应多种啊，所以WSGI应运而生，定义了一套通信标准。试想一下，如果不统一标准的话，就会存在Web框架和Web服务器数据无法匹配的情况，那么开发就会受到限制，这显然不合理的。

既然定义了标准，那么**WSGI的标准或规范是？**

web服务器在将请求转交给web应用程序之前，需要先将http报文转换为WSGI规定的格式。

WSGI规定，Web程序必须有一个可调用对象，且该可调用对象接收两个参数，返回一个可迭代对象：

1. environ：字典，包含请求的所有信息
2. start_response：在可调用对象中调用的函数，用来发起响应，参数包括状态码，headers等

通过以上学习，一起**实现一个简单WSGI服务吧**

首先，我们编写一个符合WSGI标准的一个http处理函数：

```python
def hello(environ, start_response):
    status = "200 OK"
    response_headers = [('Content-Type', 'text/html')]
    start_response(status, response_headers)
    path = environ['PATH_INFO'][1:] or 'hello'
    return [b' %s ' % path.encode()]
```

该方法负责获取environ字典中的path_info，也就是获取请求路径，然后在前端展示。

接下来，我们需要一个服务器启动WSGI服务器用来处理验证，使用Python内置的WSGI服务器模块wsgiref，编写server.py：

```python
# coding:utf-8
"""
desc: WSGI服务器实现
"""
from wsgiref.simple_server import make_server
from learn_wsgi.client import hello
def main():
    server = make_server('localhost', 8001, hello)
    print('Serving HTTP on port 8001...')
    server.serve_forever()
if __name__ == '__main__':
    main()
```

执行python server.py，浏览器打开"http://localhost:8001/a"，即可验证。

通过实现一个简单的WSGI服务，我们可以看到：通过environ可以获取http请求的所有信息，http响应的数据都可以通过start_response加上函数的返回值作为body。

当然，以上只是一个简单的案例，那么在python的Web框架内部是如何遵循WSGI规范的呢？以Flask举例，

**Flask与WSGI**

Flask中的程序实例app就是一个可调用对象，我们创建app实例时所调用的Flask类实现了__call__方法，__call__方法调用了wsgi_app()方法，该方法完成了请求和响应的处理，WSGI服务器通过调用该方法传入请求数据，获取返回数据：

```python
def wsgi_app(self, environ, start_response):
    ctx = self.request_context(environ)
    error = None
    try:
        try:
            ctx.push()
            response = self.full_dispatch_request()
        except Exception as e:
            error = e
            response = self.handle_exception(e)
        except:  # noqa: B001
            error = sys.exc_info()[1]
            raise
        return response(environ, start_response)
    finally:
        if self.should_ignore_error(error):
            error = None
        ctx.auto_pop(error)

def __call__(self, environ, start_response):
    return self.wsgi_app(environ, start_response)
```