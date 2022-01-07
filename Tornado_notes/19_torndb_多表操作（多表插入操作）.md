```

"""
多表CRUD

场景：
    学生表（3） 课程表（主表2）：N - N
    学生表 班级表（主表1）：N - 1


"""

import torndb


#一次调用，在多张表中插入数据
def insert(clsname,stuname,coursename=[]):
    conn = torndb.Connection(host='127.0.0.1:3306', database='testlogin', user='root', password='123456')

    #1.确保班级表中有数据，并且获得clsid
    sql = 'select clsid from t_cls where clsname ="%s"'%clsname
    cls = conn.query(sql)
    if not cls:
        clsid = conn.insert('insert into t_cls values(null,"%s")'%clsname)
    else:
        clsid = cls[0]['clsid']

    #2.确保课程表中有数据,并且获取courseid
    course_id_list = []
    for cour in coursename:
        course = conn.query('select courseid from t_course where courname="%s"'%cour)
        if course:
            course_id_list.append(course[0]['courseid'])
        else:
            course_id = conn.insert('insert into t_course values(null,"%s")'%cour)
            course_id_list.append(course_id)


    #3.确保学生表中有数据，并获取学生id
    stu = conn.query('select * from t_student where sname="%s"'%stuname)
    if stu:
        stu_id = stu[0]['sno']
    else:
        stu_id = conn.insert('insert into t_student values(null,"%s","%s")'%(stuname,clsid))


    #4.向学生课程表中插入数据
    for courid in course_id_list:
        conn.insert('insert into t_stu_course values("%s","%s")'%(stu_id,courid))

    conn.close()

# insert('B207Python班','zhangsan',['HTML','Python基础'])
# insert('B208Python班','lisi',['JavaScript','Python基础'])



```