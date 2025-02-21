# 红帽系 Linux 环境下 Hysteria 2 部署

## 准备工作

在开始之前，请确保已具备以下条件：

*  **一台运行红帽系 Linux 的服务器**（包括但不限于 RHEL, AlmaLinux, Rocky Linux）
*  **一个域名，并将其 DNS 解析到服务器 IP 地址**

## 为什么选择 AlmaLinux

AlmaLinux 是一个基于 RHEL (Red Hat Enterprise Linux) 的免费开源发行版，具有以下优势：

*  **稳定可靠：** 继承 RHEL 的稳定性和可靠性，适合作为服务器环境
*  **高度兼容：** 与 RHEL 高度兼容，可以无缝运行 RHEL 上的应用程序
*  **免费开源：** 免费使用，无需支付任何费用
*  **社区支持：** 拥有庞大的社区和丰富的文档资源，方便用户获取帮助和支持

## 部署步骤

### 1. 关闭 SSH 的密码登录（可选）

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

### 2. 升级系统并安装必要软件

```shell
sudo dnf -y upgrade
sudo dnf -y install curl nano firewalld
sudo systemctl start firewalld.service
```

### 3. 配置防火墙

```shell
sudo firewall-cmd --permanent --add-service=http3
sudo firewall-cmd --reload
```

可以使用以下命令查看当前防火墙策略：

```shell
sudo firewall-cmd --list-all
```

### 4. 安装 Hysteria 2

```shell
sudo bash -c "$(curl -fsSL https://get.hy2.sh/)"
```

### 5. 修改配置文件

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

### 6. 启动服务

```shell
sudo systemctl enable --now hysteria-server.service
```

## HTTP/HTTPS 伪装

修改 `masquerade` 部分的配置为：

```yaml
masquerade:
  type: proxy
  proxy:
    url: https://almalinux.org/ # 可修改为伪装目标网站
    rewriteHost: true
  listenHTTP: :80
  listenHTTPS: :443
  forceHTTPS: true
```

重启服务并且添加相应防火墙规则：

```shell
sudo systemctl restart hysteria-server.service
sudo firewall-cmd --permanent --add-service={http,https}
sudo firewall-cmd --reload
```

## 客户端配置

```yaml
server: www.domain.com:443 # 修改为你的域名

auth: hunter2 # 修改为你的密码

bandwidth:
  up: 100 mbps # 修改为本地最大上传速率
  down: 100 mbps # 修改为本地最大下载速率

socks5:
  listen: 127.0.0.1:1080 # 根据需求修改端口

http:
  listen: 127.0.0.1:8080 # 根据需求修改端口
```
