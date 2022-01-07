Tornado 的定义是 Web 框架和异步网络库，其中他具备有异步非阻塞能力，能解决他两个框架请求阻塞的问题，在需要并发时候就应该使用 Tornado。

Tornado 异步使用方式
简而言之，Tornado的异步包括两个方面，异步服务端和异步客户端。无论服务端和客户端，具体的异步模型又可以分为回调（callback）和协程（coroutine）。具体应用场景，也没有很明确的界限。往往一个请求服务里还包含对别的服务的客户端异步请求。


tornado.web.asynchronous的作用是保持长连接，也就是除非你主动调用self.finish()方法，否则requestHandler将不会返回。

tornado.gen.coroutine是使用协程的方式实现类似异步的处理效果。最新版的tornado，其实不一定需要写tornado.web.asynchronous装饰器


- 在 Tornado 中两个装饰器：
```
tornado.web.asynchronous
tornado.gen.coroutine

```
asynchronous 装饰器是让请求变成长连接的方式，必须手动调用 self.finish() 才会响应。
```
class MainHandler(tornado.web.RequestHandler):
    @tornado.web.asynchronous
    def get(self):
        # 错误方式
        self.write("Hello, world")
        
```

```
class MainHandler(tornado.web.RequestHandler):
    @tornado.web.asynchronous
    def get(self):
        # 正确方式
        self.write("Hello, world")
        self.finish()
```
coroutine 装饰器是指定改请求为协程模式，说明白点就是能使用 yield 配合 Tornado 编写异步程序。

Tronado 为协程实现了一套自己的协议，不能使用 Python 普通的生成器。



#### 实现案例 ####

1. 回调函数
```

#coding=utf-8

import tornado.web
import tornado.ioloop
import os

"""
异步服务器端处理方式1
"""


class IndexHandler(tornado.web.RequestHandler):

    @tornado.web.asynchronous
    def get(self,filename, *args, **kwargs):
        BaseDir = os.path.join(os.getcwd(),'static')
        file = os.path.join(BaseDir,filename)

        content = None

        with open(file,'rb') as fr:
            content = fr.read()

        if not content:
            self.write_error(404)
        else:
            self.set_header('Content-Type', 'image/png')
            self.write(content)
            self.finish()  #asychronous方式支持长连接，需要手动结束请求才能返回响应


app = tornado.web.Application([
    (r'/static/(.*)',IndexHandler)
])

app.listen(8888)

tornado.ioloop.IOLoop.instance().start()


```


2. 协程方式
```
#coding=utf-8

import tornado.web
import tornado.ioloop
import os
from tornado import gen
from tornado.concurrent import Future


"""
异步服务器端处理方式2
"""
class IndexHandler(tornado.web.RequestHandler):

    @gen.coroutine
    def get(self,filename, *args, **kwargs):
        #协程让特定方法完成这部分操作
        content = yield self.readImg(filename)


        if not content:
            self.write_error(404)
        else:
            self.set_header('Content-Type', 'image/png')
            self.write(content)


    def readImg(self,filename):
        BaseDir = os.path.join(os.getcwd(), 'static')
        file = os.path.join(BaseDir, filename)

        content = None

        with open(file, 'rb') as fr:
            content = fr.read()


        #类似于生成器中的send()
        future = Future()
        future.set_result(content)

        return future


app = tornado.web.Application([
    (r'/static/(.*)',IndexHandler)
])

app.listen(8888)

tornado.ioloop.IOLoop.instance().start()


```



