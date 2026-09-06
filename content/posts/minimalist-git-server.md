---
title: "128MB 内存也能跑 Git 服务器：SSH + 裸仓库实现零依赖部署"
date: 2026-08-03T20:00:13+08:00
categories: 计算机
description: "介绍如何使用原生Git搭建一个轻量级Git服务器。"
summary: "介绍如何使用原生Git搭建一个轻量级Git服务器。"
---
## 前言
我在VPS吧有一台闲置的NAT VPS，1C 128M的配置让它变得食之无味，弃之可惜，一直在想能不能把它用起来。

![hardware-specifications-of-free-nat-vps.png](https://img.mukunjin.com/2026/hardware-specifications-of-free-nat-vps.png)

最近准备把玩一下Git服务器，于是这台NAT鸡就有用了。那么，我到底是选Gitea，OneDev还是Forgejo呢？GitLab CE这些就别想了。

答案是，什么都不需要。**直接用原生Git方法搭，使用纯裸仓库。** 毕竟只是把玩一下而已，起到备份代码的作用。无Web界面，无权限管理，无法在线Code Review这些缺点对我来说无所谓了。要的就是零配置，极轻量。

## 核心原理
Git 原生支持通过 SSH 访问远程仓库，服务器端只需要提供 SSH 服务和 Git 命令即可。只需要系统有`git`命令，然后创建一个专门用来存仓库的系统用户，把仓库放在该用户目录下，通过 SSH 公钥认证控制访问即可。我这里以Alpine Linux为例进行实操。

## 开始把玩
### 服务端准备
我已经SSH上去了。先安装Git:
```bash
apk update
apk add git
```
创建 Git 专用系统用户：
```bash
adduser -D -s /bin/sh git
passwd git
```
由于我只使用密钥登录，需要把公钥复制给git用户。
```bash
mkdir -p /home/git/.ssh
cp /root/.ssh/authorized_keys /home/git/.ssh/
chown -R git:git /home/git/.ssh
chmod 700 /home/git/.ssh
chmod 600 /home/git/.ssh/authorized_keys
```
### 创建第一个仓库
```bash
# 切换到 git 用户
su - git

# 创建仓库目录
mkdir -p ~/repos/myproject.git
cd ~/repos/myproject.git

# 初始化为裸仓库
git init --bare

# 退出 git 用户
exit
```

### 客户端clone测试
我选择把标准端口22映射到外部21000。配置`SSH config`可以让命令更加简单，这里就不做了。
```bash
git clone ssh://git@别看我的IP:21000/home/git/repos/myproject.git
```
### 测试一下
我们在本地跑一下日常会用到的命令。
```bash
cd myproject
echo "test" > README.md
git add .
git commit -m "testing"   
```

![client-side-git-clone-test.png](https://img.mukunjin.com/2026/client-side-git-clone-test.png)

没有问题。
### 进阶配置
我们来配置一下限制git用户只能执行Git操作，这样 git 用户无法获得普通 Shell，只能通过 SSH 执行 Git 提供的仓库访问命令。
```bash
sed -i 's|/home/git:/bin/sh|/home/git:/usr/bin/git-shell|' /etc/passwd
```
## 结尾
完结撒花~终于把这台NAT VPS给用上了，感谢Supers大佬送的机器！