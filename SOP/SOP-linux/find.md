# find

find命令能搜索任何你想搜索的文件，格式：
find 查找范围 查找参数

```bash
1.根据名称查找文件，名称支持正则表达式和通配符
[root@server ~]# find / -name default
[root@server ~]# find /usr/ -name default
[root@server ~]# find / -name default 2>>/dev/null
2.根据拥有人搜索
[root@server ~]# find / -user apache 2>>/dev/null | xargs ls -ld
3.根据大小搜索
[root@server ~]# find / -size +100M 2>>/dev/null | xargs ls -lh

[root@server ~]# find / -size +100M -size -300M 2>>/dev/null | xargs ls -lh


# 题目：找到拥有人为aaa的文件，将其拷贝到某个目录
find / -user aaa 2>>/dev/null -exec c[ -a {} /root \:
```

