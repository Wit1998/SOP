# ansible-vault

加密yml或var文件

```
# 创建yml
ansible-vault create xxx.yml
# 查看yml
ansible-vault view xxx.yml
# 修改密码
ansible-vault rekey xxx.yml
# 修改yml
ansible-vault edit xxx.yml
# 解密yml
ansible-vault decrypt xxx.yml
# 加密yml
ansible-vault encrypt xxx.yml
# 解密成新文件，原文件还在
ansible-vault encrypt xxx.yml --output=test-1.yml
# 非交互查看
ansible-vault --vault-password-file=./mima.yml view test.yml
ansible-vault --vault-id=./mima.yml view test.yml

# 使用vault密码
ansible-playbook --vault-id @prompt xxx.yml
```

