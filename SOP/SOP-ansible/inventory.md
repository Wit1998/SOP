# inventory

## 静态inventory

### 说明：

- 如果inventory文件包含同样名字的host和group，那么ansible在列出目标主机的时候会弹出警告，并且 group会被忽略
- ansible和ansible-playbook这两个命令都可以使用“-i”来指定 inventory文件的位置。
- 默认的inventory文件在/etc/ansible/hosts
- Ansible按照如下位置和顺序来查找ansible.cfg文件：
  - 1.ANSIBLE_CONFIG环境变量所指定的文件 
  - 2.当前目录下的ansible.cfg文件 
  - 3.家目录下的的ansible.cfg文件 
  - 4./etc/ansible/ansible.cfg文件

### 写法：

```yaml
#需要配置hosts
#被管理主机使用hosts的hostname
#用"[]"表示主机组，被管理主机可以在多个组中，一个组可以同时有被管理主机和child组
#用"all:"表示所有主机
#用"upgrouped:"表示不在任何组的主机
#用":children"表示嵌套组
#指定范围主机：
	server[01:20]
	192.168.[0:1].[0-255]
```

### 复杂inventory

简介
ansible 官方对如何编写 inventory 文件的文档非常简陋，网上也很难找到相应例子，下面简单说下一些
较复杂写法和特殊技巧

```
#
$ ls ~/inventory
ansible.cfg comet common g1 g2 g3 icfs-mon j1 j2 j5 j6 j7
lotus m4 p1 reseal space1 zindex
#
$ ansible -i ~/inventory all --list-hosts
需要注意的点：
文件的读取顺序按照字母数字排序A-Z，a-z，1-9。
hostname 和 group 也一样按字母数字排序载入（hostname 和 group 的区别下面有）。
载入顺序影响最终 host 的参数值。
载入顺序也影响跨文件 group 使用，后载入的文件能使用前面的 group，反过来不行
hostname 和 group
每个 hostname 对应一个 IP，多个 hostname 组成一个 group，多个 group 也可以组成一个 group，
hostname 和 group 不能重名,且要用下划线的格式。
[group:children] # children
group1#
group2# hostname range group[1:2]
[group1]# group
hostname[1:2]# -> hostname1, hostname2
hostname[3:7:2]# -> hostname3, hostname5, hostname7
[group2]
hostname[8:9]
参数的优先级
以下从低到高
命令行的值 (例如，-u my_user, 不是 variables)
role 默认值 (defined in role/defaults/main.yml)
inventory file or script group vars
inventory group_vars/all
playbook group_vars/all
inventory group_vars/*
playbook group_vars/*
inventory file or script host vars
inventory host_vars/*
playbook host_vars/*
host facts / cached set_facts
play vars
play vars_prompt
play vars_files
role vars (defined in role/vars/main.yml)
block vars (only for tasks in block)
task vars (only for the task)
include_vars
set_facts / registered vars
role (and include_role) params
include params
extra vars (for example,-e "user=my_user")(always win precedence)
小技巧
查看制定 hostname 或 group 的 var 值
ansible -i ~/inventory group -m debug -a "var=hostvars
[inventory_hostname]" | grep $VAR
查看该 group 所有 hostname
ansible -i ~/inventory group --list-hosts
```



## 动态inventory



## 多个inventory文件

不建议

这里的[cloud-east]定义在了其他inventory里面，如果配置文件配置了这些inventory，就可以这样调用

```bash
[cloud-east]

[servers]
test.demo.example.com

[servers:children]
cloud-east
```



