# ansible变量

http://docs.ansible.com/ansible/playbooks_variables.html 

## 说明：

变量由必须以字母开头的字符串组成，并且只能包含字母，数字和下划线组成

```bash
# cmd(全局) > play
# cmd(全局) > inventory
# play > inventory
```

### 变量范围级别

- 1.Globe scope：从命令行设置变量或Ansible配置文件中设置变量 
- 2.play scope：在play和相关结构中设置变量 
- 3.host scope：通过inventory在主机组或单个主机中设置变量，fact采集变量或者register
- 如果在一个级别上定义了名字相同的两个变量，会选择先定义的那个。在inventory中定义的变量会被 playbook中定义的覆盖，playbook中定义的变量会被命令行定义的变量覆盖。在inventory中主机组变量会被主机变量覆盖

### 定义变量举例

```ini
#play定义
---
name: xxx
hosts: all
vars: 
  xx: xxx
tasks:
  debug:
  name: xxx
  msg: "{{ xx }} is good."
  var: xx
  
  
# file定义
---
name: xxx
hosts: all
vars_files:
  - fs/user.yml
tasks:
  debug:
  name: xxx
  var: xx
  
# inventory 变量定义
[xxx:vars]
xx=xx

# 通过命令行覆盖变量 (-e指令)
ansible-playbook xxx.yml --limit=server1 -e "package=apache"
```

```ini
# vars_files 路径相对路径要优于绝对路径。
因为相对路径在整个playbook脚本改变位置时，不受影响。而绝对路径会受影响
```

### group_vars和host_vars

```bash
#针对主机组（group）设置变量，只需在group_vars目录下面创建同名文件定义即可

#针对主机(host)设置变量，只需在host_vars目录下面创建同名文件定义即可
```

### 矩阵变量

```bash
users:
	fs:
		first_name: shi
		last_name: feng
		home_dir: /home/fs

#结果是
users['fs']['frist_name'] = shi
users['fs']['last_name'] = feng
users['fs']['home_dir'] = /home/fs

#带小数点的如 users.fs.first_name 都可以写成users['fs']['frist_name']这种形式
```

### register变量

管理员可以通过register状态来抓取命令的输出。输出将会被保存在变量中，并且可以后续排障使用。

可以记录状态，用作判断之类

```yaml
---
- name: install httpd package
  hosts: servera
  tasks:
  	- name: install the package
   	  yum:
   	  	name: httpd
   	  	state: latest
   	  register: install _result
    - debug: var=install_result
    
    
#返回的结果为:执行 yum install httpd:latest 之后应该打印的返回值 
```



## facts变量

从被管理主机上可以搜集到facts包含：
1.主机名
2.内核版本
3.网卡
4.地址
5.操作系统版本
6.环境变量
7.CPU的数量 
8.可用内存或者空闲内容
9.可用磁盘空间

### 常见facts

| fact                                      | variable                               |
| ----------------------------------------- | -------------------------------------- |
| short hostaname                           | ansible_hostname                       |
| FQDN                                      | ansible_fqdn                           |
| Main ipv4 address                         | anisble_default_ipv4.address           |
| a list of the names of network interfaces | ansible_interfaces                     |
| Main disk first partition size            | ansible_devices.vda.patition.vda1.size |
| a list of DNS servers                     | ansible_dns.nameservers                |
| version of the currently running kernel   | ansible_kernel                         |

### 采集fasts

```yaml
- name: test facts
  hosts: all
  tasks:
   - name： output various ansible facts
     debug:
       msg: " {{ ansible_fqdn }} 's current free mem is {{ ansible_memfree_mb }}" 
       #支持变量矩阵
       vars: ansible_date_time['date']
```



### 取消采集fasts  gather_facts: no

```yaml

- name: test facts
  hosts: all
  gather_facts: no
  tasks:
```

### fact过滤

```yaml
# 这里的ansible_enp161s0f0就是fact信息
ansible -i ~/workers/inventory galaxy3816 -m setup -a 'filter=ansible_enp161s0f0'
```

### 自定义fact

有些fact变量没有值，可以自定义fact变量。

```yaml 
# 一般定义在被管理主机的/etc/ansible/facts.d/xxx.fact

#声明
cat /etc/ansible/facts.d/a.fact
[packages]
web_package = httpd
db_package = mariadb-server
[users]
user1 = fs

#调用
"{{ ansible_local.a.packages.web_package }}"
```

## magic变量

### 常用：

- group_names
- groups
- inventory_hostname

```
hostvars 包含被管理主机的变量，并且可以用于获取其它主机变量的值。
	ansible -i ~/workers/inventory galaxy3811 -m debug -a 'var=hostvars["hostname"]'
```

