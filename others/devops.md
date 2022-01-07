# python实现tail实时查看服务器日志



import paramiko
from paramiko_expect import SSHClientInteraction

host = your host
port = your port
username = your un


# 自行修改输出函数
json_list = []
def output_func(msg):  

```python
sys.stdout.write(msg)
json_list.append(msg)
sys.stdout.flush()
```

 


def conn_tail(path):

```python
try:
    client = paramiko.SSHClient()
    client.set_missing_host_key_policy(paramiko.AutoAddPolicy)
    key_file = 'id_rsa_2048'
    key = paramiko.RSAKey.from_private_key_file(key_file, 'yourpwd')
 
    client.connect(host, port, username, key_filename=key_file)
    interact = SSHClientInteraction(client, timeout=10, display=False)
 
    interact.send('sudo su\n')
    interact.expect(prompt)
    interact.send('tail -f %s' % path)
    # log_name = path.split('/')[-1].split('.')[0]
    # interact.tail(line_prefix=log_name + ':  ',output_callback=output_func)
    interact.tail( output_callback=output_func)
```
1.使用了paramiko_expect模块，安装方式

