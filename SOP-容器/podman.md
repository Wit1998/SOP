# podman

## podman和docker区别

很多人可能遇到过开机重启时，由于Docker守护程序在占用多核CPU使用100%使用的情况，导致所有容器都无法启动，服务都不能用的情况。而且Docker守护进程是以root特权权限启动的，是一个安全问题，那么有什么方法解决呢？（使用podman）

```bash


1. 可以不用root权限启动
2. 没有daemon进程，
3. 启动容器的方式不同：
    # docker cli 命令通过API跟 Docker Engine(引擎)交互告诉它我想创建一个container，然后docker Engine才会调用OCI container runtime(runc)来启动一个container。这代表container的process(进程)不会是Docker CLI的childprocess(子进程)，而是Docker Engine的child process。
    # Podman是直接给OCI containner runtime(runc)进行交互来创建container的，所以container process直接是podman的child process。
4. 因为docker有docker daemon，所以docker启动的容器支持--restart策略，但是podman不支持，如果在k8s
中就不存在这个问题，我们可以设置pod的重启策略，在系统中我们可以采用编写systemd服务来完成自启动
```

```bash
#Podman 只是 OCI 容器生态系统计划中的一部分，主要专注于帮助用户维护和修改符合 OCI 规范的容器镜像。其它的组件还有 Buildah、Skopeo 等
```

```bash
#podman的仓库配置文件
$ cat /etc/containers/registries.conf
unqualified-search-registries = ["docker.io"]
[[registry]]
prefix = "docker.io"
location = "yn07wper.mirror.aliyuncs.com"

#注释掉默认的仓库配置
$ mv /etc/containers/registries.conf.d/000-shortnames.conf /etc/containers/registries.conf.d/000-shortnames.conf.bak

```

```bash
#针对selinux打标签的问题
$ podman run -dt  --name web2 -p 8080:80 -v /web:/usr/local/apache2/htdocs:Z httpd


#podman的容器自启动,container默认启动都是进程的方式，且命名较复杂，不方便直接systemctl enable xx.service


#--name表示使用容器的名字来代替容器的ID
#--files表示生成systemd的服务文件
#web1表示使用哪个容器生成systemd的服务文件
$ podman generate systemd --files --name web1
$ mv container-web1.service /etc/systemd/system/
$ systemctl daemon-reload

#这里要注意selinux的问题
$ ls -Z /etc/systemd/system/container-web1.service
#还原文件，取消掉标签
$ restorecon -RvF /etc/systemd/system/container-web1.service
$ systemctl enable container-web1.service --now
$ systemctl status container-web1.service

## 但是缺点是在重启后，原来的服务会自动废弃且没有回收资源，相当于只是重新启动了一个相同的服务
$ reboot
$ podman ps -a
26fecc8d5308  docker.io/library/httpd:latest  --name web1 -v /w...  2 hours ago  Created  jovial_ride

```

```bash
#第二种podman容器自启动方法
$ podman generate systemd --files --new --name web1
$ cat container-web1.service
```

```bash
# 非根用户使用podman容器
#podman如果要使用普通用户来管理容器，那么这个普通用户必须是ssh登陆或者通过终端登陆才行。否则会有问题。
#root用户的images，普通用户是看不到的。所以普通用户需要自己拉image
$ podman pull httpd
$ podman run -dt --name glsweb1 -p 54321:80 -v


#非根用户使用systemd接管podman容器
#创建~/.config/systemd/user目录来存放普通用户的systemd文件
$ mkdir ~/.config/systemd/user -p
podman generate systemd --new --files --name glsweb1 /home/gls/container-glsweb1.service
#将服务文件移动到普通用户的systemd的目录文件
$ mv container-glsweb1.service ~/.config/systemd/user/

#赋予普通用户的systemd管理权限
$ loginctl enable-linger
$ systemctl --user daemon-reload
$ ls ~/.config/systemd/user/
$ systemctl --user enable container-glsweb1.service --now
```



## 5.buildah

Buildah 是一个专注于构建 OCI 容器镜像的工具，Buildah 构建速度非常快并使用覆盖存储驱动程序，可以节约大量的空间。

## 6.skopeo

Skopeo 是一个镜像管理工具，允许我们通过 Push、Pull和复制镜像来处理 Docker 和符合 OCI 规范的镜像。