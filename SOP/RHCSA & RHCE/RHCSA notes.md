# RHCSA

```bash
# 评分
./grade.sh rhcsa 50 test

#重新练习
#在物理机shell执行
rhcsa8.sh
```



## 进入单用户模式修改root密码

```bash
# 重启机器，选择页面按e
# 在linux那一行后面添加console=ttyo rd.break
# ctrl + x 进入单用户模式，修改linux引导的参数
# 将sysroot根目录以读写形式重新挂载。 -o表示opition，rw表示读写，remount表示重新挂载。
mount -orw,remount  /sysroot 
chroot /systroot 进入root用户
id
#  修改密码
echo migwhisk | passwd --stdin root
# 创建隐藏文件
touch /.autorelabel 
exit
exit
```

## 进入系统更改网络/主机名信息

```bash
# 修改网络
#方法1 直接修改现有的配置文件
nmcli con modify "Wired connection 1" ipv4.addresses 172.24.10.150/24 ipv4.gateway 172.24.10.100 ipv4.dns 172.24.10.250 ipv4.method manual
# 激活
nmcli con up "Wired connection 1"

# 方法2 直接添加一个新的配置文件
nmcli con add con-name static ifname eth0 type ethernet ipv4.addresses 172.24.10.150/24 ipv4.gateway 172.24.10.100 ipv4.dns 172.24.10.250 ipv4.method manual
# 激活
nmcli con up static
# 验证
ip a show eth0
```

### 

```bash
# 修改主机名
hostnamectl set-hostname system.domain10.example.com
#验证主机名
hostnamectl
cat /etc/hostname
```

## 1.设置软件仓库

```bash
# rhcsa.repo信息
cat > /etc/yum.repos.d/rhcsa.repo << END
[baseos]
name = baseos
enable = yes //表示开启软件仓库
gpgcheck = 0
baseurl = http://xxxxx

[appstream]
name = appstream
enable = yes
gpgcheck = 0 
baseurl = http://xxxx
END

# 验证
yum repolist
```

## 2.配置selinux

```bash
# 把config中的SELINUX=permissive 改成enforcing
sed -i "s/SELINUX=permissive/SELINUX=enforcing/" /etc/selinux/config
# 检查是否更改
cat /etc/selinux/config |grep SELINUX
# 打开selinux
getenforce
setenforce 1
getenforce
#检查标签情况
ls -Z /var/www/html/index.html
# 使用semanage把html目录都打上标签
yum provides semanage
yum instal policycoreutils-python-utils
semanage fcontext -a -t httpd_sys_content "/var/www/html(/.*)?"
# 设置永久的web上下文
restorecon -RvF /var/www/html
# 放行selinux的安全端口
semanage port -a -l http_port_t -p tcp 82
semanage port -l |grep http

#重启httpd
systemctl restart httpd
systemctl enable httpd
# 此时本地已经可以访问web页面
curl localhost:82
# 临时放行端口
firewall-cmd --add-port=82/tcp
# 永久放行
firewall-cmd --add-port=82/tcp --per

# 将所有的放行都设置成永久(可选)
firewall-cmd --runtime-to-permanet

# 检查web
curl system1:82
curl system1:82/file1
```

## 3.创建用户账户

```bash
#名为sysmgrs的组
gorupadd sysmgrs -g 30000
#用户natasha ，作为次要组从属于sysmgrs
useradd -G sysmgrs natasha
#用户harry ，作为次要组从属于sysmgrs
useradd -G sysmgrs harry
# 用户sarah，无权访问系统上的交互式shell且不是sysmgrs的成员
useradd sarah -s /sbin/nologin
# natasha、harry和sarah的密码应当都是123
echo 123 | passwd --stdin natasha
echo 123 | passwd --stdin harry
echo 123 | passwd --stdin sarah
```

## 4.配置cron计划任务

```bash
# 以natasha身份运行
crontab -e -u natasha
# 每五分钟运行logger "EX200 in progress"(分时日月周)
*/5 * * * * logger "EX200 in progress"
# 在14:23分执行echo enjia
23 14 * * * /bin/echo enjia
#查看cron计划任务
crontab -l -u natasha

#检查cron服务
systemctl is-enabled crond
systemctl is-active crond
```

