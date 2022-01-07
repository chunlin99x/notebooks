# ansible 相关

# 一、安装方式

安装方式有两种：

1. YUM 工具
2. PIP 工具

## 1 YUM 工具安装（适用于给系统自带的 python 版本安装 ansilbe 模块）

1. 安装 epel 源



```bash
yum install epel-release  -y
```

1. 安装 Ansible



```bash
yum  install  ansible  -y
```

1. 导入模块



```python
[root@5e4b448b73e5 ~]# python
Python 2.7.5 (default, Aug  7 2019, 00:51:29)
[GCC 4.8.5 20150623 (Red Hat 4.8.5-39)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import ansible
>>> ansible.__file__
'/usr/lib/python2.7/site-packages/ansible/__init__.pyc'
>>>
```

## 2 PIP3 工具安装(使用于给指定版本的 Python 安装 ansible 模块)

1. 创建虚拟环境



```bash
[root@5e4b448b73e5 ~]# pip3 install virtualenvwrapper
```



```bash
export VIRTUALENVWRAPPER_PYTHON=$(which python3)
export WORKON_HOME=$HOME/.virtualenv
source /usr/local/bin/virtualenvwrapper.sh
```



```bash
[root@5e4b448b73e5 ~]# ~/.virtualenv
[root@5e4b448b73e5 ~]# source .bashrc
```

创建虚拟环境



```bash
[root@5e4b448b73e5 ~]# mkvirtualenv ansibleapi
```

1. 安装 ansible



```bash
(ansibleapi) [root@5e4b448b73e5 ~]# pip3 install ansible
```

1. 导入模块



```python
(ansibleapi) [root@5e4b448b73e5 ~]# python3
Python 3.7.6 (default, Aug 11 2020, 10:30:02)
[GCC 4.8.5 20150623 (Red Hat 4.8.5-39)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import ansible
>>> ansible.__file__
'/root/.virtualenv/ansibleapi/lib/python3.7/site-packages/ansible/__init__.py'
>>>
```

3 配置文件位置说明

这种安装方式，配置文件和资产配置文件不会在 `/etc` 下产生。

将会在下面的路径下：



```kotlin
/root/.virtualenv/ansibleapi/lib/python3.7/site-packages/ansible/galaxy/data/container/tests/
```

用到这些文件的时候，需要在 `/etc` 目录想创建目录 `ansible`， 之后把需要的配置文件和资产配置文件放到 `/etc/ansible/` 目录下。

# 二、 先从官方示例入手

#### [点击 官方示例源码 v2.8](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.ansible.com%2Fansible%2Flatest%2Fdev_guide%2Fdeveloping_api.html)

> 注意：下面的所有开发，都是以 `2.8` 版本为例的（ `2.9.9` 已通过测试 ）。`2.7` 和 `2.8` 版本有一些差异：
> 2.7 使用了 Python 标准库里的 命名元组来初始化选项，而 `2.8` 是 Ansible 自己封装了一个 `ImmutableDict` ，之后需要和 `context` 结合使用的。两者不能互相兼容。 [点击 2.7 官方示例](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.ansible.com%2Fansible%2F2.7%2Fdev_guide%2Fdeveloping_api.html)



```python
#!/usr/bin/env python3

import json
import shutil
from ansible.module_utils.common.collections import ImmutableDict
from ansible.parsing.dataloader import DataLoader
from ansible.vars.manager import VariableManager
from ansible.inventory.manager import InventoryManager
from ansible.playbook.play import Play
from ansible.executor.task_queue_manager import TaskQueueManager
from ansible.plugins.callback import CallbackBase
from ansible import context
import ansible.constants as C

class ResultCallback(CallbackBase):
    """A sample callback plugin used for performing an action as results come in

    If you want to collect all results into a single object for processing at
    the end of the execution, look into utilizing the ``json`` callback plugin
    or writing your own custom callback plugin
    """
    def v2_runner_on_ok(self, result, **kwargs):
        """Print a json representation of the result

        This method could store the result in an instance attribute for retrieval later
        """
        host = result._host
        print(json.dumps({host.name: result._result}, indent=4))

# since the API is constructed for CLI it expects certain options to always be set in the context object
context.CLIARGS = ImmutableDict(connection='local', module_path=['/to/mymodules'], forks=10, become=None,
                                become_method=None, become_user=None, check=False, diff=False)

# initialize needed objects
loader = DataLoader() # Takes care of finding and reading yaml, json and ini files
# 注意这里的  vault_pass 是错误的，正确的应该是 conn_pass
passwords = dict(vault_pass='secret')

# Instantiate our ResultCallback for handling results as they come in. Ansible expects this to be one of its main display outlets
results_callback = ResultCallback()

# create inventory, use path to host config file as source or hosts in a comma separated string
inventory = InventoryManager(loader=loader, sources='localhost,')

# variable manager takes care of merging all the different sources to give you a unified view of variables available in each context
variable_manager = VariableManager(loader=loader, inventory=inventory)

# create data structure that represents our play, including tasks, this is basically what our YAML loader does internally.
play_source =  dict(
        name = "Ansible Play",
        hosts = 'localhost',
        gather_facts = 'no',
        tasks = [
            dict(action=dict(module='shell', args='ls'), register='shell_out'),
            dict(action=dict(module='debug', args=dict(msg='{{shell_out.stdout}}')))
         ]
    )

# Create play object, playbook objects use .load instead of init or new methods,
# this will also automatically create the task objects from the info provided in play_source
play = Play().load(play_source, variable_manager=variable_manager, loader=loader)

# Run it - instantiate task queue manager, which takes care of forking and setting up all objects to iterate over host list and tasks
tqm = None
try:
    tqm = TaskQueueManager(
              inventory=inventory,
              variable_manager=variable_manager,
              loader=loader,
              passwords=passwords,
              stdout_callback=results_callback,  # Use our custom callback instead of the ``default`` callback plugin, which prints to stdout
          )
    result = tqm.run(play) # most interesting data for a play is actually sent to the callback's methods
finally:
    # we always need to cleanup child procs and the structures we use to communicate with them
    if tqm is not None:
        tqm.cleanup()

    # Remove ansible tmpdir
    shutil.rmtree(C.DEFAULT_LOCAL_TMP, True)
```

