

# Ansible 

### 说明：

- 架构：在Ansible的架构中有两种类型的机器，一类是控制节点，一类是被管理主机。

- inventory：被管理主机被列在Inventory中，inventory将这些被管理主机组织进不同的group中进行管理。inventory可以是静态和动态
- playbook：一个包含一个或更多的play文件叫做 playbook。Ansible用户创建高等级的plays来保证主机处在特定的状态。一个剧本在主机上执行 一系列的任务。这些剧本在text文件中被表达为YAML的格式。

- task：每个task运行一个带着特定参数的module，每个module都是一段很少的代码，该代码可以用Python， PowerShell或其他的语言写。

- handle:

- role

- 

## install（ubuntu）

```yaml
# 统一的版本为2.9.12
sudo apt update
sudo apt install python3-pip -y
pip3 install ansible==2.9.12 --user -i https://pypi.douban.com/simple/
# setuptools
# pip3 install -U pip setuptools
# ~/.local/bin/
# ~/.bashrc~/.zshrc

ANSIBLE_PATH=~/.local/bin/
PATH=$PATH:$ANSIBLE_PATH
```



### 注意：

- ansible的控制节点和被管理主机都要控制统一版本，最好是版本都一致。

- ansible默认使用ssh连接被管理主机，如果是windows需要安装Python-winrm

  

## tags

当play被标记了后，可以使用--tags参数来执行被标记了的play。

--tags

--skip-tags

### 特殊tags

Ansible有特殊的tag，比如always。即使使用--skip-tags选项，always标记的task也会被执行，除非显示的使用--skip-tags always。

有三个特殊的tag被--tags参数使用：
1.tagged，所有tag的都执行
2.untagged，所有tag的都不执行
3.all，缺省行为

