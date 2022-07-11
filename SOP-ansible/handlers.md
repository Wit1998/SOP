# handlers

### 说明：

Handlers是响应一个通知的tasks，它是需要被触发的tasks。每个handlers都有一个唯一的全局名称，并
且在一个playbook的底端等待被触发。

handlers可以被看做一个不活跃的tasks，handlers只有被notify后才会触发它的执行。

### handlers使用注意事项

1. handlers不是运行在notify而是handlers
2. task section完成才会运行handlers
3. handlers不能重名，如果重名了，只会运行一个handlers
4. handlers定义在include里面不会生效
5. 无论多少个task notify了handlers，只会运行一次handlers
6. 只有task change了才会notify handlers

注意：不能用handlers替代tasks

### 举例

```yaml
---
- name: xxx
  force_handlers: yes
  hosts: xxx
  tasks:
    - name: xxx
      yum: 
        name: xxx
        state: present
      notify:
        - restart db service
  handlers:
    - name: set db password
        service: 
          name: xxx
          state: restarted
```



## task错误处理

```bash
#task的错误控制
ignore_errors: yes

# task报告的状态，默认是task做出改变后通知handlers，没有change就不通知。
# 当task失败时，play被终止，tasks notify 和handlers都不会被执行，但如果在play中设置了force_handlers: yes，handlers就会被强制执行
force_handlers: yes

# 控制task的changed状态, 使用changed_when 可以在task满足条件时改变成changed状态
# 如command这种模块，无论执行什么都会改变成changed状态
# 无论怎样都不是changed
changed_when: false
# 当result匹配为success时会改变成changed状态
changed_when: "'Success' in command_result.stdout"
```

### TS

```bash
# 不加force_handlers:yes的情况，如果task section未完成，但notify下面的task执行成功了，重新执行的话，就会因为task执行过一次直接变成ok，而非changed，这时会自动跳过notify。
# 要让notify执行的话，需要加上changed_when: true
```



### fail模块和when的使用

```
tasks:
  - name: Run user careation script
    shell: /usr/local/bin/create_users.sh
    register: command_result
    ignore_errors: yes
    
  - name: Report script failure
    fail:
      msg: "the password is missing in the output"
    when: "'password missing' in command_result.stdout"
```

## block rescue always

```bash
#block：定义一组运行的task，block 可以将多个task整合成一个做成批处理，这样就方便changed_when的统一判断了
#rescue：如果在block中定义的task运行失败了，rescue定义的task将会被顺序执行来恢复那个错误的task。
#always：无论在block和rescue中定义的task运行成功还是失败， always定义的task总会运行
```

