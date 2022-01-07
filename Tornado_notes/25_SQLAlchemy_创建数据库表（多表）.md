```
#coding=utf-8

#多表操作（创建多表）

from sqlalchemy.engine import create_engine

#配置引擎
conn_url = 'mysql://root:123456@127.0.0.1:3306/tornado?charset=utf8'
engine = create_engine(conn_url,encoding='utf-8',echo=True)


#创建基表
from sqlalchemy.ext.declarative import declarative_base
Base = declarative_base(bind=engine)

#导入列和数据类型
from sqlalchemy import Column,ForeignKey
from sqlalchemy.types import Integer,Float,String,Text,Date,DateTime


#建表
#班级表 t_cls
#学生表 t_stu
#课程表 t_course

class Clazz(Base):
    __tablename__='t_cls'

    cno = Column(Integer,primary_key=True,autoincrement=True)
    cname = Column(String(20),unique=True,nullable=False)

    def __repr__(self):
        return u'<Clazz:%s,%s>'%(self.cno,self.cname)

import datetime
class Student(Base):
    __tablename__='t_stu'

    sno = Column(Integer,primary_key=True,autoincrement=True)
    sname = Column(String(20),unique=True,nullable=False)
    score = Column(Float(decimal_return_scale=2),default=10.00)
    birth = Column(Date,default=datetime.date.today())
    desc = Column(Text,nullable=True)
    cno = Column(Integer,ForeignKey(Clazz.cno,ondelete='CASCADE'))

    def __repr__(self):
        return u'<Student:%s,%s,%s>'%(self.sno,self.sname,self.cno)
class Course(Base):
    __tablename__='t_course'

    courseid = Column(Integer,primary_key=True,autoincrement=True)
    coursename = Column(String(20),nullable=False)

    def __repr__(self):
        return u'<Course:%s,%s>'%(self.courseid,self.coursename)


class Stu_Course(Base):
    __tablename__='t_sc'

    id = Column(Integer,primary_key=True,autoincrement=True)
    sno = Column(Integer,ForeignKey(Student.sno,ondelete='CASCADE'),nullable=False)
    courseid = Column(Integer,ForeignKey(Course.courseid,ondelete='CASCADE'),nullable=False)

    def __repr(self):
        return u'<Stu_Course:%s,%s>'%(self.sno,self.courseid)

#建表语句
Base.metadata.create_all()
























```