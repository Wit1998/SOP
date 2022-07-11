# docker代理使用国外镜像



## 1.配置镜像加速器

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
	"registry-mirrors": ["https://xxxxxxxxx"]
}
EOF
sudo systemctl daemojn-reload
sudo systemctl restart docker
```

## 2.常见镜像仓库

```bash
DockerHub镜像仓库:
https://hub.docker.com/
google镜像仓库：
https://gcr.io/google-containers/
https://gcr.io/kubernetes-helm/
https://gcr.io/google-containers/pause
quay.io镜像仓库：
https://quay.io/repository/
elastic镜像仓库：
https://www.docker.elastic.co/
RedHat镜像仓库：
https://access.redhat.com/containers
阿里云镜像仓库：
https://cr.console.aliyun.com
华为云镜像仓库：
https://console.huaweicloud.com/swr
腾讯云镜像仓库：
https://console.cloud.tencent.com/tcr
```

## 3.配置docker的代理

```bash
1.将socks5的代理转换成http的代理
[root@localhost ~]# yum -y install epel-release
[root@localhost ~]# yum -y install privoxy
[root@localhost ~]# vim /etc/privoxy/config
[root@localhost ~]# egrep '^listen-address' /etc/privoxy/config -A 1
listen-address 0.0.0.0:8118
forward-socks5t / 192.168.199.11:10808 .
[root@localhost ~]# systemctl enable privoxy --now
[root@localhost ~]# netstat -tunlp | grep 8118
tcp 0 0 0.0.0.0:8118 0.0.0.0:* LISTEN
6172/privoxy



2.配置docker使用http的代理
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo touch /etc/systemd/system/docker.service.d/proxy.conf
sudo chmod 777 /etc/systemd/system/docker.service.d/proxy.conf
sudo echo '
[Service]
Environment="HTTP_PROXY=http://10.163.1.110:8118"
Environment="HTTPS_PROXY=http://10.163.1.110:8118"
' >> /etc/systemd/system/docker.service.d/proxy.conf
sudo systemctl daemon-reload
sudo systemctl restart docker


```

