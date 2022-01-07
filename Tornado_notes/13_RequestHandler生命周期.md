```

#coding=utf-8

"""

RequestHandler生命周期模拟底层实现

"""
class BaseHandler(object):
    def initlize(self):
        print '初始化'

    def get(self,*args,**kwargs):
        raise Exception(405)

    def post(self,*args,**kwargs):
        raise Exception(405)

    def on_finish(self,*args,**kwargs):
        print '释放资源'

class IndexHandler(BaseHandler):
    def initlize(self,conn):
        print 'Index初始化',conn

    def get(self,*args,**kwargs):
        print 'GET请求处理'

    def on_finish(self,*args,**kwargs):
        print 'IndexHandler请求释放资源'


if __name__ == '__main__':
    urlpatterns = [(r'/',IndexHandler,{'hello':'123'})]
    path = '/'
    method = 'get'
    for url in urlpatterns:
        p,c,k = url
        if path == p:
            h = c()
            h.initlize(k)
            getattr(h,method)()
            h.on_finish()



```



```
#coding=utf-8

import tornado.web
import tornado.ioloop
"""
生命周期讲解
"""
class LoginHandler(tornado.web.RequestHandler):
    def initialize(self,conn):
        self.conn = conn

    def prepare(self):
        if self.request.method =='POST':
            self.uname = self.get_argument('uname')
            self.pwd = self.get_argument('pwd')
        #表单校验_查询数据库操作也可以在这边完成

    def get(self, *args, **kwargs):
        self.render('templates/login1.html')

    def post(self, *args, **kwargs):
        #1/0  有异常跳转到write_error()处理
        cursor = self.conn.cursor()
        cursor.execute('select * from t_auser where account="%s" and pwd="%s"'%(self.uname,self.pwd))
        user = cursor.fetchone()
        print user,'================'
        cursor.close()

        if user:
            self.write('登录成功！')
        else:
            self.redirect('/login/')


    def on_finish(self):
        print '释放资源'


    def write_error(self, status_code, **kwargs):
        self.write('出错啦~~')

    def set_default_headers(self):
        self.set_header('Server','hahapai')

db_config={
    'host':'127.0.0.1',
    'port':3306,
    'db':'testlogin',
    'user':'root',
    'passwd':'123456'
}
import MySQLdb

app = tornado.web.Application([
    (r'/login/',LoginHandler,{'conn':MySQLdb.connect(**db_config)})
])
app.listen(8888)

tornado.ioloop.IOLoop.instance().start()


 




```





















