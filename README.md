# 红帽系 Linux 环境下 Hysteria 2 部署

## 准备工作

在开始之前，请确保已具备以下条件：

*  一台运行红帽系 Linux 的服务器（包括并不限于 RHEL, AlmaLinux, Rocky Linux）
*  一个域名，并将其 DNS 解析到服务器 IP 地址

## 为什么选择 AlmaLinux

AlmaLinux 是一个基于 RHEL (Red Hat Enterprise Linux) 的免费开源发行版，具有以下优势：

*  **稳定可靠：** 继承 RHEL 的稳定性和可靠性，适合作为服务器环境
*  **高度兼容：** 与 RHEL 高度兼容，可以无缝运行 RHEL 上的应用程序
*  **免费开源：** 免费使用，无需支付任何费用
*  **社区支持：** 拥有庞大的社区和丰富的文档资源，方便用户获取帮助和支持

## 部署步骤

### 1. 配置 DNS 将域名指向 VPS 的 IP 地址

### 2. 关闭 SSH 的密码登录（可选）

为了提高服务器安全性，建议关闭 SSH 密码登录，并使用密钥登录。**关闭密码登录前请务必确认密钥已经正确配置**

```shell
sudo nano /etc/ssh/sshd_config
```

修改以下两项配置为 `no`：

```
PasswordAuthentication no
KbdInteractiveAuthentication no
```

修改完成后重新载入配置使其生效：

```shell
sudo systemctl reload-or-restart sshd.service
```

### 3. 升级系统并安装必要软件
```shell
sudo dnf -y upgrade
sudo dnf -y install curl nano firewalld
sudo systemctl start firewalld.service
```

### 4. 配置防火墙
```shell
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-port=443/udp
sudo firewall-cmd --reload
```

可以使用以下命令查看当前防火墙策略：

```shell
sudo firewall-cmd --list-all
```

### 5. 安装 Hysteria 2
```shell
sudo bash -c "$(curl -fsSL https://get.hy2.sh/)"
```

### 6. 修改配置文件
```shell
sudo nano /etc/hysteria/config.yaml
```

将以下内容复制到 `config.yaml` 文件中，并根据实际情况进行修改：

```yaml
listen: :443

acme:
  domains:
    - www.domain.com # 替换为你的域名
  email: xxx@gmail.com # 替换为你的邮箱地址

auth:
  type: password
  password: hunter2 # 替换为你的密码

masquerade:
  type: proxy
  proxy:
    url: https://almalinux.org/
    rewriteHost: true
```

### 7. 启动服务
```shell
sudo systemctl enable --now hysteria-server.service
```
