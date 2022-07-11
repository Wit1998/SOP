# RHCE RH249

```bash
#重置考试环境
rhce8.sh

./grade-exam.sh
```





## 查看配置信息



## 1.安装配置ansible

```yaml
# 配置软件仓库
yum repolist
yum list|grep -i ansible
sudo yum-config-manager --add-repo=http://content.example.com/rhel8.0/x86_64/ucfupdates
sudo cat >> /etc/repos.d/xxxx.repo << EOF ; gpgcheck=0 ;EOF
sudo yum -y install ansible
ansible --version

# 创建inventory文件
mkdir ansible
cd ansible
cat > inventory << EOF
node[1:5]
[dev]
node1

[test]
node2

[prod]
node[3:4]

[balancers]
node5

[webservers:children]
prod
EOF

#验证
ansible -i inventory dev/test/prod/balancers/webservers --list-hosts

# 创建配置文件
cat > /home/greg/ansible/ansible.cfg << EOF 
[defaults]
inventory=/home/gre/ansible/inventory
roles_path=/home/greg/ansible/roles
remote_user=greg
ask_pass=false

[privige_escalation]
become=true
become_method=sudo
become_user=root
become_ask_pass=false
EOF

# 验证
ansible all -a "id"

```

## 2.创建和运行ansible临时命令

```yaml
# 查询
ansible-doc yum_repostory

cat > /home/greg/ansible/adhoc.sh << EOF
#! /bin/bash
ansible all -m yum_repository -a 'name="EX294_BASE" descript="EX294 base software" baseurl="http://xxxxxx gpgcheck=yes gpgkey=http://xxxxxxx"'
ansible all -m yum_repository -a 'name="EX294_STREAM" descript="EX294 stream software" baseurl="http://xxxxxx gpgcheck=yes gpgkey=http://xxxxxxx"'
EOF

chmod +x /home/greg/ansible/adhoc.sh

# 检查
ansible all -a 'yum repolist'
./adhoc.sh
ansible all -a 'yum repolist'
```

## 3.安装软件包

```yaml
cat > /home/greg/ansible/packages.yml << EOF
---
- name: install pkg
  hosts: dev,test,prod
  tasks:
  # 安装软件
    - name: use yum module to install pkg
      yum:
        name: 
          - php
          - mariadb
        state: latest     
# 方法2
#    - yum:
#       name: "{{ item }}"
#        state: latest
#      loop:
#      - php
#      - mariadb    
  - name: instgall pkg
    hosts: dev
    tasks:
      # 安装软件包组
      - name: use yum module
        yum: 
          name: "@RPM Development Tools"
          state: latest
      # 更新软件
      - name: use yum module
        yum:
          name: -*-
          state: latest
# 方法2(针对dev的hosts写在同一个play里面)
#      - name: use yum module
#        yum: 
#          name: "@RPM Development Tools"
#          state: latest
#        when: "'dev' in group_names"
#      - name: use yum module
#        yum:
#          name: -*-
#          state: latest
#        when: "'dev' in group_names"

```

## 4.使用系统role

安装ntp

```yaml
# 下载rhel的软件包
yum list |grep role
sudo yum -y install rhel-system-roles

# 添加路径
yum -qa |grep rhel-system-roles
yum -ql xxx
# 修改配置文件，让ansible配置文件加载rhel-system-roles
roles_paths=/home/greg/ansible/roles;/user/share/ansible/roles

# 查看role的参数
ansible-galaxy list
# 定义role的变量
cat > timesync.yml << EOF
---
- name: use system role
  hosts: all
  vars:
    timesysnc_ntp_servers:
    	- hostname: 172.25.254.254
    	  iburst: yes
  role:
    - rhel-system-roles.timesync
EOF

# 查看节点的ntp服务器
ansible all -a 'chronyc sources'

#运行
anisble-playbook all timesync.yml
#检查
ansible all -m shell -a 'chronyc -n sources'
ansible all -m shell -a 'grep -i iburst /etc/chrony.conf'
```

## 5.使用ansible galaxy安装角色

