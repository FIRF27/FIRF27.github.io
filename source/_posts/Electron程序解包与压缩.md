---
title: Electron程序解包与压缩
tags: [Electron, js逆向]
categories:
  - [Electron, js逆向]
date: 2025-12-26 09:24:58
---

# Electron程序解包与压缩



## 安装 asar

```
$ npm install asar -g
```

## 解包

```
$ asar e app.asar app  

or

$ asar extract app.asar targetFolder
```

## 压缩

```
# 进入刚刚解包的文件夹
$ cd targetFolder && asar p ./ app.asar

or

$ asar pack /path/to/folder app.asar

```

