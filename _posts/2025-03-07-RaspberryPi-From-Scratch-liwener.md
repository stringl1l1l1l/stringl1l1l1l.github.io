---
title: "从零开始的树莓派折腾之旅"
date: 2025-03-07
author: liwener
---

## 环境部署

查看发行版版本

```shell
lsb_release -a
```



### Apt 换源

```shell
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo vi /etc/apt/sources.list
```

```
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware

# 以下安全更新软件源包含了官方源与镜像站配置，如有需要可自行修改注释切换
deb https://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
# deb-src https://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
```



### Zerotier 安装

虚拟局域网，方便跨局域网间的 SSH 连接。

```shell
curl -s https://install.zerotier.com | sudo bash

sudo zerotier-cli join <YOUR_NETWORK_ID>
sudo systemctl enable --now zerotier-one
systemctl status zerotier-one
```



### 启用 mDNS

使得局域网内可以使用 {hostname}.local 访问到主机。

```shell
sudo apt install avahi-daemon -y
sudo systemctl start --now avahi-daemon
systemctl status avahi-daemon 	# 查看 avahi-daemon 启动状态
```

（可选：修改主机名）

```shell
sudo hostnamectl set-hostname <NEW_HOSTNAME>
```



### 创建新sudo用户

方法一（推荐）
```shell
sudo adduser username
sudo usermod -aG sudo username
```

方法二
```shell
sudo useradd -m username # 	-m 选项会自动创建 /home/username 目录
sudo passwd username
sudo useradd -m -s /bin/bash username # -s /bin/bash 让用户使用 bash 作为默认 shell
sudo useradd -m -g users -G sudo username # -g users 设定主组, -G sudo 让用户加入 sudo 组（赋予 sudo 权限）
```

相关命令总结

| **任务**             | **命令**                       |
| -------------------- | ------------------------------ |
| 创建用户（无家目录） | sudo useradd username          |
| 创建用户并创建家目录 | sudo useradd -m username       |
| 交互式创建用户       | sudo adduser username          |
| 设置密码             | sudo passwd username           |
| 赋予 sudo 权限       | sudo usermod -aG sudo username |
| 删除用户             | sudo userdel username          |
| 删除用户并删除家目录 | sudo userdel -r username       |



#### Oh-my-zsh 安装

首先安装 zsh

```sh
sudo apt install zsh -y
```

然后安装 oh-my-zsh

``````sh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
``````

插件下载和使用

```sh
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

vim ~/.zshrc

source ~/.zshrc
```

```
# 添加到 ~/.zshrc
plugins=(git web-search jsontools z zsh-syntax-highlighting zsh-autosuggestions)
export ZSH_AUTOSUGGEST_STRATEGY=(history completion)
```

```
docker pull forceless/pptagent
```

安装 Powerlevel10k 主题

```sh
git clone https://github.com/romkatv/powerlevel10k.git $ZSH_CUSTOM/themes/powerlevel10k
```



### 配置 SSH 秘钥

只有公钥可以上传，服务器使用公钥来验证由私钥生成的签名，认证客户端身份。

客户端生成公钥

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
将公钥上传到服务器
```bash
# Windows 客户端
scp ~/.ssh/id_rsa.pub user@example.com:~/

# 在服务器端执行下面的命令
mkdir  ~/.ssh
chmod 700 ~/.ssh
cat  ~/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
rm ~/id_rsa.pub
```
```bash
# Linux 客户端
ssh-copy-id user@example.com
```



## 软件安装

### Docker 安装

```shell
 curl -fsSL https://get.docker.com -o get-docker.sh
 sudo sh get-docker.shdocker pull forceless/pptagent
```

（可选）添加用户到 docker 组
```shell
sudo usermod -aG docker <username>
newgrp docker
```



### Clang 套件安装

```sh
# 1. clangd
sudo apt install clangd
# xx替换为填写安装的clangd版本号
sudo update-alternatives --install /usr/bin/clangd clangd /usr/bin/clangd-xx 100

# 2. clang-tidy
sudo apt install clang-tidy

# 3. clang-format
sudo apt install clang-format

# 4. clang
sudo apt install clang
```

.clang-format 配置文件

