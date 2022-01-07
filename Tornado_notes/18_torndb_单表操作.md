### 简介 ###
torndb是一个轻量级的基于MySQLdb封装的一个模块，从tornado3.0版本以后，其已经作为一个独立模块发行了。
torndb依赖于MySQLdb模块，因此，在使用torndb模块时，要保证系统中已经有MySQLdb模块。


### 使用步骤 ###
1.安装模块
```
pip install torndb

```
2.建立数据库连接
```
import torndb
conn = torndb.Connection(host='127.0.0.1:3306', database='testlogin', user='root', password='123456')

```
3.断开连接
```
conn.close()

```

4.单表CRUD
```
#1.增加一条记录
    conn.insert('insert into t_auser values(null,"%s","%s",now())'%(account,pwd))

#2.增加多条记录
    def insertMany(args=[]):
        sql = 'insert into t_auser values(null,%s,%s,now())'
        conn.insertmany(sql,args)
    # insertMany([('lisi','123'),('wangwu','123')])
    
#3.查询
    sql = 'select * from t_auser'
    users = conn.query(sql)
    
#4.条件查询
    sql = 'select * from t_auser where account ="%s" and pwd ="%s"'%(account,pwd)
    users = conn.query(sql)

#5.模糊查询（以XXX开头）
    sql = 'select * from t_auser where account like "{account}%%"'.format(account=account)
    users = conn.query(sql)
    
#6.模糊查询（以XXX结尾）
    sql = 'select * from t_auser where account like "%%{account}"'.format(account=account)
    users = conn.query(sql)
    
#7.模糊查询（包含XXX）
    sql = 'select * from t_auser where account like "%%{account}%%"'.format(account=account)
    users = conn.query(sql)

    
#8.排序
    def query_all_order_by(rule):
        conn = torndb.Connection(host='127.0.0.1:3306', database='testlogin', user='root', password='123456')
        order = 'ASC'
        if rule.startswith('-'):
            order = 'DESC'
            rule = rule[1:]
        sql = 'select * from t_auser order by %s %s'%(rule,order)
        users = conn.query(sql)
        conn.close()
    
        return users

# print query_all_order_by('-id')


#9.分页
def query_all_page(num,size=2):
    conn = torndb.Connection(host='127.0.0.1:3306', database='testlogin', user='root', password='123456')

    sql = 'select * from t_auser limit %s,%s'%((num-1)*size,size)
    users = conn.query(sql)
    conn.close()

    return users

# print query_all_page(2)


#10.更新
def update(account,newpwd):
    conn = torndb.Connection(host='127.0.0.1:3306', database='testlogin', user='root', password='123456')

    sql = 'update t_auser set pwd ="%s" where account ="%s"'%(newpwd,account)
    conn.update(sql)
    conn.close()


# update('zhangsan','456')


#11.删除
def delete(account):
    conn = torndb.Connection(host='127.0.0.1:3306', database='testlogin', user='root', password='123456')

    sql = 'delete from t_auser where account ="%s"'%(account)
    conn.execute(sql)
    conn.close()


# delete('wangwu')
    
    
    
    
    
```