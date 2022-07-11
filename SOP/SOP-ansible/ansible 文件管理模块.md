# ansible 文件管理模块

- blockinfile 在插入内容时添加上下的备注
- copy 将控制节点向被管理主机拷贝
- fetch 将被管理主机向控制节点拉取
- file 可以使用该模块创建、删除文件或目录等。(present、absent)
- lineinfile 在文本中插入文件
- stat
- synchronize 适用于远程文件的更改，有些方面与copy类似
- template 将j2模板里面的值映射出来，并保存在被管理主机上

## fetch

```yaml
把某文件拉取到管理主机某目录
---
tasks:
  - name: xxx
    fetch:
      src: xxx
      dest: xxx
      # 确保将所有主机名、目录等信息都复制过来
      flat: no
```

## copy

```yaml
# 把管理主机文件拷贝到目标主机
---
tasks:
  - name: xxx
    copy: 
      src: xxx
      dest: xxx
      owner: xxx
      group: xxx
      mode: xxx
      # 设置selinux用户类型
      setype: samba_share_t
```

## file

```yaml
# 管理目标主机的文件属性
# 修改selinux的context
---
tasks:
  - name: xxx
    file: 
      path: xxx
      seuser: _default
      serole: _default
      setype: _default
      selevel: _default
   
# 删除文件
tasks:
  - name: xxx
    file: 
      path: xxx
      state: absent
      
```

## lineinfile

```yaml
#插入一行文字
---
tasks:
  - name: xxx
    lineinfile:
      path: xxx
      line: this line was added by the lineinfile module.
      state: present
      
# 替换文字
---
tasks:
  - name: xxx
    lineinfile:
      # 匹配替换的内容
      regexp: '^this'
      # 插入在
      insertafter: '^www.xxx'
      path: xxx
      line: that line was added by the lineinfile module.
      state: present
      
# 插入文字到指定位置
---
tasks:
  - name: xxx
    lineinfile:
      insertafter: '^this'
      path: xxx
      line: that line was added by the lineinfile module.
      state: present
```

## block

```yaml
# 插入一个上下的备注标语，就形成了一个block
---
tasks:
  - name: xxx
    block:
      path: xxx
      block: |
        this block of text consists of two lines.
        they have been added by the blockinfile module
      state: present
```

