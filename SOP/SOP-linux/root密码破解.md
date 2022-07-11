# root密码破解

进入维护系统(initramfs)

```bash
#重新挂载sysroot
mount -o rw.remount /sysroot
#将当前文件系统的根作为root的根
chroot /sysroot
vim /etc/shadow
passwd
#或者
echo 1 |passwd --stdin root
#重新更改selinux标签
touch /.autorelabel
```

