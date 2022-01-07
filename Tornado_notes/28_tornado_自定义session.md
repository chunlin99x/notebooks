session.py

```
#coding=utf-8

import uuid
import pickle

class Session(object):
    def __init__(self):
        self.__sessionid =uuid.uuid4().get_hex()
        self.cache = {}

    def set(self,key,value):
        self.cache[key] = value

    def get(self,key,default=None):
        return self.cache.get(key,default)

    def clear(self):
        self.cache.clear()

    @property
    def sessionid(self):
        return self.__sessionid

    #序列化session对象
    def serialization(self):
        return pickle.dumps(self)


    @staticmethod
    def deserialization(str):
        return pickle.loads(str)

import redis
class SessionManager(object):
    conn = redis.Redis(host='127.0.0.1',port=6379,db=3)

    @classmethod
    def cache2Redis(cls,session):
        cls.conn.set(session.sessionid,session.serialization(),ex=14*24*60*60)


    @classmethod
    def getSessionBySid(cls,sessionid):
        rs = Session.deserialization(cls.conn.get(sessionid)) if cls.conn.get(sessionid) else None

        if not rs:
            rs = Session()

        return rs











```

views.py

```
#coding=utf-8

import tornado.web
from session import *
from models import *

class BaseHandler(tornado.web.RequestHandler):
    def prepare(self):
        #从cookie中获取sessionid
        c_sessionid = self.get_cookie('sessionid')
        session = SessionManager.getSessionBySid(c_sessionid)

        if c_sessionid!=session.sessionid:
            self.set_cookie('sessionid',session.sessionid,expires_days=14)

        self.session = session


    def on_finish(self):
        SessionManager.cache2Redis(self.session)


class LoginHandler(BaseHandler):
    def get(self, *args, **kwargs):
        self.render('login.html')

    def post(self, *args, **kwargs):
        account = self.get_argument('account','')
        pwd = self.get_argument('pwd','')
        user = User(account=account,pwd=pwd)

        if account=='zhangsan' and pwd =='123':
            self.session.set('user',user)
            self.redirect('/user')

class UserHandler(BaseHandler):
    def get(self, *args, **kwargs):
        user = self.session.get('user')
        self.write(u'欢迎%s登录成功！'%user.account)











```