下面是把官方的示例分解开了

## 1. 首先是需要导入的模块



```swift
import json
import shutil
from ansible.module_utils.common.collections import ImmutableDict
from ansible.parsing.dataloader import DataLoader
from ansible.vars.manager import VariableManager
from ansible.inventory.manager import InventoryManager
from ansible.playbook.play import Play
from ansible.executor.task_queue_manager import TaskQueueManager
from ansible.plugins.callback import CallbackBase
from ansible import context
import ansible.constants as C
```

### 核心类介绍

| 导入类完整路径                                               | 功能用途                                                 |
| ------------------------------------------------------------ | -------------------------------------------------------- |
| `from ansible.module_utils.common.collections import ImmutableDict` | 用于添加选项。比如: 指定远程用户`remote_user=None`       |
| `from ansible.parsing.dataloader import DataLoader`          | 读取 json/ymal/ini 格式的文件的数据解析器                |
| `from ansible.vars.manager import VariableManager`           | 管理主机和主机组的变量管理器                             |
| `from ansible.inventory.manager import InventoryManager`     | 管理资源库的，可以指定一个 `inventory` 文件等            |
| `from ansible.playbook.play import Play`                     | 用于执行 Ad-hoc 的类 ,需要传入相应的参数                 |
| `from ansible.executor.task_queue_manager import TaskQueueManager` | ansible 底层用到的任务队列管理器                         |
| `ansible.plugins.callback.CallbackBase`                      | 处理任务执行后返回的状态                                 |
| `from ansible import context`                                | 上下文管理器，他就是用来接收 `ImmutableDict` 的示例对象  |
| `import ansible.constants as C`                              | 用于获取 ansible 产生的临时文档。                        |
| `from ansible.executor.playbook_executor import PlaybookExecutor` | 执行 playbook 的核心类                                   |
| `from ansible.inventory.host import Group`                   | 对 ***主机组\*** 执行操作 ，可以给组添加变量等操作，扩展 |
| `from ansible.inventory.host import Host`                    | 对 ***主机\*** 执行操作 ，可以给主机添加变量等操作，扩展 |

## 2. 回调插件

> 回调插件就是一个类。
> 用于处理执行结果的。
> 后面我们可以改写这个类，以便满足我们的需求。



```python
class ResultCallback(CallbackBase):
    """回调插件，用于对执行结果的回调，
    如果要将执行的命令的所有结果都放到一个对象中，应该看看如何使用 JSON 回调插件或者
    编写自定义的回调插件
    """
    def v2_runner_on_ok(self, result, **kwargs):
        """
        将结果以 json 的格式打印出来
        此方法可以将结果存储在实例属性中以便稍后检索
        """
        host = result._host
        print(json.dumps({host.name: result._result}, indent=4))
```

## 3. 选项

> 这是最新 2.8 中的方式。老版本，请自行谷歌。



```python
# 需要始终设置这些选项，值可以变
context.CLIARGS = ImmutableDict(
  connection='local', module_path=['module_path'],
  forks=10, become=None, become_method=None, 
  become_user=None, check=False, diff=False)
```

## 4. 数据解析器、密码和回调插件对象

