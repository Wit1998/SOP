# NFS网络文件系统

```bash
安装nfs-untils nfs-client
挂载要共享的分区（不挂载默认共享根目录）
echo "/nfs-share 192.168.199.0/24(rw)" >> /etc/exports
systemctl restart nfs-server
systemctl restart nfs-server
systemctl stop firewalld
setenforce 0
exportfs
mkdir /nfs-mountpoint
mount -t nfs 192.168.199.0/24:/nfs-share /nfs0-mountpoint
#此时远端客户端已经可以挂载目录了，但要实现读写，服务端必须给共享目录提供更高的权限
chmod o+w /nfs-share/
```

## vdo

