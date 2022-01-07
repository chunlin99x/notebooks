#### websocket ####

WebSocket是HTML5规范中新提出的客户端-服务器通讯协议，协议本身使用新的ws://URL格式。

WebSocket 是独立的、创建在 TCP 上的协议，和 HTTP 的唯一关联是使用 HTTP 协议的101状态码进行协议切换，使用的 TCP 端口是80，可以用于绕过大多数防火墙的限制。

WebSocket 使得客户端和服务器之间的数据交换变得更加简单，允许服务端直接向客户端推送数据而不需要客户端进行请求，两者之间可以创建持久性的连接，并允许数据进行双向传送。

目前常见的浏览器如 Chrome、IE、Firefox、Safari、Opera 等都支持 WebSocket，同时需要服务端程序支持 WebSocket。




#### Tornado中的websocket ####

Tornado提供支持WebSocket的模块是tornado.websocket，其中提供了一个WebSocketHandler类用来处理通讯。

WebSocketHandler.open()
当一个WebSocket连接建立后被调用。

WebSocketHandler.on_message(message)
当客户端发送消息message过来时被调用，注意此方法必须被重写。

WebSocketHandler.on_close()
当WebSocket连接关闭后被调用。

WebSocketHandler.write_message(message, binary=False)
向客户端发送消息messagea，message可以是字符串或字典（字典会被转为json字符串）。若binary为False，则message以utf8编码发送；二进制模式（binary=True）时，可发送任何字节码。

WebSocketHandler.close()
关闭WebSocket连接。

WebSocketHandler.check_origin(origin)
判断源origin，对于符合条件（返回判断结果为True）的请求源origin允许其连接，否则返回403。可以重写此方法来解决WebSocket的跨域请求（如始终return True）。




#### 服务器端 ####


```
#coding=utf-8
from tornado.websocket import WebSocketHandler
import tornado.ioloop
import tornado.web

class IndexHandler(WebSocketHandler):
    def open(self):
        print("WebSocket opened")

    def on_message(self, message):
        #向浏览器发送消息
        self.write_message(u"You said: " + message)

    def on_close(self):
        print("WebSocket closed")

    #重写这个方法解决WARNING:tornado.access:403 GET /socket/ 问题
    def check_origin(self, origin):
        return True

    #ping通客户端后的响应
    def on_pong(self, data):
        print data,'on_pong'

class WebHandler(tornado.web.RequestHandler):
    def get(self, *args, **kwargs):
        self.render('web.html')

import os
app = tornado.web.Application([
    (r'/socket/',IndexHandler),
    (r'/',WebHandler),

],template_path=os.path.join(os.getcwd(),'templates'),websocket_ping_interval =1)

app.listen(8888)

tornado.ioloop.IOLoop.instance().start()





```


#### 客户端 ####
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <input type="button" value="建立连接" onclick="conn()"/>
    <input type="button" value="发送消息" onclick="send()"/>
    <input type="button" value="关闭连接" onclick="disconn()"/>

    <script>
        function conn(){

            ws = new WebSocket("ws://127.0.0.1:8888/socket/");
            ws.onopen = function() {
               console.log('连接成功！~')
            };
            ws.onmessage = function (evt) {
               console.log(evt.data);
            };
            ws.onclose =function(){
                console.log('关闭连接了~')
            }
        }

        function send(){
            ws.send('hello world')
        }

        function disconn(){
            ws.close()
        }

    </script>
</body>
</html>

```