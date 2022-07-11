# 文件系统（File System）

标准分区：物理硬件disk——>固件driver——>内核kernvl——> 分区partition——>文件系统fs

## 分区（partition）：

sector=512Byte

TiB=1024GiB=1024MiB=1024KiB=1024Byte 

### MBR：

主引导记录（master boot record,MBR）位于硬盘的第⼀物理扇区。由于历史原因，硬盘的⼀个扇区⼤⼩是512字节，包含最多446字节的启动代码、4个硬盘分区表项（每个表项16字节，共64字节）、2个签名字节（0x55,0xAA）。

- mbr单分区⼤⼩不超过2T。
- mbr主分区加扩展分区的数量最多等于4。
- mbr的初始化当想建⽴第四个主分区的时候，会将剩下的所有空间划分成扩展分区，然后在扩展分区⾥⾯划出⼀部分作为逻辑分区。



### GPT

gpt的初始化最多可以达到1024个主分区+扩展分区。GPT单分区最⼤分区
18EB->18,874,368T

## linux硬盘文件分区：

#通过命令⾏⽅式对磁盘进⾏分区（两种⽅式，第⼀种就是MBR，第⼆种就是GPT）

如果你采⽤MBR的⽅式进⾏分区就使⽤fdisk命令
如果你采⽤GPT的⽅式进⾏分区就使⽤gdisk命令
还有其他分区的命令parted

```bash
Generic
d delete a partition（删除⼀个分区）
F list free unpartitioned space
l list known partition types（列出当前⽀持的分区种类）
n add a new partition(添加⼀个新分区)
p print the partition table（列出当前状态的所有分区）
t change a partition type(修改分区类型，没有实际意义)
v verify the partition table
i print information about a partition
```

```bash
#创建扩展分区后，
后续创建的分区都是逻辑分区，虽然名字上和差别不大，是从/dev/sda5开始（因为前面的全部都分配给主区了），但是逻辑分区的引导是有扩展分区引导的，而非主引导
```

```bash
#重定向分区
fdisk /dev/sda < test &>> /dev/null
cat test 
n
p
1

+1G
n
w

```

## 格式化

### block节点

```bash
#在Linux操作系统中格式化文件系统的原理是什么呢？
#分区的最小单位是sector（扇区），一个扇区是512B
#假如分区可以直接使用，那么存储一个20MB的文件需要多少个扇区呢？
#答案是40960，也就是说存储一个20MB的文件需要40960个扇区。
#那么读取这个20MB的文件就需要读40960次。效率太低了。
#上面只是我们不能直接使用分区的一个原因。还有其他原因，请自行了解。
#当我们的分区进行格式化之后，会将磁盘的读和写的粒度放大。
#格式化的详细操作
1.会立刻分配一部分空间作为inode节点空间
2.刨除inode节点占用的空间，剩下的空间用作划分block，block包含了多个扇区，1个block等于2^n个扇区，如果n等于0，那么一个block就等于1个扇区的大小，如果n=1，那么block就等于2个扇区的大小，如果n=2，那么block就等于4个扇区的大小。单个block越大，就意味着粒度越大。划分block的目的是要将文件以block的数量来存放.
block越大对磁盘的读取效果就越好。block大了，一次性读取的空间变多了，这样你的读效率就会提升。block并不是越大越好，如果block太大，但是你存放的文件特征是小文件，那么会造成大量磁盘的浪费。所以格式化文件系统的时候block的大小要选对。
```

```bash
mkfs.ext4 /dev/xxx
mkfs.xfs /dev/xxx
```

### inode节点

文件的属性就是文件的元数据。描述文件的部分，都指的是文件的元数据。

比如文件的大小，文件的权限，文件的拥有者，文件的所属组，文件的名字，文件的时间，文件的selinux安全上下文。

```bash
block是用来存放文件内容，inode节点是用来存放文件元数据的。
[root@server ~]# cat /etc/hosts （查看文件的内容） 存放在block里面
127.0.0.1 localhost localhost.localdomain localhost4
[root@server ~]# ls -l /etc/hosts（查看文件属性） 存放在inode里面。
-rw-r--r--. 1 root root 158 Sep 10 2018 /etc/hosts

# inode节点除了记录了我们文件的元数据之外，还记录了该文件的block所在位置。

# 每个文件都有一个inode节点。如果一个文件系统的inode节点数量使用完了。
```

## mount

