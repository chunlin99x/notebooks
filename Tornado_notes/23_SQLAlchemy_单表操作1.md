```
#8.数据表中的单表CRUD操作

from sqlalchemy.orm import sessionmaker

 #8.1：增加一条记录
def addUser(account,pwd):
    # 建立数据库连接
    db_session = sessionmaker(bind=engine)  # 相当于创建数据库连接池（默认有5个连接）
    session = db_session()  # 获取连接池中的一个连接

    user = User(account=account,pwd=pwd)
    session.add(user)
    session.commit()
    session.refresh(user)
    session.close()#将连接放回连接池中
    return user

# addUser('zhangsan',123)

 #8.2:增加多条记录
def addManyUser(users=[]):
    # 2.建立数据库连接
    db_session = sessionmaker(bind=engine)  # 相当于创建数据库连接池（默认有5个连接）
    session = db_session()  # 获取连接池中的一个连接

    us =[]
    import datetime
    for ac,pwd,sc in users:
        us.append(User(account=ac,pwd=pwd,score=sc,birth=datetime.datetime.today()))
    session.add_all(us)
    session.commit()#提交事务
    [session.refresh(u) for u in us]#刷新属性值
    session.close()
    return us

# addManyUser(users=[('xiaoming','123',88.9),('xiaohong','123',100)])


  #8.3:增加不同对象数据
def addDiffObj(*args):
    db_session = sessionmaker(bind=engine)
    session = db_session()

    session.add_all(args)
    session.commit()
    [session.refresh(a) for a in args]
    session.close()
    return args

# addDiffObj(User(account='李四',pwd='123',score=66.6),Address(aname='上海市'))


```

```


#解决输出中文乱码问题
import sys
reload(sys)
sys.setdefaultencoding('utf-8')

```