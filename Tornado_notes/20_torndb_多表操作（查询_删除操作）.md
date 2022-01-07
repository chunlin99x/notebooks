#### 更新只有单表操作 ####

```

#查询学生选课情况
def query(stuname):
    conn = torndb.Connection(host='127.0.0.1:3306', database='testlogin', user='root', password='123456')
    datas = conn.query('select * from t_student s,t_stu_course sc,t_course cour where s.sno =sc.sno and sc.courseid = cour.courseid and s.sname="%s"'%stuname)
    conn.close()

    return datas

# print query('zhangsan')

#删除选择某门课程
def delete(coursename):
    conn = torndb.Connection(host='127.0.0.1:3306', database='testlogin', user='root', password='123456')
    conn.execute('delete from t_course where courname ="%s"'%coursename)
    conn.close()

# delete('Python基础')

```