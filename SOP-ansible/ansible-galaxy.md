# ansible-galaxy

```

# 初始化拉取的role
ansible-galaxy install -r roles/requirements.yml -p roles


ansible-galaxy list
ansible-galaxy delete
ansible-galaxy remove

```

拉取的配置文件

```
#requirement.yml

---
- src: git@workstation.lab.example.com:student/bash_env
  scm: git
  version: master
  name: student.bash_env
```

创建playbook去使用这个拉取的role

```
---
- name: xxx
  hosts: xxx
  var: 
    xxx: xxx
  pre_tasks:
    - name: xxx
      user:
        name: xx
        state: xx
        force: xx
        remove: xx
  roles:
    - student.bash_env
  post_tasks:
    - name: xxx
      user:
        name: xxx
        state: xx
        password: xxx
```