> 数据解析器，用于解析 存放主机列表的资源库文件 （比如： `/etc/ansible/hosts`） 中的数据和变量数据的。
> 密码这里是必须使用的一个参数，假如通过了公钥信任，也可以给一个空字典



```bash
# 需要实例化一下
loader = DataLoader()

# ssh 用户的密码, 2.9.9 版本中测试通过，需要安装 sshpass 程序
passwords = dict(conn_pass='secret')

# 实例化一下
results_callback = ResultCallback()
```

## 5. 创建资源库对象

> 这里需要使用数据解析器
> `sources` 的值可以是一个配置好的 `inventory` 资源库文件；也可以是一个含有以 逗号 `,` 为分割符的字符串,
> 注意不是元组哦，正确示例：`sources='localhost,'`
> 这里写的是自己主机上实际的资源库文件



```bash
inventory = InventoryManager(loader=loader, sources='/etc/ansible/hosts')
```

## 6. 变量管理器

> 假如有变量，所有的变量应该交给他管理。
> 这里他会从 `inventory` 对象中获取到所有已定义好的变量。
> 这里也需要数据解析器。



```undefined
variable_manager = VariableManager(loader=loader, inventory=inventory)
```

## 7. 创建一个 Ad-hoc

> 这里创建一个命令行执行的 ansible 命令是可以是多个的。以 `task` 的方式体现。
> 既然都是 python 写的，那么这些变量的值都可以是从数据库中取到的数据或者从前端接收到的参数。
> 开发的雏形若隐若现了...



```java
play_source =  dict(
        name = "Ansible Play",
        hosts = 'nginx',
        gather_facts = 'no',
        tasks = [
            dict(action=dict(module='shell', args='ls'), register='shell_out'),
            dict(action=dict(module='debug', args=dict(msg='{{shell_out.stdout}}')))
         ]
    )
```

## 9. 从 Play 的静态方法 `load` 中创建一个 play 对象。



```undefined
play = Play().load(play_source, variable_manager=variable_manager, loader=loader)
```

## 8. 任务队列管理器

> 要想执行 Ad-hoc ，需要把上面的 play 对象交个任务队列管理器的 `run` 方法去运行。



```python
# 先定义一个值，防止代码出错后，   `finally` 语句中的 `tqm` 未定义。
tqm = None
try:
    tqm = TaskQueueManager(
              inventory=inventory,
              variable_manager=variable_manager,
              loader=loader,
              passwords=passwords,
              stdout_callback=results_callback,  # 这里是使用了之前，自定义的回调插件，而不是默认的回调插件 `default`
          )
    result = tqm.run(play) # 执行的结果返回码，成功是 0
finally:
    # `finally` 中的代码，无论是否发生异常，都会被执行。
    # 如果 `tqm` 不是 `None`, 需要清理子进程和我们用来与它们通信的结构。
    if tqm is not  None:
        tqm.cleanup()

    # 最后删除 ansible 产生的临时目录
    # 这个临时目录会在 ~/.ansible/tmp/ 目录下
    shutil.rmtree(C.DEFAULT_LOCAL_TMP, True)
```

## 总结一下

### Ad-hoc 模式到 API 的映射

