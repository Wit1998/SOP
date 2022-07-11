# SOP-Hexo

https://hexo.io/zh-cn/docs/

```bash
#安装 Git
sudo yum install git-core
#安装 Node.js  https://github.com/nodesource/distributions
(centos为例)
#注册rpm
#Node.js v18.x
curl -fsSL https://rpm.nodesource.com/setup_18.x | bash -
yum makecache
yum install nodejs
#测试
curl -fsSL https://deb.nodesource.com/test | bash -
#安装 Hexo (注意版本一致性)
npm install -g hexo-cli
npm install hexo
echo 'PATH="$PATH:./node_modules/.bin"' >> ~/.profile




```

```bash
#建站
#会创建 public 和 source 目录
hexo init <folder>
cd <folder>
npm install


```

```bash
#安装插件hexo admin
npm install --save hexo-admin
```

