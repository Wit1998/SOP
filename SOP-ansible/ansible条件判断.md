# ansible条件判断

只有达到预期和未达到预期

```yaml
---
- name: xxx
  vars:
    state: fales
    my_service: httpd
  tasks:
    yum:
      name: "{{ my_service }}"
      when: my_service is defined
```



```yaml
---
- name: Demonstrate the "in" keyword
  hosts: all
  gather_facts: yes
  vars:
    supported_distors:
      - RedHat
      - Fedora
  tasks:
    - name: install httpd
      yum:
        name: http
        state: present
      when: ansible_distribution in supported_distros
```

常见判断

```
equal

```



## 多条件判断语法

```
# 且(都满足)
and
# 或(满足一个)
or

#下面也表示且
when:
  - xxx
  - xxx 
  
when: >
  ( xxx and
    xxx )
  or 
  ( xxx and 
    xxx )
```

