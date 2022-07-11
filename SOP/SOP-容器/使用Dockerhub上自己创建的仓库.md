# 使用Dockerhub上自己创建的仓库

```bash
#首先，我们需要先理解docker的镜像仓库，docker镜像仓库的结构分为仓库本身，仓库里面的小仓库，还有tag。
我们来分析一下dockerhub的仓库构成。dockerhub的镜像仓库名称为docker.io，当我们想要从dockerhub上拉
取ubuntu镜像的时候，我们就从docker.io的大仓库中找到ubuntu这个小仓库，找到ubuntu这个小仓库之后，我们
需要指定我们要从ubuntu这个小仓库中拉取哪个tag（标记）个ubuntu镜像。
docker pull docker.io/library/ubuntu:latest
[root@localhost ~]# docker pull centos:centos7
centos7: Pulling from library/centos
2d473b07cdd5: Pull complete
Digest: sha256:0f4ec88e21daf75124b8a9e5ca03c37a5e937e0e108a255d890492430789b60e
Status: Downloaded newer image for centos:centos7
docker.io/library/centos:centos7
#一般来说我们习惯将大仓库叫做docker registry，小仓库的名字叫做repository
1.在dockerhub上创建账号
2.在dockerhub登陆自己的账号并创建小仓库(repository)
3.修改镜像的tag
[root@localhost ~]# docker images
[root@localhost ~]# docker tag ubuntu:latest a390133368/gls-pub:ubuntu-latest
[root@localhost ~]# docker images
4.使用自己的账号登陆到dockerhub并上传镜像到自己的小仓库
[root@localhost ~]# docker login
[root@localhost ~]# docker push a390133368/gls-pub:ubuntu-latest
#查看已经push的镜像
```

