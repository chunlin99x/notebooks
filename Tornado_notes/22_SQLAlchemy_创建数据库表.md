#### 实现步骤 ####



1. 安装sqlalchemy模块 
```
    pip install sqlalchemy
```

2. 配置引擎
```
from sqlalchemy.engine import create_engine

    conn_url = 'mysql://root:123456@127.0.0.1:3306/testlogin?charset=utf8'
    engine = create_engine(conn_url,encoding='utf-8',echo=True)
    
```

3. 声明ORM基类（这个基类的子类会自动和数据库表进行关联）
```
from sqlalchemy.ext.declarative import declarative_base
    Base = declarative_base(bind=engine)
    
```

4. 导入列和数据类型
```
from sqlalchemy import Column
from sqlalchemy.types import Integer,String,Date,DateTime,Float,Text

```

5. 创建ORM类
```
class User(Base):
    __tablename__='t_cuser'

    id = Column(Integer,primary_key=True,autoincrement=True)
    account = Column(String(length=8),unique=True)
    pwd = Column(String(length=3))
    birth = Column(Date)
    score = Column(Float(decimal_return_scale=2))

    def __repr__(self):
        return '[User:%s,%s]'%(self.id,self.account)

class Address(Base):
    __tablename__='t_addr'

    id = Column(Integer,primary_key=True,autoincrement=True)
    aname = Column(String(30),unique=True)
    

```


6. 利用基类创建数据库表
```
    #如果表已经存在，则不执行当前存在表的创建操作
    Base.metadata.create_all()



    #7.利用基类删除所有的数据库表
    Base.metadata.drop_all()

```


   



















