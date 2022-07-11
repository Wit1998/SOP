# docker网络模式

4种网络模式分别为bridge，host，none，container。常用的是host模式和bridge模式。

1. ## docker的bridge网络模式

bridge网络模式会为每个容器分配IP地址。

```bash
#可以看到在host上看到的两个桥接到docker网桥的网卡和容器内部看到的网卡并不是一个，因为mac地址不同。
#在host上看到的网卡和容器内部的网卡其实叫做veth pair

docker inspect centos-test  |grep -i macaddress
ip addr show vethbb7ec02


#veth pair其实是一对虚拟网卡，这一对虚拟网卡就像"网线"一样。
#物理机创建一个类似网关的地址，创建容器时物理机会新开一个网卡，虚拟容器开一个网卡，然后像物理网线一样连接起来，网关也会分配具体的IP给容器。

#按道理说，这是一个私有地址，应该不能通过物理机的防火墙访问internet，这里做了伪装(SNAT)
iptables -L -t nat -n
Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination         
MASQUERADE  all  --  172.17.0.0/16        0.0.0.0/0           
POSTROUTING_direct  all  --  0.0.0.0/0            0.0.0.0/0           
POSTROUTING_ZONES_SOURCE  all  --  0.0.0.0/0            0.0.0.0/0           
POSTROUTING_ZONES  all  --  0.0.0.0/0            0.0.0.0/0

# 查看docker网络
docker network ls
#连接到bridge的容器都在bridge的网桥中被记录了
docker network inspect bridge

```

```bash
#总结：Docker的bridge网络模式的底层采用的是Linux bridge，veth pair，iptables

```

## 2.Docker的Host网络模式

#采用host网络模式的Docker容器，可以直接使用宿主机的IP地址与外界通信，若是宿主机的eth0是一个公有IP，那么容器也会共享这个公有IP，同时容器服务的端口也可以使用宿主机的端口，无需进行SNAT的转换。

```bash
#使用--network指定容器运行时的网络模式
docker run -dt --name centos1 --network host centos
docker exec -it centos1 ip a show


docker inspect centos1 |grep -A20 Networks 
docker network inspect host
docker network list
```

```bash
#带来的好处就是性能好，带来的劣势就是容器的网络缺少了隔离性，增加了风险，并且由于所有容器和宿主机共享同一个网络，网络资源也受到了限制。
```



## 3. Docker的none网络模式

因为如果使用了none作为Docker容器的网络模式，那么就意味着容器会禁用网络功能，只会留一个环回接口。

```bash
#
docker run -dt --name centos2 --network none centos

docker exec -it centos2 ip a show
docker network inspect none

```

```bash
#none模式的好处就是Docker容器几乎不参与网络的配置，那么如果你想针对这个容器配置网络，就需要第三方的driver。none模式的优点就是让你容器的网络设置更加的开放，不再局限于Docker自带的网络模式。
```

## 4.Docker的container网络模式

container类型的网络表明指定一个容器，和那个容器共享一个网络

```bash
docker run -dt --network container:centos1 --name centos3 centos


#与共享容器的区别
"NetworkMode": "container:93964dfe493c3a0212d1a78c2735d96c348ad1e4f6419de3909a75f0902660de",
"NetworkMode": "none",
"Networks": {}
"Networks": {"none": {...}}

```

## 5.Docker自定义网络和网络连接

建立的自定义网络就是桥接模式，和桥接模式的特点一致，可以上网，也是做了SNAT

```bash
docker network create fs-network
ip a show
docker run -dt --name fs-centos1 --network fs-network centos
docker run -dt --name fs-centos2 --network fs-network centos
docker inspect fs-centos1 |grep -i network
docker inspect fs-centos2 |grep -i network

docker network disconnect fs-network fs-centos1
docker network disconnect fs-network fs-centos2

docker network rm fs-network
```

## 6.Docker容器间网络通信

第一个要保证容器间的通信，第二要保证外部能够访问到我们的容器承载的业务。

```
docker run -dt --name fs-centos1 centos
docker run -dt --name fs-centos2 centos
docker exec -it fs-centos1 ip a
docker exec -it fs-centos2 ip a
docker exec -it fs-centos2 ping -c1  172.19.0.2
```

## 7.Docker容器被外部访问

Docker容器被外部访问一般基于两种网络模式，第一种是bridge，第二种是Host。
如果是bridge模式则需要通过DNAT的方法让我们的Docker容器被访问到
如果是Host模式，则我们的Docker容器可以直接被访问到

```bash
#本机端口访问
docker run -dt --name gzy-web1 nginx
docker inspect fs-web
ping 172.17.0.2
curl 172.17.0.2
#上面的情况默认外部是访问不到nginx容器的，因为nginx容器所获得的地址是docker网桥内部的地址。

#内网访问
#docker自动做了SNAT的端口映射，可实现内网访问
docker run -dt --name fs-web2 -p 8080:80 nginx
curl 10.8.0.218:8080
#使用host的网络模式也可实现内网访问
docker run -dt --name fs-web3 --network host nginx
curl 10.8.0.218:80
## 注意，这里内网需要关闭防火墙，或者允许此端口的暴露
systemctl stop httpd
ss -tunlp
curl 10.8.0.218:80
```

