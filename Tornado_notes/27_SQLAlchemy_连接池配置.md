```
#配置连接池

from sqlalchemy.engine import create_engine

#配置引擎
conn_url = 'mysql://root:123456@127.0.0.1:3306/tornado?charset=utf8'
engine = create_engine(conn_url,encoding='utf-8',echo=False,pool_size=10,max_overflow=20,pool_recycle=3600,pool_timeout=3600)
#pool_size=10  核心连接数为10
#max_overflow=20 非核心连接数为20(超过核心连接数后最多创建20个连接)
#pool_recycle=3600 等待3600秒（1小时）后回收非核心连接数
#pool_timeout=3600 所有连接数都被占用时，剩余请求连接将等待3600秒获取连接接，如果获取不到，将连接失败~


```