```yaml
BasedOnStyle: WebKit
Language: Cpp

# 缩进规则
UseTab: Never        # 禁止使用真实 Tab 字符，改用空格模拟
TabWidth: 4          # 1 个 Tab 对应 4 个空格
IndentWidth: 4       # 缩进宽度为 4 个空格（等同于 Tab 宽度）

# 大括号换行风格（与 WebKit 风格一致）
BreakBeforeBraces: Linux

# 其他常用 WebKit 风格优化
AllowShortBlocksOnASingleLine: true
AllowShortFunctionsOnASingleLine: Inline
BinPackParameters: false           # 禁止将函数参数挤在一行
ColumnLimit: 120                   # 行宽限制 120
PointerAlignment: Right            # 指针符号 `*` 靠近类型名
SpaceBeforeParens: ControlStatements  # `if`/`for` 等控制语句后加空格

# 额外自定义规则（可选）
AlignAfterOpenBracket: AlwaysBreak
AlignConsecutiveMacros: true
AlignConsecutiveDeclarations: false # 变量对齐
```



### VPN 与内网穿透

最近去武汉出差，一直没有时间折腾的 VPN 突然成为了重大需求。虽然已经折腾过 frp 了，但对于想要远程控制复杂的内网环境，还是配个 VPN 省事。

先介绍一下我的网络环境。出差时手上只有一台 Macbook Air M1 (Mac)，并且白嫖了一台 Azure 服务器 (VPS)。校园大局域网内共有 4 台设备：宿舍内一台不断电的 Win 11 个人 PC（Win），作为 SMB / WebDav 服务器提供流媒体服务，有线连接到一台 TP-LINK 无线路由器；实验室局域内一台树莓派 4b（Raspberrypi），以及一台不断电的 Ubuntu 服务器 (Server)。除了路由器之外，上述所有设备均通过 zerotier 组成 `172.22.0.0/16` 下的虚拟局域网。

#### Wireguard
VPN 工具选择了 Wireguard，相比 OpenVPN 复杂的证书分发和管理，Wireguard 简单的多，使用类似 SSH RSA 非对称加密的思想创建隧道。

##### 安装

```shell
# 参考官网信息：https://www.wireguard.com/install/

# macos: 去 github 下可以避开恶心的 App store
https://github.com/zakosaba/wireguard-macos-app

# ubuntu:
sudo apt update
sudo apt install wireguard -y

# win: 
https://download.wireguard.com/windows-client/wireguard-installer.exe

```

##### 配置
- **VPS**
1. 生成公私钥
```shell
cd ~
umask 077
wg genkey | tee privatekey | wg pubkey > publickey
```

2. 创建并编辑配置文件：
```shell
sudo touch wg0.conf
# 创建硬链接方便直接在HOME目录下编辑
sudo ln wg0.conf /etc/wireguard/wg0.conf
```

```toml
# wg0.conf
[Interface]
Address = 10.10.0.1/32  # 自定义VPS在VPN中的内网IP地址
SaveConfig = true
ListenPort = 51820      # 必须和您在安全组中开放的UDP端口一致
PrivateKey = xxx        # 前一步生成的 privatekey 

# --- Win ---
[Peer]
PublicKey = xxxx
AllowedIPs = 10.10.0.2/32

# --- Mac ---
[Peer]
PublicKey = xxxx
AllowedIPs = 10.10.0.3/32

# --- Raspberrypi ---
[Peer]
PublicKey = xxxx
AllowedIPs = 10.10.0.4/32

```

3. 应用配置文件(必须在 /etc/wireguard 下)
```shell
sudo wg-quick up wg0 # wg0 对应/etc/wireguard目录下的配置文件 wg0.conf
```

4. 关闭
```shell
sudo wg-quick down wg0
```


- **Clients**
1. 生成公私钥
2. 创建并编辑配置文件
```shell
[Interface]
PrivateKey = xxxx # 刚刚生成的本地公钥
Address = 10.10.0.x/32

[Peer]
PublicKey = xxxx # VPN服务端的公钥
AllowedIPs = 10.10.0.0/24
Endpoint = {VPS.IP}:51820
PersistentKeepalive = 25
```
3. 应用配置文件

##### 进阶
完成上面的步骤之后已经可以根据配置文件中定义的 ip 10.10.0.x/32 互相访问了，但对于用惯了局域网内 zerotier + mDNS 的我，这个