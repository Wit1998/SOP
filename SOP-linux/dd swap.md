# dd  swap

## dd

```bash
1.用dd命令测磁盘的写速度
[root@server ~]# dd if=/dev/zero of=/dev/null
[root@server ~]# dd if=/dev/zero of=/root/file count=1000 bs=1M
[root@server ~]# ls -lh /root/file
-rw-r--r--. 1 root root 1000M May 27 23:43 /root/file

2.用dd命令创建厚置备的虚拟机磁盘文件
kvm虚拟机
[root@server ~]# qemu-img create test 100G
[root@server ~]# ls -lh test
[root@server ~]# dd if=/dev/zero of=/root/test count=102400 bs=1M


```



## swap

```bash
1.创建分区作为交换分区
2. mkswap /dev/xxxx
3. blkid
4. vi /etc/fstab
5. UUID=xxxxx swap swap defaults 0 0
# 使用mount -a 是不生效的
6. swapon/swapoff -a

```

