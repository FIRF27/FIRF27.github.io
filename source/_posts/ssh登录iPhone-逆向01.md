---
title: ssh登录iPhone-逆向01
date: 2025-03-07 13:41:51
tags: [iOS, 逆向]
categories: 
  - [iOS]
---

# ssh登录iPhone-逆向01

## SSH登录iPhone

### 无线模式 (iphone和mac在同一局域网下)

- 在iPhone上通过Cydia安装OpenSSH工具
- mac终端输入$ssh 账户@ip地址 (iOS下2个常用账户：root、mobile)

例：`ssh root@170.16.4.000`

> 初始密码:alpine

### 有线模式/USB

> 该种方式采用的是Mac上的一个服务程序usbmuxd

位置在/System/Library/PrivateFrameworks/MobileDevice.framework/Resources/usbmuxd

- 第一步需要下载一个usbmuxd工具包（主要用到里面的usbmux.py、tcprelay.py文件）

[https://git.sukimashita.com/usbmuxd.git/](https://links.jianshu.com/go?to=https://git.sukimashita.com/usbmuxd.git/)

选择1.0.8版本 , 将usbmux.py、tcprelay.py文件放在方便的位置

- 终端输入 `python tcprelay.py的路径 -t 22:10010`

例：`python /Users/mac/Desktop/python-client/tcprelay.py -t 22:10010`

> iPhone默认是使用22端口进行SSH通信，采用的是TCP协议. 我们iPhone 22端口映射到Mac本地的10010端口

- 另起一个终端窗口输入`ssh root@localhost -p 10010`

或`ssh root@127.0.0.1 -p 10010`

> 附: 终端输入`exit`退出登录

### 拓展:ssh免密登陆

```
修改密码啥的见Cydia介绍
```

*ssh 每次登录都需要输入密码令人烦躁*

- 创建ssh

```
ssh-keygen
```

- 将生成的公钥追加到授权文件尾部

终端执行下面命令（如果iphone没有.ssh文件夹 自己手动创建）

```
scp -P 10010 ~/.ssh/id_rsa.pub root@localhost:~/.ssh
```

接着登陆iPhone 执行命令

```
cat ~/.ssh/id_rsa.pub >> authorized_keys
rm ~/.ssh/id_rsa.pub
```

退出手机在登录就不需要密码了
