---
title: 服务器升级OpenSSH脚本
tags: [Linux]
categories:
  - [Linux]
date: 2025-04-29 14:56:58
---

# 麒麟V10服务器升级OpenSSH


> 服务器架设在本地局域网，无法访问外网。 为修复OpenSSH漏洞,需要升级OpenSSH

## 下载所有依赖包
- openssh-9.8p1.tar.gz
- openssl-3.3.1.tar.gz
- zlib-1.3.1.tar.gz

## 脚本正文

```bash
#!/bin/bash

cd /opt
tar -zxf zlib-1.3.1.tar.gz
cd zlib-1.3.1
#检查配置并设置安装路径
./configure --prefix=/usr/local/zlib
#编译安装
make && make install

cd /opt
tar -zxf openssl-3.3.1.tar.gz
cd openssl-3.3.1
./Configure --prefix=/usr/local/openssl
#允许4个核心编译，速度快
make -j 4 && make install
#让系统能够找到并加载 OpenSSL 库
echo "/usr/local/openssl/lib" >> /etc/ld.so.conf
#应用库配置
ldconfig

cd /opt
tar -zxf openssh-9.8p1.tar.gz
cd openssh-9.8p1/
#备份
mv /etc/ssh /etc/ssh_old.bak
mv  /usr/bin/ssh /usr/bin/ssh_old.bak
mv  /usr/bin/ssh-keygen /usr/bin/ssh-keygen_old.bak
mv  /usr/sbin/sshd /usr/sbin/sshd_old.bak
cp /etc/pam.d/sshd /etc/pam.d/sshd_old.bak
./configure --prefix=/usr/local/openssh --sysconfdir=/etc/ssh --with-pam --with-ssl-dir=/usr/local/openssl --with-zlib=/usr/local/zlib
make && make install

chown -R root:root /etc/ssh
chmod 600 /etc/ssh/*_key
#更新系统服务和配置文件
cp /usr/local/openssh/sbin/sshd /usr/sbin/sshd
cp /usr/local/openssh/bin/ssh /usr/bin/ssh
cp /usr/local/openssh/bin/ssh-keygen /usr/bin/ssh-keygen
cp -p contrib/redhat/sshd.init /etc/init.d/sshd
chmod +x /etc/init.d/sshd

 #执行前，请确认sshd_config路径
echo 'PermitRootLogin yes' >> /etc/ssh/sshd_config
echo 'PubkeyAuthentication yes' >> /etc/ssh/sshd_config
echo 'PasswordAuthentication yes' >> /etc/ssh/sshd_config


systemctl enable --now sshd
#重启ssh服务
systemctl restart sshd

ssh -V

```

新建文件`updateOpenSSH.sh`, 将上面内容复制进去

## 执行脚本

将上面4个文件`openssh-9.8p1.tar.gz` `openssl-3.3.1.tar.gz` `zlib-1.3.1.tar.gz` `updateOpenSSH.sh`通过U盘拷贝到服务器`/opt`路径下

给`updateOpenSSH.sh`执行权限

```
chmod +x updateOpenSSH.sh
```

执行脚本

```
sh updateOpenSSH.sh
```

## 验证版本

```
sshd -V
# 如果输出9.8p1 那么就是成功了!
```





