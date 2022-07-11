# Iptables & firewalld



# firewalld

### 放行服务和放行端口

```bash
# service规则放行
firewall-cmd --add-service=http
firewall-cmd --remove-service=http
firewall-cmd --list-services
#port规则的放⾏
firewall-cmd --add-port=33333/tcp
firewall-cmd --add-port=33333/udp
firewall-cmd --list-ports
```

### 富规则

```bash

firewall-cmd --add-rich-rule="rule family=ipv4 source address=2.2.2.0/24 port port=8089 protocol=tcp reject"
#如果想让上⾯的临时规则变成永久规则，可以使⽤下⾯的命令
firewall-cmd --runtime-to-permanent

#如果使⽤firewall-cmd --reload命令之后规则还会存在，就是永久的规则

#重载防⽕墙规则
firewall-cmd --reload

#为了避免防⽕墙规则reload，建议敲两次
firewall-cmd --add-service=ftp --permanent
firewall-cmd --add-service=ftp
```

### firewalld转发端⼝（DNAT）

```

```