![img](https://upload-images.jianshu.io/upload_images/11414906-b41e13d8ab02e57e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png

# 三、二次开发

说完官方示例，那么我们究竟可以使用 ansible API 干些不一样的事情呢？

## 1. 重写回调插件

> 首先，官方示例值的回调函数没有进行进一个的格式化，这个我们可以改写一下这个类里的回调函数。
> 还有，可以增加上失败的信息展示以及主机不可达的信息展示。



```ruby
class ResultCallback(CallbackBase):
    """
    重写callbackBase类的部分方法
    """
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.host_ok = {}
        self.host_unreachable = {}
        self.host_failed = {}
    def v2_runner_on_unreachable(self, result):
        self.host_unreachable[result._host.get_name()] = result

    def v2_runner_on_ok(self, result, **kwargs):
        self.host_ok[result._host.get_name()] = result

    def v2_runner_on_failed(self, result, **kwargs):
        self.host_failed[result._host.get_name()] = result
```

### 如何使用



```css
for host, result in results_callback.host_ok.items():
    print("主机{}, 执行结果{}".format(host, result._result))
```

# 四、如何执行 playbook

![img](https://upload-images.jianshu.io/upload_images/11414906-20cc63166d288e0d.png?imageMogr2/auto-orient/strip|imageView2/2/w/1170/format/webp)

shark

## 1. 先看 `playbook_executor.PlaybookExecutor` 源码



```python
class PlaybookExecutor:

    '''
    This is the primary class for executing playbooks, and thus the
    basis for bin/ansible-playbook operation.
    '''

    def __init__(self, playbooks, inventory, variable_manager, loader, passwords):
        self._playbooks = playbooks
        self._inventory = inventory
        self._variable_manager = variable_manager
        self._loader = loader
        self.passwords = passwords
        self._unreachable_hosts = dict()
...略...
```

## 2. 执行 `playbook`

> 其他用到的部分和 Ad-hoc 方式时的一样



```python
from ansible.executor.playbook_executor import PlaybookExecutor

playbook = PlaybookExecutor(playbooks=['/root/test.yml'],  # 注意这里是一个列表
                 inventory=inventory,
                 variable_manager=variable_manager,
                 loader=loader,
                 passwords=passwords)

# 使用回调函数
playbook._tqm._stdout_callback = results_callback

result = playbook.run()

for host, result in results_callback.host_ok.items():
    print("主机{}, 执行结果{}".format(host, result._result['result']['stdout'])
```

# 五、动态添加主机和主机组

> 使用 ansible API 可以从其他来源中动态的创建资源仓库。比如从 CMDB 的数据库中。

## InventoryManager 类

方法

![img](https://upload-images.jianshu.io/upload_images/11414906-6b59843a8783bde4.png?imageMogr2/auto-orient/strip|imageView2/2/w/686/format/webp)



```csharp
inv = InventoryManager(loader=loader, sources='localhost,')

# 创建一个组  nginx
inv.add_group('nginx')

# 向 nginx 组中添加主机
inv.add_host(host='node1',group='nginx')
inv.add_host(host='node2',group='nginx')

# 获取目前所有的组和主机的信息
In [38]: inv.get_groups_dict()
Out[38]: {'all': ['localhost'], 'ungrouped': ['localhost'], 'nginx': ['node1', 'node2']}
```

> 这样就会动态创建一个 nginx 的组，并且向组内添加了两个主机

# 写到一个类里



```python
import json
import shutil
from ansible.module_utils.common.collections import ImmutableDict
from ansible.parsing.dataloader import DataLoader
from ansible.vars.manager import VariableManager
from ansible.inventory.manager import InventoryManager
from ansible.playbook.play import Play
from ansible.executor.task_queue_manager import TaskQueueManager
from ansible.plugins.callback import CallbackBase
from ansible import context
import ansible.constants as C


class ResultCallback(CallbackBase):
    """
    重写callbackBase类的部分方法
    """
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.host_ok = {}
        self.host_unreachable = {}
        self.host_failed = {}
        self.task_ok = {}
    def v2_runner_on_unreachable(self, result):
        self.host_unreachable[result._host.get_name()] = result

    def v2_runner_on_ok(self, result, **kwargs):
        self.host_ok[result._host.get_name()] = result

    def v2_runner_on_failed(self, result, **kwargs):
        self.host_failed[result._host.get_name()] = result

class MyAnsiable2():
    def __init__(self,
        connection='local',  # 连接方式 local 本地方式，smart ssh方式
        remote_user=None,    # ssh 用户
        remote_password=None,  # ssh 用户的密码，应该是一个字典, key 必须是 conn_pass
        private_key_file=None,  # 指定自定义的私钥地址
        sudo=None, sudo_user=None, ask_sudo_pass=None,
        module_path=None,    # 模块路径，可以指定一个自定义模块的路径
        become=None,         # 是否提权
        become_method=None,  # 提权方式 默认 sudo 可以是 su
        become_user=None,  # 提权后，要成为的用户，并非登录用户
        check=False, diff=False,
        listhosts=None, listtasks=None,listtags=None,
        verbosity=3,
        syntax=None,
        start_at_task=None,
        inventory=None):

        # 函数文档注释
        """
        初始化函数，定义的默认的选项值，
        在初始化的时候可以传参，以便覆盖默认选项的值
        """
        context.CLIARGS = ImmutableDict(
            connection=connection,
            remote_user=remote_user,
            private_key_file=private_key_file,
            sudo=sudo,
            sudo_user=sudo_user,
            ask_sudo_pass=ask_sudo_pass,
            module_path=module_path,
            become=become,
            become_method=become_method,
            become_user=become_user,
            verbosity=verbosity,
            listhosts=listhosts,
            listtasks=listtasks,
            listtags=listtags,
            syntax=syntax,
            start_at_task=start_at_task,
        )

        # 三元表达式，假如没有传递 inventory, 就使用 "localhost,"
        # 指定 inventory 文件
        # inventory 的值可以是一个 资产清单文件
        # 也可以是一个包含主机的元组，这个仅仅适用于测试
        #  比如 ： 1.1.1.1,    # 如果只有一个 IP 最后必须有英文的逗号
        #  或者： 1.1.1.1, 2.2.2.2

        self.inventory = inventory if inventory else "localhost,"

        # 实例化数据解析器
        self.loader = DataLoader()

        # 实例化 资产配置对象
        self.inv_obj = InventoryManager(loader=self.loader, sources=self.inventory)

        # 设置密码
        self.passwords = remote_password

        # 实例化回调插件对象
        self.results_callback = ResultCallback()

        # 变量管理器
        self.variable_manager = VariableManager(self.loader, self.inv_obj)

    def run(self, hosts='localhost', gether_facts="no", module="ping", args='', task_time=0):
        """
        参数说明：
        task_time -- 执行异步任务时等待的秒数，这个需要大于 0 ，等于 0 的时候不支持异步（默认值）。这个值应该等于执行任务实际耗时时间为好
        """
        play_source =  dict(
            name = "Ad-hoc",
            hosts = hosts,
            gather_facts = gether_facts,
            tasks = [
                # 这里每个 task 就是这个列表中的一个元素，格式是嵌套的字典
                # 也可以作为参数传递过来，这里就简单化了。
               {"action":{"module": module, "args": args}, "async": task_time, "poll": 0}])

        play = Play().load(play_source, variable_manager=self.variable_manager, loader=self.loader)

        tqm = None
        try:
            tqm = TaskQueueManager(
                      inventory=self.inv_obj ,
                      variable_manager=self.variable_manager,
                      loader=self.loader,
                      passwords=self.passwords,
                      stdout_callback=self.results_callback)

            result = tqm.run(play)
        finally:
            if tqm is not None:
                tqm.cleanup()
            shutil.rmtree(C.DEFAULT_LOCAL_TMP, True)

    def playbook(self,playbooks):
        """
        Keyword arguments:
        playbooks --  需要是一个列表类型
        """
        from ansible.executor.playbook_executor import PlaybookExecutor

        playbook = PlaybookExecutor(playbooks=playbooks,
                        inventory=self.inv_obj,
                        variable_manager=self.variable_manager,
                        loader=self.loader,
                        passwords=self.passwords)

        # 使用回调函数
        playbook._tqm._stdout_callback = self.results_callback

        result = playbook.run()


    def get_result(self):
      result_raw = {'success':{},'failed':{},'unreachable':{}}

      # print(self.results_callback.host_ok)
      for host,result in self.results_callback.host_ok.items():
          result_raw['success'][host] = result._result
      for host,result in self.results_callback.host_failed.items():
          result_raw['failed'][host] = result._result
      for host,result in self.results_callback.host_unreachable.items():
          result_raw['unreachable'][host] = result._result

      # 最终打印结果，并且使用 JSON 继续格式化
      print(json.dumps(result_raw, indent=4))
```

> 这个类可以执行 Ad-hoc 也可以执行 playbook
> 为了测试方便，这里设置了一些默认值：
> 连接模式采用 `local` ,也就是本地方式
> 资产使用 `"localhost,"`
> 执行 Ad-hoc 时，默认的主机是 `localhost`
>
> 执行 Playbook 时，必须传入 `playbooks` 位置参数，其值是一个包含 `playbook.yml` 文件路径的列表，如:
> 绝对路径写法 `["/root/test.yml"]`
> 当前路径写法 `["test.yml"]`

## 使用范例一：

> ```
> 记得要先建立免密登录
> ```

> #### 执行 默认的 Ad-hoc



```undefined
实例化
ansible2 = MyAnsiable2()

执行 ad-hoc
ansible2.run()

打印结果
ansible2.get_result()
```

输出内容



```csharp
[root@635bbe85037c ~]# /usr/local/bin/python3 /root/code/ansible2api2.8.py
{
    "success": {
        "localhost": {
            "invocation": {
                "module_args": {
                    "data": "pong"
                }
            },
            "ping": "pong",
            "ansible_facts": {
                "discovered_interpreter_python": "/usr/bin/python"
            },
            "_ansible_no_log": false,
            "changed": false
        }
    },
    "failed": {},
    "unreachable": {}
}
```

## 使用范例二:

> #### 执行自定义的 ad-hoc

#### 资产配置文件 `/etc/ansible/hosts` 内容



```css
[nginx]
172.19.0.2
172.19.0.3
```

#### 代码



```java
使用自己的 资产配置文件，并使用 ssh 的远程连接方式
ansible2 = MyAnsiable2(inventory='/etc/ansible/hosts', connection='smart')

执行自定义任务，执行对象是 nginx 组
ansible2.run(hosts= "nginx", module="shell", args='ip a |grep "inet"')

打印结果
ansible2.get_result()
```

输出内容



```php
[root@635bbe85037c ~]# /usr/local/bin/python3 /root/code/ans
ible2api2.8.py
{
    "success": {
        "172.19.0.2": {
            "changed": true,
            "end": "2019-08-11 11:09:28.954365",
            "stdout": "    inet 127.0.0.1/8 scope host lo\n 
   inet 172.19.0.2/16 brd 172.19.255.255 scope global eth0",
            "cmd": "ip a |grep \"inet\"",
            "rc": 0,
            "start": "2019-08-11 11:09:28.622177",
            "stderr": "",
            "delta": "0:00:00.332188",
            "invocation": {
                "module_args": {
                    "creates": null,
                    "executable": null,
                    "_uses_shell": true,
                    "strip_empty_ends": true,
                    "_raw_params": "ip a |grep \"inet\"",
                    "removes": null,
                    "argv": null,
                    "warn": true,
                    "chdir": null,
                    "stdin_add_newline": true,
                    "stdin": null
                }
            },
            "stdout_lines": [
                "    inet 127.0.0.1/8 scope host lo",
                "    inet 172.19.0.2/16 brd 172.19.255.255 scope global eth0"
            ],
            "stderr_lines": [],
            "ansible_facts": {
                "discovered_interpreter_python": "/usr/bin/python"
            },
            "_ansible_no_log": false
        },
        "172.19.0.3": {
            "changed": true,
            "end": "2019-08-11 11:09:28.952018",
            "stdout": "    inet 127.0.0.1/8 scope host lo\n    inet 172.19.0.3/16 brd 172.19.255.255 scope global eth0",
            "cmd": "ip a |grep \"inet\"",
            "rc": 0,
            "start": "2019-08-11 11:09:28.643490",
            "stderr": "",
            "delta": "0:00:00.308528",
            "invocation": {
                "module_args": {
                    "creates": null,
                    "executable": null,
                    "_uses_shell": true,
                    "strip_empty_ends": true,
                    "_raw_params": "ip a |grep \"inet\"",
                    "removes": null,
                    "argv": null,
                    "warn": true,
                    "chdir": null,
                    "stdin_add_newline": true,
                    "stdin": null
                }
            },
            "stdout_lines": [
                "    inet 127.0.0.1/8 scope host lo",
                "    inet 172.19.0.3/16 brd 172.19.255.255 scope global eth0"
            ],
            "stderr_lines": [],
            "ansible_facts": {
                "discovered_interpreter_python": "/usr/bin/python"
            },
            "_ansible_no_log": false
        }
    },
    "failed": {},
    "unreachable": {}
}
```

## 使用范例三 执行 playbook:

> #### 沿用上例继续执行一个 playbook

#### playbook `/root/test.yml` 内容



```bash
---
- name: a test playbook
  hosts: [nginx]
  gather_facts: no
  tasks:
  - shell: ip a |grep inet
...
```

#### 代码



```bash
ansible2 = MyAnsiable2(inventory='/etc/ansible/hosts', connection='smart')

传入playbooks 的参数，需要是一个列表的数据类型，这里是使用的相对路径
相对路径是相对于执行脚本的当前用户的家目录
ansible2.playbook(playbooks=['test.yml'])


ansible2.get_result()
```

输出内容



```csharp
[root@635bbe85037c ~]# /usr/local/bin/python3 /root/code/ansible2api2.8.py
{
    "success": {
        "172.19.0.2": {
            "changed": true,
            "end": "2019-08-11 11:12:34.563817",
            "stdout": "    inet 127.0.0.1/8 scope host lo\n    inet 172.19.0.2/16 brd 172.19.255.255 scope global eth0",
            "cmd": "ip a |grep inet",
            "rc": 0,
            "start": "2019-08-11 11:12:34.269078",
            "stderr": "",
            "delta": "0:00:00.294739",
            "invocation": {
                "module_args": {
                    "creates": null,
                    "executable": null,
                    "_uses_shell": true,
                    "strip_empty_ends": true,
                    "_raw_params": "ip a |grep inet",
                    "removes": null,
                    "argv": null,
                    "warn": true,
                    "chdir": null,
                    "stdin_add_newline": true,
                    "stdin": null
                }
            },
            "stdout_lines": [
                "    inet 127.0.0.1/8 scope host lo",
                "    inet 172.19.0.2/16 brd 172.19.255.255 scope global eth0"
            ],
            "stderr_lines": [],
            "ansible_facts": {
                "discovered_interpreter_python": "/usr/bin/python"
            },
            "_ansible_no_log": false
        },
        "172.19.0.3": {
            "changed": true,
            "end": "2019-08-11 11:12:34.610433",
            "stdout": "    inet 127.0.0.1/8 scope host lo\n    inet 172.19.0.3/16 brd 172.19.255.255 scope global eth0",
            "cmd": "ip a |grep inet",
            "rc": 0,
            "start": "2019-08-11 11:12:34.290794",
            "stderr": "",
            "delta": "0:00:00.319639",
            "invocation": {
                "module_args": {
                    "creates": null,
                    "executable": null,
                    "_uses_shell": true,
                    "strip_empty_ends": true,
                    "_raw_params": "ip a |grep inet",
                    "removes": null,
                    "argv": null,
                    "warn": true,
                    "chdir": null,
                    "stdin_add_newline": true,
                    "stdin": null
                }
            },
            "stdout_lines": [
                "    inet 127.0.0.1/8 scope host lo",
                "    inet 172.19.0.3/16 brd 172.19.255.255 scope global eth0"
            ],
            "stderr_lines": [],
            "ansible_facts": {
                "discovered_interpreter_python": "/usr/bin/python"
            },
            "_ansible_no_log": false
        }
    },
    "failed": {},
    "unreachable": {}
}
```

## 使用范例四 playbook 中提权



```bash
---
- name: a test playbook
  hosts: [nginx]
  gather_facts: no
  remote_user: shark
  become: True
  become_method: sudo
  vars:
    ansible_become_password: upsa
  tasks:
  - shell: id
...
```

## 使用范例五 执行异步任务

**代码**



```python
ansible2 = MyAnsiable2(connection='smart')

ansible2.run(module="shell", args="sleep 15;hostname -i", task_time=15)

ansible2.get_result()
```

**执行输出结果**



```shell
(ansible) [root@iZ2zecj761el8gvy7p9y2kZ ~]# python3 myansibleapi2.py
{
    "success": {
        "localhost": {
            "started": 1,
            "finished": 0,
            "results_file": "/root/.ansible_async/118567079981.4210",
            "ansible_job_id": "118567079981.4210",
            "changed": true,
            "ansible_facts": {
                "discovered_interpreter_python": "/usr/bin/python"
            },
            "_ansible_no_log": false
        }
    },
    "failed": {},
    "unreachable": {}
}
```

`ansible_job_id` 是这个任务返回的任务 ID

获取任务结果



```python
(ansible) [root@iZ2zecj761el8gvy7p9y2kZ ~]# ansible 127.0.0.1 -i 127.0.0.1, -m async_status -a "jid=118567079981.4210"
127.0.0.1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "ansible_job_id": "118567079981.4210",
    "changed": true,
    "cmd": "sleep 15;hostname -i",
    "delta": "0:00:15.034476",
    "end": "2020-06-11 12:06:57.723919",
    "finished": 1,
    "rc": 0,
    "start": "2020-06-11 12:06:42.689443",
    "stderr": "",
    "stderr_lines": [],
    "stdout": "127.0.0.1 172.17.125.171 ::1",
    "stdout_lines": [
        "127.0.0.1 172.17.125.171 ::1"
    ]
}
```

## 使用范例六 在 playbook 中执行异步任务

playbook 中执行异步任务可以说和 api 的开发几乎没有关系, 是在 playbook 中实现的。



```yaml
---
- name: a test playbook
  hosts: [localhost]
  gather_facts: no
  tasks:
  - shell: sleep 10;hostname -i
    async: 10  # 异步
    poll: 0
    register: job
  - name: show  job id
    debug:
      msg: "Job id is {{ job }}"
...
```

执行和获取结果参考前面的范例三

# 参考

Inventory 部分参数说明



```python
ansible_ssh_host
      将要连接的远程主机名.与你想要设定的主机的别名不同的话,可通过此变量设置.

ansible_ssh_port
      ssh端口号.如果不是默认的端口号,通过此变量设置.

ansible_ssh_user
      默认的 ssh 用户名

ansible_ssh_pass
      ssh 密码(这种方式并不安全,我们强烈建议使用 --ask-pass 或 SSH 密钥)

ansible_sudo_pass
      sudo 密码(这种方式并不安全,我们强烈建议使用 --ask-sudo-pass)

ansible_sudo_exe (new in version 1.8)
      sudo 命令路径(适用于1.8及以上版本)

ansible_connection
      与主机的连接类型.比如:local, ssh 或者 paramiko. Ansible 1.2 以前默认使用 paramiko.1.2 以后默认使用 'smart','smart' 方式会根据是否支持 ControlPersist, 来判断'ssh' 方式是否可行.

ansible_ssh_private_key_file
      ssh 使用的私钥文件.适用于有多个密钥,而你不想使用 SSH 代理的情况.

ansible_shell_type
      目标系统的shell类型.默认情况下,命令的执行使用 'sh' 语法,可设置为 'csh' 或 'zsh'.

ansible_python_interpreter
      目标主机的 python 路径.适用于的情况: 系统中有多个 Python, 或者命令路径不是"/usr/bin/python",比如  \*BSD, 或者 /usr/bin/python
      不是 2.X 版本的 Python.我们不使用 "/usr/bin/env" 机制,因为这要求远程用户的路径设置正确,且要求 "python" 可执行程序名不可为 python以外的名字(实际有可能名为python26).

      与 ansible_python_interpreter 的工作方式相同,可设定如 ruby 或 perl 的路径....
```

# Ansible API: Play中variable（变量）的使用

```python
play_source =  dict(
        name = "Ansible Play",
        hosts = '192.168.1.1',
        gather_facts = 'no',
        vars=dict(ccc='play variable ccc',ddd='play variable ddd'),
        tasks = [
             dict(block = [
             			dict(action=dict(module='script', args='/test.sh "{{ddd}}" {{aaa}} "{{fff}}"'),register='x', vars=dict(eee='task variable eee',fff='task variable fff')),
                                dict(action=dict(module='command', args='uname'))
                            ], 
                  when = 'eee is defined', 
                  rescue = [dict(action=dict(module='debug' ,args=dict(msg="error occur")))], 
                  always = [dict(action=dict(module='shell' ,args='df'))],
                  vars = dict(aaa='"block variable aaa"',bbb='block variable bbb')
                 ),
             dict(action=dict(module='script', args='/test.sh "{{ccc}}" 123 {{x.rc}}'))
          ]
    )
play = Play().load(play_source, variable_manager=variable_manager, loader=loader)

tqm = None
try:
    tqm = TaskQueueManager(
              inventory=inventory,
              variable_manager=variable_manager,
              loader=loader,
              options=options,
              passwords=None,
              stdout_callback=results_callback,  
          )
    result = tqm.run(play) 
finally:
    if tqm is not None:
        tqm.cleanup()
    shutil.rmtree(C.DEFAULT_LOCAL_TMP, True)

```

```python
import json
import shutil
from ansible.module_utils.common.collections import ImmutableDict
from ansible.parsing.dataloader import DataLoader
from ansible.vars.manager import VariableManager
from ansible.inventory.manager import InventoryManager
from ansible.playbook.play import Play
from ansible.executor.task_queue_manager import TaskQueueManager
from ansible.plugins.callback import CallbackBase
from ansible import context
import ansible.constants as C
 
class ResultCallback(CallbackBase):
    """A sample callback plugin used for performing an action as results come in
    If you want to collect all results into a single object for processing at
    the end of the execution, look into utilizing the ``json`` callback plugin
    or writing your own custom callback plugin
    """
    def v2_runner_on_ok(self, result, **kwargs):
        """Print a json representation of the result
        This method could store the result in an instance attribute for retrieval later
        """
        host = result._host
        print(json.dumps({host.name: result._result}, indent=4))
 
# since the API is constructed for CLI it expects certain options to always be set in the context object
context.CLIARGS = ImmutableDict(connection='ssh', module_path=['/to/mymodules'], forks=10, become=None,
                                become_method=None, become_user=None, check=False, diff=False)
 
# initialize needed objects
loader = DataLoader() # Takes care of finding and reading yaml, json and ini files
passwords = dict(vault_pass='secret')
 
# Instantiate our ResultCallback for handling results as they come in. Ansible expects this to be one of its main display outlets
results_callback = ResultCallback()
 
# create inventory, use path to host config file as source or hosts in a comma separated string
inventory = InventoryManager(loader=loader, sources='192.168.1.2,192.168.1.3,')
 
# variable manager takes care of merging all the different sources to give you a unified view of variables available in each context
variable_manager = VariableManager(loader=loader, inventory=inventory)
variable_manager.extra_vars = {'ansible_ssh_user': 'test', 'ansible_ssh_pass': 'test', 'StrictHostKeyChecking': 'no'}
 
# create data structure that represents our play, including tasks, this is basically what our YAML loader does internally.
play_source =  dict(
        name = "Ansible Play",
        hosts = 'all',
        gather_facts = 'no',
        tasks = [
            dict(action=dict(module='shell', args='ls'), register='shell_out'),
            dict(action=dict(module='debug', args=dict(msg='{{shell_out.stdout}}')))
         ]
    )
 
# Create play object, playbook objects use .load instead of init or new methods,
# this will also automatically create the task objects from the info provided in play_source
play = Play().load(play_source, variable_manager=variable_manager, loader=loader)
 
# Run it - instantiate task queue manager, which takes care of forking and setting up all objects to iterate over host list and tasks
tqm = None
try:
    tqm = TaskQueueManager(
              inventory=inventory,
              variable_manager=variable_manager,
              loader=loader,
              passwords=passwords,
              stdout_callback=results_callback,  # Use our custom callback instead of the ``default`` callback plugin, which prints to stdout
          )
    result = tqm.run(play) # most interesting data for a play is actually sent to the callback's methods
finally:
    # we always need to cleanup child procs and the structures we use to communicate with them
    if tqm is not None:
        tqm.cleanup()
 
    # Remove ansible tmpdir
    shutil.rmtree(C.DEFAULT_LOCAL_TMP, True)

```

