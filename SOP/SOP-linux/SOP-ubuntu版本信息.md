# ubuntu版本信息

```
# 通过/proc文件系统查看。 proc是系统映像
cat /proc/version
# gcc可以查看出版本
which gcc
gcc -v
# uname 也是查看/proc这个文件系统
#查看系统内核版本号及系统名称
uname -a
#显示linux的内核版本和系统是多少位的
uname --m
getconf LONG_BIT
# LSB(Linux Standard Base)
lsb_release -a
uname --s  显示内核名字
uname --r 显示内核版本
uname --n  显示网络主机名
uname --p 显示cpu
```

