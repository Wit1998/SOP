# linux文件归档

注意打包、解包和压缩、解压缩的区别

## zip

```bash
#zip(gz压缩包)
#gzip (不支持目录的压缩，主要针对单个文件进行压缩)
gzip [file] (原文件消失，变成zip的文件包)
#查看压缩包内容
gzip -l xxx.gz
guzip [file]
# 压缩比例
gzip -9 [file]
# 解压
guzip xx.gz(原压缩文件消失，变成文件)

#bzip2(bz2压缩包)
bzip2 xxx
bzip2 xxx.bz2
#查看压缩文件类型
file xxx.bz2
```

## tar

```bash
# c 打包 
# f 指定文件 
# v 详细信息(写脚本时不需要) 
# j 通过bzip2对打包的包再进行压缩或解压
# z 通过gzip对打包的包再进行压缩或解压
tar -zcvf xxx (原文件还在)

# x 解包
tar -zxvf xxx (原文件还在)
```

