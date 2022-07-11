# ansible.cfg

### 说明：

ansible需要知道如何和被管理主机建立连接。一个最常见的方式是改变配置文件来控制使用什么方式什么用 户来管理被管理主机。这些信息包含： 

- ①使用哪个inventory文件 
- ②使用什么连接协议（默认是ssh），使用什么端口 
- ③使用什么用户 
- ④如果一个用户没有特殊权限是否需要做提权操作，例如使用sudo提升到root 
- ⑤是否提示输入ssh密码或者输入root的密码来获得提权

```ini
defaults部分设置
[defaults]
# 指定inventory文件
inventory=./inventory
# 指定默认用户
remote_user=devops
# 指定用户ssh连接的时候是否提示输入密码
# 如果服务器没有公钥或私钥，可以打开这个选项，让服务器生成密钥
ask_pass=false/true
#这是设置是否检查SSH主机的密钥。可以设置为True或False
host_key_checking=False

#privilege_escalation部分设置
[privilege_escalation] 
# 表示是否需要提权
become=true/false
# 表示提权方式
become_method=sudo
# 表示提权到什么用户
become_user=root
# 表示提权是否需要密码 （如果设置不需要，要在目标服务器上面设置用户免密）
become_ask_pass=false/true

cat >> /etc/sudoers << END
student		ALL=(ALL:ALL)	 NOPASSWORD: ALL
END

```



```bash
#即使inventory没有local，也会出现本机
ansible -i inventory local --list-hosts
```

