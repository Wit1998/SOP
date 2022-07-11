# loop循环

task循环的次数，取决于loop的参数个数

```yaml
---
- name: insatll app
  hosts: servera
  vars_files:
    - var/1.yml
  tasks:
    - name: yum install
      yum: 
        name: "{{ item }}"
        state: latest
      loop: "{{ web_related_pkg }}"
    - name: start service
      service: 
        name: "{{ item }}"
        enabled: true
        state: started
      loop: "{{ web_related_service }}"
```

```
# 1.yml

web_related_pkg:
	- httpd
	- firewalld
web_related_service:
	- httpd
	- firewalld
```



当loop循环的变量是变量矩阵时。(有几个"-"，循环多少次)

```yaml
---
- name: manage user
  hosts: servera
  vars_files:
    - var/user.yml
  tasks:
    - name: create user
      user: 
        name: "{{ item.name }}"
        state: present
        groups: "{{ item.group }}"
      loop: {{ users }}
```

```yaml
# user.yml

users:
  - name: fs001
    group: fs001
  - name: fs002
    group: fs002
  - name: fs003
    group: fs003
  - name: fs004
    group: fs004
```



## with_item(低版本)

和loop一致

## with_file(低版本)

把文件打印处理

```

```

