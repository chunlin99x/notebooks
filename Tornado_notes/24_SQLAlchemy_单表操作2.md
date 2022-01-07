```

#1.查询表中所有数据（全表查）
def query_all(cls):
    #建立一个连接池
    Session = sessionmaker(bind=engine)
    #从连接池中获取一个连接
    session = Session()
    #查询表中所有数据
    objs = session.query(cls).all()
    #将连接放回连接池中
    session.close()
    #方法返回
    return objs

# print query_all(User)


#2.排序查询
def orderbyCls(cls,column):
    from sqlalchemy.sql.expression import text
    db_session = sessionmaker(bind=engine)
    session = db_session()

    objs = session.query(cls).order_by(text(column)).all()
    #objs = session.query(cls).order_by(cls.id.desc()).all()
    session.close()

    return objs

# print orderbyCls(User,'id')


#3.获取表中记录数
def count(cls):
    db_session = sessionmaker(bind=engine)
    session = db_session()
    c = session.query(cls).count()
    session.close()
    return c

# print count(User)


#4.分页
def page(cls,num,size=2):
    db_session = sessionmaker(bind=engine)
    session = db_session()
    datas = session.query(cls).offset((num-1)*size).limit(size).all()
    session.close()
    return datas

# print page(User,2)


#5.通过主键查询记录
def getClsByPk(cls,pk):
    db_session = sessionmaker(bind=engine)
    session = db_session()
    data = session.query(cls).get(pk)
    session.close()
    return data

# print getClsByPk(User,3)



#6.将公共部分提取成装饰器
def wrapper_session(func):
    def _wrapper(*args,**kwargs):
        from sqlalchemy.orm.session import sessionmaker
        conn_pool = sessionmaker(bind=engine)
        conn = conn_pool()
        data = func(conn,*args,**kwargs)
        conn.close()
        return data
    return _wrapper


#通过某个字段删除一条记录
@wrapper_session
def deleteByCoumn(session,cls,id):
    session.query(cls).filter(cls.id==id).delete()
    session.commit()

# deleteByCoumn(cls=User,id=6)

#通过对象来更新属性
@wrapper_session
def updateUserByAttr(session,obj):
    session.add(obj)
    session.commit()

# u = getClsByPk(User,3)
# u.pwd = '111'
# updateUserByAttr(obj=u)


#filter(单条件查询)
@wrapper_session
def filter1(session,account):
    u = session.query(User).filter(User.account==account).all()
    return u

# print filter1('zhangsan')


#filter(多条件查询)(与的关系)
@wrapper_session
def filter2(session,account,pwd):
    from sqlalchemy import and_
    u = session.query(User).filter(and_(User.account == account, User.pwd == pwd)).all()
    # u = session.query(User).filter((User.account == account, User.pwd == pwd).all()
    return u

# print filter2('zhangsan','123')


#filter(多条件查询)(或的关系)
@wrapper_session
def filter3(session,account,pwd):
    from sqlalchemy import or_
    u = session.query(User).filter(or_(User.account==account,User.pwd==pwd)).all()
    return u

# print filter3('zhangsan','111')



#filter(多条件查询)(非的关系)
@wrapper_session
def filter4(session,account):
    from sqlalchemy import not_
    u = session.query(User).filter(not_(User.account==account)).all()
    return u

# print filter4('zhangsan')

#filter(多条件查询)(嵌套使用)
@wrapper_session
def filter5(session,account,pwd):
    from sqlalchemy import not_,or_
    u = session.query(User).filter(not_(or_(User.account==account,User.pwd==pwd))).all()
    return u

# print filter5('zhangsan','111')



#分组查询
@wrapper_session
def group_by_query(session):
    from sqlalchemy.sql.functions import func
    datas = session.query(func.count(User.id),User.pwd).group_by(User.pwd).all()
    return datas

# print group_by_query()


#查看部分字段的值
@wrapper_session
def query_part(session):
    # datas = session.query(User.id,User.account).all()
    datas = session.query(User.id.label(u'编号'),User.account.label(u'用户名')).all()
    return datas

# print query_part()
```