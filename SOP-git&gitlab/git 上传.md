# git 上传

以上传至github为例。

```bash
# 把上次目录推到本地git仓库
cd [dir]
git init .
git add .
git commit -m 'description'


ssh-keygen -t rsa -C 'description'
# 

# 然后在setting中配置生成的key

# 连接远端仓库
 git remote add origin [https://xxx.git]
# 推送到仓库
git push -u origin master
```



