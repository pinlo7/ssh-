SSH 免密登录：基于非对称加密的实现与配置

SSH免密登录基于非对称加密实现，核心是本地生成密钥对、公钥上传服务器、私钥本地验证，一次配置后永久免密，比密码登录更安全高效。

一、核心原理

- 本地生成私钥（id_ed25519） + 公钥（id_ed25519.pub），私钥本地保密、公钥可公开。

- 公钥上传到服务器 ~/.ssh/authorized_keys（认证白名单）。

- 登录时：服务器用公钥加密随机数 → 本地用私钥解密 → 验证通过则免密登录。

- 配置多个服务器免密，只需要生成一次密钥，把同一个公钥分发到所有服务器就行，不用每个服务器都重新生成密钥。 

二、Linux/macOS 配置（最常用）

1. 本地生成密钥对（推荐 ED25519）

# 生成ED25519密钥（全程回车，不设密码）
ssh-keygen -t ed25519 -C "your-email@example.com"

# 兼容旧系统用RSA（4096位）
# ssh-keygen -t rsa -b 4096 -C "your-email@example.com"

生成后默认在 ~/.ssh/ 目录：

- id_ed25519：私钥（严禁泄露）

- id_ed25519.pub：公钥（需上传）

2. 一键上传公钥到服务器（推荐）

# 自动添加到 authorized_keys
ssh-copy-id user@remote-ip
# 例：ssh-copy-id root@192.168.1.100

输入服务器密码，完成后即可免密登录。

3. 手动上传（无 ssh-copy-id 时）

# 查看公钥
cat ~/.ssh/id_ed25519.pub

# 复制到服务器
cat ~/.ssh/id_ed25519.pub | ssh user@remote-ip "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys && chmod 700 ~/.ssh"

4. 验证免密登录

ssh user@remote-ip
# 无需密码直接登录

三、Windows 配置（PowerShell/Git Bash）

1. 生成密钥

# PowerShell 执行
ssh-keygen -t ed25519 -C "your-email@example.com"

默认路径：C:\Users\你的用户名.ssh\

2. 上传公钥

# 同Linux
ssh-copy-id user@remote-ip

3. 验证

ssh user@remote-ip

四、常见问题与权限修复

1. 权限必须正确（否则免密失效）

# 服务器端执行
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

2. 多服务器/多密钥管理（~/.ssh/config）

# 编辑配置
vim ~/.ssh/config

# 添加内容
Host my-server
    HostName 192.168.1.100
    User root
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes

# 登录时简化
ssh my-server

3. 密钥加密（带密码）

生成密钥时设置 passphrase，登录需先解锁私钥：

# 缓存密钥（一次输入，会话有效）
eval $(ssh-agent)
ssh-add ~/.ssh/id_ed25519

五、安全建议

- 优先用 ED25519，比 RSA 更安全、体积更小。

- 私钥 绝不泄露、绝不上传服务器。

- 服务器禁用密码登录（仅密钥）：

sudo vim /etc/ssh/sshd_config
PasswordAuthentication no
ChallengeResponseAuthentication no
sudo systemctl restart sshd

需要我帮你生成一份可直接复制的一键配置脚本，包含密钥生成、公钥上传、权限修复和多服务器配置模板吗？
