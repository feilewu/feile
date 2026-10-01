
# 账户准备

```
# 1. 创建 git 用户（禁用密码登录，更安全）
sudo adduser git --disabled-password --gecos ""

# 2. 一键创建 .ssh 目录并生成 Ed25519 密钥对
# (利用 -u git 确保生成的文件直接属于 git 用户，无需后续 chown)
sudo -u git mkdir -p /home/git/.ssh
sudo chmod 700 /home/git/.ssh
sudo -u git ssh-keygen -t ed25519 -C "github-actions-deploy-key" -f /home/git/.ssh/github_actions -N ""

# 3. 将公钥加入授权列表
sudo -u git sh -c 'cat /home/git/.ssh/github_actions.pub >> /home/git/.ssh/authorized_keys'

# 4. 严格设置规范的权限
sudo chmod 600 /home/git/.ssh/authorized_keys
sudo chmod 644 /home/git/.ssh/github_actions.pub

```

# 目录赋权

用acl权限控制

```
sudo setfacl -R -m u:git:rwx /opt/1panel/www/sites
```


# GitHub仓库setting里配置

```
sudo cat /home/git/.ssh/github_actions
```

Repository secrets里新建一个键值对即可

key为SERVER_SSH_KEY，value为命令查询出的私钥值

![[github创建部署用户-1.png]]