## 5.创建共享目录

```bash
#创建目录
mkdir /home/managers
# 该目录属于sysmgrs组
chgrp sysmgrs /home/managers
# 目录组成员拥有rwx权限，other没有权限
chmod g=rwx,o=--- /home/managers/
# 将/home/managers中创建的文件自动将组所有权设置到sysmgrs组
chmod g+s /home/managers
# 检查
ls -ld /home/managers/
```

## 6.配置NTP

配置系统，使其成为host.domain10.example.com的NTP客户端

```bash
# 在chrony.conf添加域名
vim /etc/chrony.conf
server host.domain10.example.com iburst
# 方法二
echo "server host.domain10.example.com iburst" |tee -a /etc/chrony.conf

# 重启服务并开启开机启动
systemctl restart chronyd
systemctl enable chronyd

# 验证
systemctl state chronyd
chronyc sources
chronyc -n sources
ping -c1 host.domain10.example.com
```

## 7.配置autofs

```bash
# 安装nfs客户端
yum -y install nfs-utils
yum -y install autofs
vim /etc/auto.master
# 添加家目录的自动挂载
/rhel	/etc/auto.user1

# 在添加配置,把远端的目录以读写的形式挂载下来
cat > /etc/auto.user1 << EOF
user1	-rw		host.domain10.example.com:rhel/user1
EOF

# 检查配置
grep rhel /etc/auto.master
cat /etc/auto.user1
ls /rhel

# 设置开机启动并立即启动
systemctl enable autofs --now
#检查启动情况
systemctl status autofs
# 重启配置
systemctl restart autofs
ls /rhel/ -d

# 验证
grep user1 /etc/passwd
df -hT /rhel/user1
su - user1
pwd
touch test
ls test
```

## 8.配置文件权限

```bash
cp /etc/fstab /var/tmp/fstab
chown root:root /var/tmp/fstab
# 所有人都没有写权限
chmod a-x /var/tmp/fstab
# 其他人只有读权限
chmod o=r-- /var/tmp/fatab
# 用户natasha能够读取和写入/var/tmp/fstab
setfacl -m u:natasha:rw /var/tmp/fstab
# 用户harry无法写入或读取/var/tmp/fstab
setfacl -m u:harry:--- /var/tmp/fstab
```

## 9.配置用户账户

```
useradd user2 -u 3388
echo 123 | passwd --stdin user2
```

## 10.查找文件

```bash
mkdir dfiles
# 查找当user3所有的所有文件并将其副本放入/root/dfiles目录
#cp -a 表示复制原有属性; {}表示前面查找的对象
find / -user user3 -exec cp -a {} /root/dfiles/ \;
```

## 11.查找字符串

查找文件/usr/share/rhel.xml中包含字符串re的所有行。将所有副本按原始顺序放在文件/root/files中

```
grep re /usr/share/rhel.xml > /root/files
cat files
```

## 12.创建存档(文件归档)

```bash
# 安装tar
yum provides tar
yum -y install tar
# 使用gzip的方式压缩
tar --help |grep gzip
tar -zcvf books.tar.gz /usr/local

# 安装bzip2
yum provides bzip2
yum -y install 
# 使用bzip2的方式压缩
tar -jcvf books.tar.bz2 /usr/local

# 检查文件类型
file books.tar.*
```

## 13.调整逻辑卷大小

ext4或xfs两种文件系统

将vo的逻辑卷调整到180M，确保文件系统不变。

```bash
# 查看逻辑卷属性
lvs
# 查看卷组属性
vgs
#扩容ext4
lvextend /dev/vg-exam/vo1 -L 180M
#扩容xfs
lvextend /dev/vg-exam/vo2 -L 180M

# 动态调整
# ext4 (指定块设备)
resize2fs /dev/vg-exam/vo1
# xfs (指定挂载点)
xgs_growfs /mnt/vo2

# 验证
lvs
df -hT /mnt/vo1 /mnt/vo2
```

## 14.添加交换分区

