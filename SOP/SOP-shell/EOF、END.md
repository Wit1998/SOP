# EOF、END

这些都叫**HERE文档**。可以替换做任意一对的字符

## EOF

### cat和EOF(end of file)的使用

```bash
# 创建文件
cat  >  filename
#将几个文件内容合并为一个文件内容。
cat   file1   file2  > file

# 追加/覆盖文件
# cat和eof结合使用具有追加功能
# 追加内容到文件
cat  <<EOF >> test.sh  内容  EOF
cat >>test.sh  <<EOF  内容  EOF
# 覆盖/创建内容到文件
cat <<EOF > test.sh   内容  EOF   
cat >test.sh  <<EOF   内容  EOF    # 不可追加，直接覆盖

# 常见用法
cat >./test1.txt <<EOF
xx
xx
EOF
```

## <<-EOF和<<EOF区别

```bash
# <<-EOF将忽略起止内容中前面的tab制表符，而<<EOF将不会，
```