```bash
cat /etc/fstab
#第一列是文件系统所在的分区路径
#第二列是文件系统的挂载点
#第三列是文件系统的类型
#第四列是挂载参数
#第五列和第六列分别写0，只有在一些特殊的文件系统，后两列才需要改成非0
#/dev/sda5 /mount-point1 ext2 defaults 0 0

#盘符分区会有唯一的uuid，也可使用uuid挂载
ls -l /dev/disk/by-uuid/
lrwxrwxrwx. 1 root root 10 Jun  6 05:55 abd835ed-e7b2-4a71-aef2-28b30a555a0c -> ../../dm-1
lrwxrwxrwx. 1 root root 10 Jun  6 05:55 be1df4cb-5bf5-4d90-bf18-f931930b3198 -> ../../dm-0
lrwxrwxrwx. 1 root root 10 Jun 12 00:05 e2aafd9a-d4fd-410a-8d22-b136316c8913 -> ../../sda1
```

# 逻辑卷（LV）

pv(physical volume)物理卷
vg(volume group)卷组
lv(logical volume)逻辑卷

逻辑分区：物理硬件——>固件——>内核——> 分区——>PV（物理卷）——>VG（卷组）——>LV（逻辑卷）——>文件系统FS

区别与标准分区

```bash
#pv的创建，pv的删除（向pv里面加入新的块设备就相当于扩容pv，从pv里面删除块设备，就相当于缩容pv）
#vg的创建，vg的删除，vg的扩容，vg的缩容（不包含）
#lv的创建，lv的删除，lv的扩容，lv的缩容（不包含）

#创建物理卷
#先分区
[root@server ~]# fdisk /dev/sdb -l
[root@server ~]# pvcreate /dev/sdb{1..2}
[root@server ~]# pvs
[root@server ~]# pvdisplay /dev/sdb1
#pv可以直接指定一块硬盘
[root@server ~]# pvcreate /dev/sdc


#卷组的创建
[root@server ~]# vgcreate vg1 /dev/sdb1
[root@server ~]# vgs
#vg的删除
[root@server ~]# vgremove vg1
#vg的扩容
[root@server ~]# vgextend vg2 /dev/sdb2

[root@server ~]# vgcreate vg2 /dev/sdb1 -s 8M
[root@server ~]# vgdisplay vg2


#逻辑卷的创建
-n参数表示逻辑卷的名字
-L参数表示逻辑卷的大小 300M,2G
vg2就表示使用卷组vg2创建该逻辑卷
[root@server ~]# lvcreate -n lv2 -L 800M vg2
#TS:Volume group "vg2" has insufficient free space (89 extents): 100 required.
#卷组空间不足，需扩容卷组
[root@server ~]# vgextend vg2 /dev/sdb2
[root@server ~]# vgs
[root@server ~]# lvcreate -n lv2 -L 800M vg2
[root@server ~]# lvs

[root@server ~]# ls -l /dev/dm-3 /dev/dm-4
brw-rw----. 1 root disk 253, 3 May 27 22:51 /dev/dm-3
brw-rw----. 1 root disk 253, 4 May 27 22:54 /dev/dm-4
#dm设备就表示逻辑卷的本尊。dm全拼是device mapper
[root@server ~]# ls /dev/mapper/ -l
[root@server ~]# ls -l /dev/disk/by-uuid/

#逻辑卷的扩容
[root@server ~]# lvcreate -n lv1-ext4 vg2 -L 200M
[root@server ~]# lvcreate -n lv1-xfs vg2 -L 500M
[root@server ~]# lvs
[root@server ~]# mkfs.ext4 /dev/vg2/lv1-ext4
[root@server ~]# mkfs.xfs /dev/vg2/lv1-xfs
[root@server ~]# mkdir /ext4-test /xfs-test
[root@server ~]# mount -t ext4 /dev/vg2/lv1-ext4 /ext4-test/
[root@server ~]# mount /dev/vg2/lv1-xfs /xfs-test/
[root@server ~]# df -Th /xfs-test/ /ext4-test/
[root@server ~]# dd if=/dev/zero of=/xfs-test/file
[root@server ~]# dd if=/dev/zero of=/ext4-test/file

#在线扩容，在线扩容指的是文件系统不能卸载
将lv1-ext4扩容到300M
讲lv1-xfs扩容到700M

[root@server ~]# lvextend /dev/vg2/lv1-ext4 -L 300M
[root@server ~]# lvextend /dev/vg2/lv1-xfs -L 700M
[root@server ~]# lvs
#因为扩容的部分没有格式化，所以在文件系统上检测不到。
#ext4将扩容的空间加入到文件系统操作：
[root@server ~]# resize2fs /dev/vg2/lv1-ext4
[root@server ~]# df -Th /ext4-test/
[root@server ~]# xfs_growfs /dev/vg2/lv1-xfs
[root@server ~]# xfs_growfs /xfs-test/
[root@server ~]# df -Th /xfs-test/
```

