# ansible ad hoc



```bash
ansible-doc [modules]
ansible-doc -l
--become/-b 表示提权 
```



## 常用模块

```bash
ansible -m的模块参数
#ping (表示被管理主机上使用ping命令来ping控制节点)

#shell 

#command 执行命令模块（如果不加-m参数，默认就是command）

#unarchive 解压模块
	-a "src=/tmp/install/zabbix-3.0.4.tar.gz dest=/tmp/ mode=0755 copy=yes" 

```

## FILE modules

```bash
#copy 拷贝模块
	-a "src=storage dest=/home/xg owner=xg group=xg mode=0755 backup=no"
	#输入content的内容到dest文件
	-a "content=storage dest=/home/xg owner=xg group=xg mode=0755 backup=no"
#file

#lineinfile
#synchronize

```

## software package modules

```bash
#package
#yum
#apt
#dnf
#gem
#pip
```

## system modules

```bash
#firewalld
#reboot
#service
#user 用户模块
ansible -u 指定用户
	ansible intranetweb -m user -a 'name=fs uid=5001 state=present' 
	ansible intranetweb -m user -a 'name=fs uid=5001 state=absent' 
```

## NET tools modules

```bash
#get_url 下载模块
	-a "url=http://www.guojinbao.com dest=/tmp/guojinbao mode=0440 force=yes"
#nmcli
#uri
```



### shell和command的区别

- command 模块命令将不会使用 shell 执行. 因此, 像 $HOME 这样的变量是不可用的。还有像<, >, |, ;, &都将不可用。
- shell 模块通过shell程序执行， 默认是/bin/sh, <, >, |, ;, & 可用。但这样有潜在的 shell 注入风险
- command 模块更安全，因为他不受用户环境的影响。 也很大的避免了潜在的 shell 注入风险.
- 如果你需要安全的使用带有变量的 shell 模块， 使用`{{ var | quote }}` 代替 `{{ var }}` , 确保输入不包含分号或者流式操作.