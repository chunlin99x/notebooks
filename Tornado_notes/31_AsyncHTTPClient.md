```
#coding=utf-8


"""
异步请求客户端

"""
from tornado.httpclient import AsyncHTTPClient

def parse(content):
    import bs4
    bs = bs4.BeautifulSoup(content,'html.parser')
    atext = [a.text for a in bs.select('ul.foot_nav.main a')]
    for a in atext:
        print a


def handle_response(response):
    body = response.body
    # print body

    import os
    BASE_DIR = os.path.join(os.getcwd(),'templates')
    file = os.path.join(BASE_DIR,'index.html')

    with open(file,'wb') as fw:
        fw.write(body)

    parse(body)







def load1(url,callback):
    http_client = AsyncHTTPClient()
    http_client.fetch(url, callback=callback)


load1('http://www.bjsxt.com',callback=handle_response)


import tornado.ioloop
tornado.ioloop.IOLoop.instance().start()#协程调度器



```