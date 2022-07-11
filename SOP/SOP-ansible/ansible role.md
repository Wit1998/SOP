# ansible role

## role的目录

- tasks 真正的工作目录
- template task需要用到的模板，task自动会去里面找，不用声明
- files task需要用到的文件，task自动会去里面找，不用声明
- vars task需要用到的变量，task自动会去里面找，不用声明
- default 优先级很低，如果没有设置，会使用这里的缺省值。在任何地方有声明变量都会覆盖掉default变量
- handlers 把task中有notify的模块写到handles

```yaml
---
- name: xxx
  hosts: xx
  # pre_tasks优先级最高
  pre_tasks:
    - debug: 
        msg: 'pre-task'
        notify: my handler
  roles:
    - role1
    - role2
      # roles下定义优先级高，但不会覆盖vars目录下面的变量
      var1: val1
      var2: val2
  # play下定义var，优先级低于在roles里面定义
  vars: 
    var1: val1
  tasks:
    - debug:
        msg: 'first task'
      notify: my handler
  handlers: 
    - name: my handler
      debug:
        msg: Running my handler
```

## system-roles

```

```

