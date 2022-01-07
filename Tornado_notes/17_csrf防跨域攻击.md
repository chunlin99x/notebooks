```
#表单
{% raw xsrf_form_html() %}


#Application配置

settings={'debug':True,'xsrf_cookies': True}

app = tornado.web.Application([
    (r'^/login/$',IndexHandler),
    ...

],**settings)

```

