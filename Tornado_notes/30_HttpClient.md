

```
#coding=utf-8


"""
同步请求
"""
from tornado.httpclient import HTTPClient, HTTPRequest


#发送GET同步请求
def getReq(url):
    httpclient = HTTPClient()
    response = httpclient.fetch(url)
    return response.body

#print getReq('http://www.bjsxt.com')

#发送POST同步请求
def postReq(reqeust):
    httpclient = HTTPClient()
    response = httpclient.fetch(reqeust)
    return response.body


request = HTTPRequest(url='http://www.bjsxt.com',method='POST',headers={'User-Agent':'hahaha'},body='uname=zhangsan')
print postReq(request)

```


