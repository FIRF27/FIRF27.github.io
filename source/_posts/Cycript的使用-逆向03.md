---
title: Cycript的使用-逆向03
date: 2025-03-07 13:42:31
tags: [iOS, 逆向]
categories: 
  - [iOS]
---

# Cycript的使用-逆向03

#### Cycript能干什么

> Cycript可以用来探索、修改、调试正在运行的Mac/iOS App

[官网：http://www.cycript.org/](http://www.cycript.org/)

[文档：http://www.cycript.org/manual/](http://www.cycript.org/manual/)

通过Cydia安装Cycript就可以使用了

#### Cycript开启和关闭

```
Cycript

Cycript -p 进程ID/进程名称

# 退出：
controle + D

取消输入：control + C
```

#### 获取进程名称/进程id

```
# 获取所有进程
ps -A
ps - A ｜ grep 关键词
```

#### Cycript常用语法

```
UIApp UIApp 等价于[UIApplication sharedApplication]

定义变量 var 变量名 = 变量值

用内存地址获取对象 #内存地址

已加载的所有OC类 ObjectiveC.classes

查看对象的所有成员变量 *对象

递归打印View的所有子控件 对象.recursiveDescription().toString()

筛选出某种类型的对象 choose(UIViewController) choose(UITableViewCell)
```

#### Cycript高级使用

我们可以将常用的Cycript代码封装在一个.cy文件中
然后将.cy文件拖进手机`/usr/lib/cycript0.9`路径下
使用时 导入XX.py @import XX
