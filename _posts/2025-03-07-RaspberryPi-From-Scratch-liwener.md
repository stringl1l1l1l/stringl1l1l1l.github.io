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



