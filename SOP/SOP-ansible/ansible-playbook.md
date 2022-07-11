# ansible-playbook



anisble do hoc命令是单行，一个简单的任务，运行一次。ansible真正强大的地方是使用ansible的playbook重复 运行多次复杂的任务。一个play是是一组有序的任务，该paly对应着在inventory被选择的主机。一个playbook是一个包含若干个paly的text文本文件。

在palybook中，你可以用易懂和立即能运行的格式将tasks 保存在play中。tasks本身由于书写方式的原因，就是一个一部一部部署你的架构或者应用的文档。

```bash
--limit(-l) 限制inventory的主机

#注意tab和空格的区别
tab会转义成^
# -v -vv -vvv
-v 显示gathering tasks的细节
-vv 显示ansible.cfg、ansible版本、playbook
-vvv 把连接信息和被管理主机列出来

# 都无法判断逻辑错误
#检查yml文件语法错误
ansible-palybook --syntax-check xxx.yml
#模拟执行dryrun(只用于检测能否运行)
ansible-palybook -C xxx.yml
```

## 举例

```bash
- name: xxx
  hosts: 
  tasks:
    - name: xxx
      user:
        name: xxx
        uid: xxx
        state: present
        
      
```

## yaml strings

```bash
#字符不使用引号、使用单引号、使用引号都一样
hello world
'hello world'
"hello world"
# 使用'|'表示换行操作,输出的每一行都会换行
tasks:
  - name: use debug
    debug:
    msg: |
      hello
      world
结果为： "hello\nworld"
# 使用'>'表示一行操作,输出的每一行都是一行
tasks:
  - name: use debug
    debug:
    msg: |
      hello
      world
结果为： "helloworld"
```

## ansible-playbook通配符运算符

```yaml
# 任意字符*
hosts: '*.lab.example.com'

# 与字符& 表示在lab和datacenter1两个主机组都存在的主机
hosts: lab,&datacenter1
hosts: &lab,datacenter1

# 或字符! 
hosts: all,!lab
```

## ansbile forks

多线程执行脚本

```
ansible-config dump|grep -i fork
ansible-config list|grep -i forks
# 修改ansible.cfg可以修改参数
```

## ansible serial

在一个task里面执行多少个主机。(滚动更新)

如同时更新时，需要先停掉服务，如果全部同时停服务，则会导致服务停机。使用serial就可以避免这个问题

```yaml
# 每次执行一个被管理主机
---
- name: xxx
  hosts: xxx
  serial: 1
  tasks:
    - name: xx
```

## ansible 静态导入和动态包含

静态导入play不会认为本yml文件有task，真正的task在import_tasks的地方

动态包含play就直接认为是本yml文件的task

```
---
 - name: xx
   hosts: xx
   tasks:
     - import_tasks: xx.yml


---
 - name: xx
   hosts: xx
   tasks:
     - include_tasks: xxx.yml
```

## ansible troubleshooting

```
# 打印日志
[defaults]
log_path=xxx

# 添加细节
```

