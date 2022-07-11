# docker持久化

## 1.Docker镜像的分层

```ini
Docker容器和Docker镜像的关系。
Docker镜像运行之后就是Docker容器。（通俗的解释）
基于Docker镜像挂载一个可读可写的文件系统，这个文件系统就是Docker容器的根分区。有了根分区，我们才能认为Docker容器像一个操作系统，可以运行。
Docker镜像本身是没有写权限的。基于Docker镜像之上挂载一个可读可写的文件系统就可以当这个文件系统是一个Docker容器的文件系统。
#也就是说我们针对Docker容器的操作，其实是我们在可读可写的文件系统上操作，而不是操作Docker容器底层的镜像。因Docker镜像是不具备写权限的。所以无论你在Docker容器内做任何操作都不会影响到Docker镜像。
为何Docker容器的存储是非持久化的？
#因为Docker容器的文件系统是即用即删的。也就意味着，当你的Docker容器被删除之后，你挂载的Docker文件系统也会被删除，那么你在Docker容器中做的任何操作，都会随着Docker容器的文件系统删除而删除。

```

## 2.Docker容器使数据卷Volume

```bash
#/var/lib/docker/volume是存储本地volume的默认路径，所以说我们创建的volume默认都存放在这个位置
docker volume create fs1
docker run -dt --name fs-vollume-centos -v fs1:/fs1 centos
```

## 3.Docker容器使用bind mounts

容器的bind mounts相比于容器的数据卷来说，使用上更加的灵活。

```
mkdir /mnt/fs2
docker run -dt --name fs-bind-centos -v /mnt/fs2:/fs2 centos
```

## 4.Docker容器持久化实践（apache）

```bash
docker run -dt --name web1 --network host httpd
docker  exec -it web1 find  -name *.html
#了解到apache的工作目录在/usr/local/apache2/htdocs

#持久化该目录
mkdir /web
docker run -dt --name web2 --network host -v /web:/usr/local/apache2/htdocs httpd
docker run -dt --name web3 -p 8080:80 -v /web:/usr/local/apache2/htdocs httpd
echo "fsloveqwl" >> /web/index.html
curl localhost
curl localhost:8080
```

## 5.Docker容器的自启动策略

docker容器默认情况下，如果宿主机关机再开机之后，docker容器将不会自动启动。

```bash
1.我们必须要确保容器的服务是开机自启动的
systemctl enable docker --now
2.我们必须要确保哪些容器需要开机自启动
#--restart参数有三个
# 默认是no，表示容器退出时，不重启容器
# on-failure表示只有容器在非0状态退出时才重新启动容器(推荐)
# always表示无论任何状态退付出，都重启容器
docker run -dt --name web2 -v /web:/usr/local/apache2/htdocs --network host --restart=on-failure httpd
```