```yaml
# 创建一个playbook，下载并安装role

cat > requirements.yml << EOF
- src: http://xxxx
  name: balancer
- src: http://xxxx
  name: phpinfo
EOF

# 还原
# ansible-galaxy remove phpinfo balancer
#执行前判断现象 
ansible-galaxy list
ls /home/xxx/ansible/roles

# 运行playbook安装role
ansible-galaxy install -r ./requirements.yml

# 验证
ansible-galaxy list
ls /home/xxx/ansible/roles
```

## 6.创建一个web role

```yaml
# 初始化一个apache的role
ansible-galaxy init apache
ansible-galaxy list

# 编辑task
cat > roles/apache/tasls/main.yml << EOF
---
- name: install pkg
  yum:
    name: httpd
    state: latest

- name: set httpd service
  service:
    name: httpd
    state: started 
    enable: yes

- name: set firewalld service
  service:
    name: firewalld
    state: started 
    enable: yes

- name: set firewall to allow http traffic
  firewalld:
    service: http
    immediate: yes
    permanent: yes
    state: enable
    
- set web content
  template:
    src: index.html.j2
    dest: /var/www/html/index.html
EOF
  
# 编辑j2文件
cat > roles/apache/templates/index.html/j2 << EOF
Welcome to {{ ansible_fqdn }} on {{ ansible_default_ipv4['address'] }}

# 创建playbook
cat > apache.yml << EOF
---
- name: use apache role
  hosts: webservers
  roles:
    - apache
EOF

# 执行play
ansible-playbook apache.yml

#检查
curl node3.domainx.example.com
curl node4.domainx.example.com
```

## 7.ansible galaxy使用角色

```yaml
cat  > role.yml << EOF
--- 
- name: use haproxy role
  hosts: balancer
  roles:
    - balancer
EOF

# 执行负载均衡role的play
ansible-playbook role.yml

#检查node5放行的端口
firewall-cmd --list-all
firewall-cmd --add-port=80/tcp
firewall-cmd --add-port=80/tcp --per

#检查 (页面会在node3和node4之间切换)
curl node5.domainx.example.com

# 添加play使用phpinfo这个role生成子页面
cat  >> role.yml << EOF
- name: use phpinfo role
  hosts: webservers
  roles:
    - phpinfo
EOF

#检查
本地浏览器访问node3.domainx.example.com/hello.php;node4.domainx.example.com/hello.php
```

## 8.创建和使用逻辑卷

```yaml
# 事先查看node逻辑卷情况
ansible all -a 'vgs'
#
cat > lv.yml << EOF
---
- name: create lv
  hosts: all
  tasks:
    - block:
        - name: create a lv use research vg
          lvol:
            vg: research
            lv: data
            size: 1500
        - name: fomat ext4 fs
          filesystem:
            fstype: ext4
            dev: /dev/research/data       
      rescue:
        - name: output some info
          debug:
            msg: Could not createe logical volume of that size
          when: ansible_lvm.vgs.research is defined
        - name: create a lv use research vg
          lvol:
            vg: research
            lv: data
            size: 800
          when: ansible_lvm.vgs.research is defined
        - name: formate ext4 fs
          filesystem:
            fstype: ext4
            dev: /dev/research/data
          when: ansible_lvm.vgs.research is defined
        - name: Volume group does not exist
          debug:
            mgs: Volume group does not exist
          when: ansible_lvm.vgs.research is defined
```

## 9.生成主机文件

```yaml
cat > /home/greg/ansible/hosts.j2 << EOF
{% for host in groups['all']  %}
{{ hostvars[host]['ansible_default_ipv4']['address'] }} {{ hostvars[host]['ansible_fqdn'] }} {{ hostvars[host]['ansible_hostname'] }}
{% endfor %}
EOF

cat > hosts.yml << EOF
---
- name: create a host file
  hosts: all
  tasks:
    - name: template a host file
      template:
        - src: hosts.j2
          dest: /etc/myhosts
      when: '"dev" in group_names'
EOF

# 执行
ansible-playbook hosts.yml

#判断
ansible all -a "cat /etc/myhosts"
```

## 10.修改文件内容

创建web内容目录

