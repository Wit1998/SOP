# SELinux

DAC（自主访问控制）：

就是以三种用户（user、group、other）来管理权限，root有所有权限。缺陷是以不同身份用户启动进程时，进程的访问权限受用户显示，如使用root启动，就会有所有权限

MAC（强制访问控制）：
系统的Policy决定了主体能操作访问哪些客体
即便是ROOT进程，系统Policy配置了你能做什么，你只能做什么，在下MAC模式，ROOT进程和普通
进程是无区别对待的。

```bash
# 查看标记
ps -auxZ |grep httpd
ls -Z

#不同位置创建的文件，权限不同


#基本操作


semanage fcontext 
restorecon -Rv .

touch /.autorelabel


semanage port -a -t http_port_t -p tcp 12345


#开启和关闭selinux
三种状态：Enforcing\Permissive\disabled(打开\容忍\关闭)

getenforce
setenforce 0
setenforce 1

#日志log
/var/log/audit/audit.log
#配置文件
/etc/selinux/config

#boolean
semanage boolean -l |grep samba

setsebool -P samba create_home_dirs on
setsebool -P samba create_home_dirs off
```