```bash
free -m
# 添加新分区并修改分区类型
fdisk /dev/vdb
n
p
2
+567M
t
2
82
w

# 查看新添加的分区
ls /dev/vdb*
partprobe
# 格式化
mkswap /dev/vdb2
# 添加swap信息
vim /etc/fstab
UUID=xxxxx swap swap defaults 0 0
# 使用命令加载一下永久挂载的新添加部分
swapon -a
free -m
```

## 15.创建逻辑卷

```bash
1. 磁盘分区
fdisk /dev/vdb
n
p
3
+1G
w

# 检查
ls /dev/vdb*

2. 创建物理卷
pvcreate /dev/vdb3
# 检查
pvs

3. 创建卷组
vgcreate -s 20M npgroup /dev/vdb3
vgs

4. 创建逻辑卷
# 一个单元20M，45个就是900M
lvcreate -n np -l 45
# 检查
lvs

5. 格式化
mkfs.ext3 /dev/npgroup/np

6. 创建挂载点
mkdir /mnt/np
ls -ld /mnt/np

7. 永久挂载
lsblk
# 添加挂载信息
vim /etc/fstab
/dev/npgroup/np /mnt/np ext3 defaults 0 0
mount -a 
df -hT
```

## 16.创建VDO卷

```bash
1. 安装vdo
yum -y install vdo
2. 创建vdo
ls /dev/vdc*
vdo create --name=vdoname --device=/dev/vdc --vdoLogicalSize=80G 

3. 格式化vdo为xfs文件系统
mkfs.xfs /dev/mapper/vdoname 

4. 文件系统挂载
mkdir /vbark

# 新增挂载点,持续性挂载
vim /etc/fstab
# vdo不能直接使用defaults的参数
# _netdev表示把该设备当作了网络设备上使用的卷，
/dev/mapper/vdoname /vbark xfs defaults,_netdev 1 2

mount -a 
df -hT
```

## 17.配置系统调优

```bash
# 查看当前调优状态
tuned-adm list
#关掉调优
tuned-adm off
# 查看当前最优调优是哪一种类型
tuned-adm recommend
# 调优到最优
tuned-adm profile virtual-guest
# 检查
tuned-adm list
```

## 18.容器



必须使用普通用户elovodo来操作，且不能使用su切换，要使用ssh连接

```bash
# 使用elovode连接server
ssh elovodo@xxx

# 配置容器镜像仓库registry
podman login http://xxxxx:5000

# 拉取镜像
podman pull xxxxx/xxxx

# 将考试要求的文件拷贝到指定目录中作为容器的之就行挂载存储
# 拷贝目录
cp -a /var/log/journal/* /home/elovodo/container_journal/
# 拷贝隐藏目录 
# []里面表示匹配一个字符， !.表示非. , (这样可以避免复制.. 会把上级目录也复制进来)
cp -a /var/log/journal/.[!.]*

# 运行容器
podman images
# 容器持久性挂载
# :Z 可以让容器自动解决SElinux权限问题
podman run -itd -v /home/elovodo/container_journal:/var/log/journal:Z --name container_logserver [image]

# 进入容器检查
podman ps
podman exec -it container_logserver bash

# 查看容器日志文件
ls /var/log/journal/gzytest -a
ls /var/log/journal/.gzytest -a

mkdir ~/.config/systemd/user -p
# 此时文件是空的
ls -R .config/
cd .config/systemd/user/
ls
# 创建普通用户的systemd配置文件
podman generate systemd --new --files --name container_logserver
# 查看生成的systemd文件
cat container-container_logserver.service

# 修改systemd文件
mv container-container_logserver.service container_logserver.service

# 停掉之前启动的容器
podman ps -a
podman stop container_logserver
podman rm container_logserver

# 让普通用户具备自定义自己systemd服务的权限
loginctl enable-linger
# 以user用户重新加载systemd
systemctl --user daemon-reload
# 设置开机启动并立即启动
systemctl --user enable container_logserver.service --now
# 查看服务状态
systemctl --user status container_logserver.service

#检查容器启动情况
podman ps

#必须保证system1重启之后，使用elovodo用户登录之后，看到容器正常启动
```