```bash
cat > issue.yml << EOF
---
- name: modify file content
  hosts: all
  tasks:
    - copy: 
        content: Development
        dest: /etc/issue
      when: '"dev" in group_names'
    - copy:
        content: Test
        dest: /dev/issue
      when: '"test" in group_names'
    - copy:
        conent: Production
        dest: /etc/issue
      when: '"prod" in group_names'
      
# 检查
ansible all -a 'cat /etc/issue'
```

## 11.创建web内容目录

```bash
# 注意生成或创建文件时selinux的权限问题

cat > /home/greg/ansible/webcontent.yml << EOF
---
- name: set web content
  tasks:
    - name: create a directory
      file:
        path: /webdev
        state: directory
        group: webdev
        mode: "2775"
        setype: "http_sys_content_t"
    - name: create a soft link
      file:
        sec: /webdev
        dest: /var/www/html/webdev
        state: link 
    - name: set web content
      copy:
        content: Development
        dest: /webdev/index.html
        setype: "http_sys_content_t"
    - name: start httpd service
      service:
        name: httpd
        state: started
        enabled: yes
    - name: set firewall rule to allow http traffic
      firewalld:
        service: http
        permanent: yes
        immediate: yes
        state: enabled
        
 
 # 测试
 curl xxxxxx
```

## 12.生成硬件报告

```bash
cat > /home/greg/ansible/hwreport.yml << EOF
---
- name: create hardware report
  hosts: all
  vars:
    hardware:
      - hw_name: HOST
        hw_info: "{{ ansible_hostname }}"
      - hw_name: MEMORY
        hw_info: "{{ ansible_bios_version }}"
      - hw_name: BIOS
        hw_info: "{{ ansible_bios_version }}"
      - hw_name: DISK_SIZE_VDA
        hw_info: "{{ ansible_devices['vda']['size'] | default['NONE'] }}"
      - hw_name: DISK_SIZE_VDB
        hw_info: "{{ ansible_devices['vdb']['size'] | default['NONE'] }}"
    tasks:
      - name: get hw report from url
        get_url:
          url: http://xxx
          dest: /root/hwreport.txt
      - name: set hw report content
        lineinfile:
          path: /root/hwreport.txt
          # 只有当item.hw_name相等的时候，才执行下一个，不相等覆盖原有的值。循环赋值
          regexp: '^{{ item.hw_name }}='
          line: "{{ item['hw_name'] }}={{ item['hw_info'] }} "
        loop: "{{ hardware }}"
```

## 13.使用ansible vault

```bash
# 加密解密密码为xxxx
cat > /home/greg/ansible/secret.txt << EOF xxxx EOF

cat > locker.yml << EOF
---
pw_developer: xxx
pw_manager: xxx
EOF

# 使用secret.txt加密locker.yml
ansible-vault encrypt --vault-id=./secret.txt locker.yml

# 检查
cat locker.yml
ansible-vault view --vault-id=./secret.txt locker.yml
```

## 14.创建批量添加用户role

```bash
wget https://xxx
cat user_list.yml

cat > user.yml << EOF
---
- name: create user on dev and test
  hosts: dev.test
  var_files:
    - locker.yml
    - user_list.yml
  tasks:
    - name: create group
      group:
        name: devops
    - name: create user
      user: 
        name: "{{ item['name'] }}"
        password: "{{ pw_developer | password_hash('sha512', 'mysecretsalt') }}"
        groups: devops
      loop: "{{ users }}"
      when: item.job == 'developer'
- name: create user on prod
  hosts: prod
  var_files:
    - locker.yml
    - user_list.yml
  tasks:
    - name: create group
      group:
        name: opsmgr
    - name: create user
      user:
        name: "{{ item['name'] }}"
        pssword: "{{ pw_developer | password_hash('sha512', 'mysecretsalt') }}"
        # 这里如果是group，就是添加的primary group
        groups: opsmgr
      loop: "{{ users }}"
      when: item.job == 'manager'
      
      
# 测试
anisble dev -a "id gzy001"
```

15.重新设置ansible vault 密码

```bash
wget http://xx
cat salaries.yml
# 查看密码
ansible-vault view salaries.yml
ansible-vault rekey salaries.yml

#测试
cat salaries.yml
ansible-vault view salaries.yml
```

