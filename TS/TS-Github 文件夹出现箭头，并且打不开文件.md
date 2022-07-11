# Github 文件夹出现箭头，并且打不开文件

当你在自己的项目里clone了别人的项目，[github](https://so.csdn.net/so/search?q=github&spm=1001.2101.3001.7020)就将他视为一个子系统模块，导致在上传代码时该文件夹上传失败，并在github上显示向右的白色箭头。

```bash
删除子文件夹里面的.git文件
执行git rm --cached [文件夹名]
执行git add .  (或是把 . 换成自己要传的文件夹）
执行git commit -m "commit messge"
执行git push origin [branch_name]
```

