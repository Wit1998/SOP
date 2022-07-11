# 容器（docker）

容器是一门技术，不只是docker，docker是容器的实现，还有类似podman的技术，容器属于云计算范畴，不能代替虚拟化。

docker、rocket、warden



## docker基本操作

```bash
#images/container
docker images -a
docker rmi [IMAGES ID] 
docker rmi -r [IMAGES ID]
docker rmi `docker images -q`

docker ps -a 
docker rm [CONTAINER ID] 
docker rm -r [CONTAINER ID] 
docker rm `docker container -aq`

#tag
docker tag xxxx:xxxx xxxx:xxxx

#search镜像
docker search [IMAGES ID]

#Docker镜像导出
#-o参数表示输出为指定的文件
docker save ubuntu:latest -o ubuntu-latest.tar

#Docker镜像导入
#-i参数表示导入指定的文件
docker load -i ubuntu-latest.tar
docker load < ubuntu-latest.tar
```

## docker运行

```bash
1. 一次性运行容器
#需要注意的是如果docker run 后面指定的镜像没有加tag，那么默认就会运行tag为latest的镜像。
docker container run xxx

2. 交互式运行容器
#-i就表示交互的意思
#-t就表示tty，表示为容器开启一个终端
#-p [主机端口:容器端口] 表示指定端口
#--name [容器名] 表示指定容器名
docker run -it centos

#容器并没有/boot目录，原因也很好理解，因为容器不是操作系统，所以不需要引导，也自然就没有内核存放的位置。
#容器第一个进程为/bin/bash之类的，主机第一个进程为/usr/lib/systemd/systemd
#因此exit退出容器，就相当于结束了容器操作系统的生命周期

#如果你使用-i -t参数以交互式的形式运行了容器，那么你可以使用ctrl+p+q（按住ctrl，再按p，p松手，再按q）临时的退出容器，这个容器就并没有真正的被关闭。

docker ps -a
docker attach xxxxxx



3. 后台运行容器
#-d参数表示daemon，表示将容器以守护进程的形式运行
docker run -d centos
#那为什么在后台运行之后也是直接退出了呢？运行一次在前台退出和在后台退出是一样的。就好比运行一个ls命令一样，不管你是在前台运行还是在后台运行，都是运行完之后就退出

#如何让一个容器在运行之后不退出呢？我们需要增加一个-t参数，-t参数的意思是开启一个终端
#docker run "image" "command"后面的command会替代image构建的最后一个命令。
docker run -dt centos /bin/bash

4. 进入容器
docker exec -it 1facba8577ae bash
#假如run的时候使用的是/bin/bash， exec使用的是bash,进入容器后退出，这里只退出了bash的终端，/bin/bash的依然存在。
[root@646717583ed6 /]# ps -aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1  12036  1960 pts/0    Ss+  13:12   0:00 /bin/bash
root        15  0.5  0.2  12036  2120 pts/1    Ss   13:16   0:00 bash
root        29  0.0  0.2  47572  2088 pts/1    R+   13:16   0:00 ps -aux

```

## 容器的生命周期管理

```bash
1. 容器的启动
docker run / docker container run
2. 容器的正常停止
docker stop [CONTAINER ID]
3. 容器的手动启动
#没有加-d参数的容器，即使我们将它手工启动，也不会让他持续性运行
#只有以后台的形式运行的容器才有生命周期管理的意义
docker start [CONTAINER ID]
#Exited后面有一个数字，数字代表着容器结束时候的返回值，返回值是0表示容器是正常结束的，返回值非0表示容器是非正常结束的

4. 容器强制杀掉
docker kill [CONTAINER ID] 

5.容器的重启
docker restart [CONTAINER ID] 

```

## 查看容器信息

```bash
1.查看容器log
#重命名容器名称
docker run -dt --name web nginx
docker logs web

2. 查看容器进程
#一般情况你可以进入到容器内部使用ps命令查看容器的进程信息，但并不是所有的容器都支持这么操作
[root@localhost ~]# docker exec -it centos-test bash
[root@localhost ~]# docker exec -it web bash
root@cf839d63840e:/# ps aux
bash: ps: command not found
root@cf839d63840e:/# top
bash: top: command not found
root@cf839d63840e:/# exit
docker top web

3. 查看容器内部细节
#返回json格式的所有容器信息
docker inspect web

4.容器与主机间的文件拷贝
#将ubuntu-latest.tar拷贝到centos-test容器的根目录下
docker cp ./ubuntu-latest.tar centos-test:/
#将centos-test容器目录下的ubuntu-latest.tar拷贝到主机当前目录下
docker cp centos-test:/ubuntu-latest.tar .
```

```
docker container prune
docker image prune
```

