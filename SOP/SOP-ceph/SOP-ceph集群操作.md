# ceph集群操作

## 删除镜像

```bash
#解挂
sudo umount 挂载目录
#unmap
sudo rbd unmap 镜像名 --name client.admin
#删除镜像 &是作为后台程序跑
sudo rbd remove 镜像名 &
#查看镜像是否已删除
sudo rbd list 镜像名称
```

## 删除目录

```bash
#查看mon的关于含有delete的配置
sudo ceph config show-with-defaults mon.ceph1143 | grep delete
#将允许删除pool的环境变量设置为true
sudo ceph tell mon.* injectargs --mon_allow_pool_delete true
#从平衡池中抽出数据池
sudo ceph balancer pool rm ceph156-data-ecp6_1
#
sudo ceph osd tier rm-overlay 数据池
sudo ceph osd tier rm 数据池名 缓存池名
 
#删除缓存池
sudo ceph osd pool rm area1-pool-cache area1-pool-cache --yes-i-really-really-mean-it
#删除数据池
sudo ceph osd pool rm area1-data-ecp22_2 area1-data-ecp22_2 --yes-i-really-really-mean-it
#如果元数据池是单独使用的，也要删除元数据池
sudo ceph osd pool rm area1-pool-meta area1-pool-meta --yes-i-really-really-mean-it
```

