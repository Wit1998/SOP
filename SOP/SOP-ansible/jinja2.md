# jinja2

http://docs.jinkan.org/docs/jinja2/index.html

### 说明：

和copy模块很像，但是template模块可以获取变量的值，而copy是原封不动的复制

### 分隔符

- {% EXPR %}作为循环或条件判断表达式
- {{ EXPR }}输出变量的值
- {# COMMENT #}表示注释

```yaml
#推荐在j2文件中添加改变了
ansible_managed = Ansible managed: {file} modified on %Y-%m-%d %H:%M:%S by {uid} on {host}

#使用方法
cat /etc/motd/xxx.j2
This system is based on
{{ ansible_distribution }} {{ ansible_distribution_version }} 
deployed on {{ ansible_architecture }} architecture

---
- name: test jinja2
  hosts: server1
  tasks:
    - name: use jinja2 template
      template:
        src: xxx.j2
        dest: /etc/motd
        owner: root
        group: root
        mode: 0664
```

## ansible_managed

```yaml
# 定义在ansible.cfg

[defaults]
ansible_managed = Ansible managed: modified on %Y-%m-%d %H:%M:%S
```



if 

```

```



for

```

```

