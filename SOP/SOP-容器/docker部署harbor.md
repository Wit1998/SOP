- 启动容器时，进入容器内部，可以发现逻辑卷都默认挂载在/var/lib目录

```bash
TS: 
Get "https://10.8.0.115:5000/v2/": http: server gave HTTP response to HTTPS client 由于默认都是https，本地仓库走http，因此需要加insecure-registries参数，然后重启docker服务 vim /etc/docker/daemon.json  systemctl daemon-reload systemctl restart docker 
TS: 
Error response from daemon: pull access denied for harbor, repository does not exist or may require 'docker login': denied: requested access to the resource is denied 
检查网络情况 
TS： 
WARNING: The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested standard_init_linux.go:228: exec user process caused: exec format error arm64的机器的原因 
TS: 
下载docker-compose的python版本问题。更换3.8  
参考：https://stackoverflow.com/questions/63207385/how-do-i-install-pip-for-python-3-8-on-ubuntu-without-changing-any-defaults 
sudo apt install python3.8 
#安装好python3.8后，需要指向python3.8，有两种方式。映射或别名 
which python3.8 
ln -s /usr/bin/python3.8 /usr/bin/python 
alias python="python3.8" 
#下载3.8的pip 
python -m pip install pip 
#安装好pip3.8后，需要指向pip3.8，有两种方式。映射或别名
```

# 安装harbor

https://www.jianshu.com/p/df2e5d02825c

```bash
wget https://github.com/goharbor/harbor/releases/download/v1.10.11/harbor-offline-installer-v1.10.11.tgz pip install docker-compose tar -zxvf harbor-offline-installer-v1.10.11.tgz -C /opt vim /opt/harbor/harbor.yml 
# 修改port和hostname 
bash /opt/harbor/prepare bash /opt/harbor/install.sh docker-compose up -d 
# harbor镜像传入本地harbor(直接docker ps -q就可)  docker rm `docker ps -q`
docker images |awk '{print $1":"$2}' |grep harbor |grep -v 'fs.harbor' |while read name; do docker tag  $name $(print fs.harbor.com:8091/harbor/`echo $name`) &&sleep 1;done 
docker images |awk '{print $1":"$2}' |grep harbor |grep -v 'fs.harbor' |while read name; do docker push $(print fs.harbor.com:8091/harbor/`echo $name`);done 
```