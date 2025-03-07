---
title: Mac搭建Java开发环境
tags: [Java]
categories:
  - [Java]
date: 2025-03-07 15:56:17
---

## JDK的下载和安装

官网获取JDK:

https://www.oracle.com/cn/

![image-20250307160107769](https://p.ipic.vip/f9nxvj.png)



傻瓜式安装一路到底

## 配置环

在`~/.zshrc ` 追加

```
# 我选择的是21版本

# JDK
# 配置JDK路径
export JAVA_21_HOME=/Library/Java/JavaVirtualMachines/jdk-21.jdk/Contents/Home
# 设置默认JDK版本
export JAVA_HOME=$JAVA_21_HOME
export CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:.

# 配置alias命令动态切换JDK版本
alias jdk21="export JAVA_HOME=$JAVA_21_HOME"


```

修改完毕后, 执行

```she
# 让配置立即生效
$ source ~/.zshrc

$ java --version

# java 21.0.6 2025-01-21 LTS
# Java(TM) SE Runtime Environment (build 21.0.6+8-LTS-188)
# Java HotSpot(TM) 64-Bit Server VM (build 21.0.6+8-LTS-188, mixed mode, sharing)
```



##  手撸Java代码验证

```
 $ vim test.java
 
class Test {
	public static void main(String[] args) {
		System.out.println("Hello World!");
	}
}

 $ java test.java
 # Hello World!

```

## 安装IntelliJ IDEA 社区版

官网地址： https://www.jetbrains.com/idea/download/?section=mac

界面往下拉, 选择`IntelliJ IDEA Community Edition`版本.

![image-20250307161623022](https://p.ipic.vip/yhw7sg.png)

