# SOP-Gitlab 数据迁移(ubuntu)

科学上网，可选外网迁移方案：https://medium.com/gits-apps-insight/migrating-gitlab-to-another-server-990092c5179

```bash
#1.对应两个gitlab的版本
查看gitlab版本(二选一):
cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
gitlab-rake gitlab:env:info
两台服务器的Gitlab版本必须是统一的，如有不统一，请先进行升级统一。
目前我们使用的gitlab版本是：12.8.7
如需升级可在数据迁移完成之后进行gitlab版本升级。
#2.备份数据，将旧服务器上的数据打包
gitlab-rake gitlab:backup:create RAILS_ENV=production
备份后的文件一般是位于/var/opt/gitlab/backups下, 自动生成文件名文件名如 1614232417_2021_02_25_12.8.7_gitlab_backup.tar
#3.数据迁移，将备份数据拷贝至新服务器（已经搭建好giltab同版本环境）
服务器之间的拷贝命令：scp
把对应版本的数据从旧服务器上拷贝到新服务器的gitlab备份目录里
scp 1614232417_2021_02_25_12.8.7_gitlab_backup.tar ubuntu@10.8.0.15:1614232417_2021_02_25_12.8.7_gitlab_backup.
tar
sudo cp 1614232417_2021_02_25_12.8.7_gitlab_backup.tar /var/opt/gitlab/backups/
#4.执行备份文件恢复
新服务器执行恢复命令
chown -R git.git /var/opt/gitlab/backups/
gitlab-rake gitlab:backup:restore RAILS_ENV=production BACKUP=1614232417_2021_02_25_12.8.7
注意：这里没有后面的_gitlab_backup.tar名字
一路yes，恢复是会先删除新服务器上所有gitlab数据的。
备份完成会有如下红字警告
Warning: Your gitlab.rb and gitlab-secrets.json files contain sensitive data
and are not included in this backup. You will need to restore these files manually
红字部分表示gitlab.rb 和 gitlab-secrets.json 两个文件包含敏感信息。未被备份到备份文件中。需要手动备份
备份tar包一定要放到备份路径下。恢复是删除原有数据，恢复备份tar包中的数据。
在新服务器恢复备份，一定要记得将gitlab.rb 和 gitlab-secrets.json 手动复制到相应路径下。
gitlab.rb路径：/etc/gitlab/gitlab.rb
gitlab-secrets.json路径：/etc/gitlab/gitlab-secrets.json
#5.重载配置
gitlab-ctl reconfigure
#6.重启gitlab
gitlab-ctl restart
#7.check检查
gitlab-rake gitlab:check SANITIZE=true
```

```bash
参考连接：
https://www.cnblogs.com/jxd283465/p/11525629.html
https://cloud.tencent.com/developer/article/1535601
```



