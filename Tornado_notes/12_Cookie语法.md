#### 设置Cookie值

```
#普通方式
    self.set_cookie('k1','v1',expires_days=3)
    
    self.get_cookie('k1')
 
#加密方式   
    self.set_secure_cookie('k1','v1')
    
    'cookie_secret':'fdsa'
    
    self.get_secure_cookie('k1','v1')

```