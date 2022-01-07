```

#插入操作(多表插入操作)
def insert(cname,sname,score,content,coursenames=[]):
    from sqlalchemy.orm.session import sessionmaker
    db_session = sessionmaker(bind=engine)
    session = db_session()


    try:
        #创建班级
        session.begin_nested()  # 保存断点
        if session.query(Clazz.cno).filter(Clazz.cname==cname).count()==1:
            cls =session.query(Clazz.cno).filter(Clazz.cname==cname).first()
        else:
            cls = Clazz(cname=cname)
            session.add(cls)
            session.commit()  # 提交事务
            session.flush([cls,])#将改变提交到数据库
            print cls.cno

        #创建学生
        import datetime
        stu = Student(sname=sname,score=score,birth=datetime.datetime.strptime('1983-01-02','%Y-%m-%d'),desc=content,cno=cls.cno)
        session.add(stu)
        session.commit()
        session.flush([stu,])


        #创建课程
        cs = []
        print coursenames
        for courname in coursenames:
            if session.query(Course).filter(Course.coursename==courname).count()==1:

                course = session.query(Course).filter(Course.coursename==courname).first()


            else:
                course = Course(coursename=courname)
                session.add(course)
                session.commit()
                session.flush([course,])

            print course
            cs.append(course)

        #插入课程学生中间表内容
        for c in cs:
            sc = Stu_Course(sno=stu.sno,courseid=c.courseid)
            session.add(sc)
            session.commit()
            session.flush([sc,])




    except Exception as e:
        print e
        session.rollback()

    session.close()


# insert(cname,sname,score,content,coursenames=[])
# insert('B209Python班','wangwu',100,'Django学习',coursenames=['Python','D'])



#多表查询
def query():
    from sqlalchemy.orm.session import sessionmaker
    db_sesion = sessionmaker(bind=engine)
    session = db_sesion()

    #查询同班同学学生信息
    #笛卡尔积（交叉连接）
    # datas = session.query(Clazz,Student).all()

    #等值连接
    # datas = session.query(Clazz,Student).filter(Clazz.cno==Student.cno).all()

    #非等值连接
    # datas = session.query(Clazz,Student).filter(Clazz.cno>8,Clazz.cno==Student.cno).all()

    #内连接
    # datas = session.query(Clazz,Student).join(Student,Clazz.cno==Student.cno).all()

    #外连接（只有左连接）
    # datas = session.query(Clazz,Student).outerjoin(Student,Clazz.cno==Student.cno).all()

    #原生查询
    sql ='select * from t_cls'
    datas =session.execute(sql)

    session.close()
    return datas

# print query()
print query().fetchall()



```