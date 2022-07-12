## nmcli 

前提：安装networkmanage

```
systemctl status NetworkManger

```

```bash
1、查看网络接口信息(device)
nmcli
nmcli device status
nmcli device show
nmcli device show interface-name

2、查看连接信息（conn）
nmcli connection show
nmcli connection show --active 
nmcli connection show interface-name
3、激活/取消连接
nmcli connection [up/down] [connection-name] # 打开/取消连接
nmcli device [connect/disconnect] interface-name # 激活/取消接口
```

```bash
4、创建动态ip地址连接
nmcli connection add type [ethernet/] con-name [connection-name] ifname [interface-name]
5、创建静态ip地址连接
nmcli connection add type [ethernet/] con-name [connection-name] ifname [interface-name] ipv4.method manual ipv4.addresses address
6、网卡参数
ipv4.method [manual/auto]
ipv4.addresses 
ipv4.dns
ipv4.dns-search [example.com]
ipv4.ignore-auto-dns [ture]
connection.autoconnect [yes]
connection.id eth0
connection.interface-name [eth0]

7、修改连接配置(modify)
nmcli connection modify []
```



