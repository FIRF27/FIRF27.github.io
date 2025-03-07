---
title: 使用symbolicatecrash解析KSCrash的崩溃日志
tags: [iOS]
categories:
  - [iOS]
date: 2025-03-07 15:14:14
---

### 1.新建文件夹`crashLog`

```shell
$ mkdir -p crashLog 
# pwd: /Users/apple/Desktop/crashLog
```

### 2. 找出symbolicatecrash

```shell
$ cd /Applications/Xcode.app/Contents
$ find . -name symbolicatecrash
# ./SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash

# 复制symbolicatecrash到crashLog文件夹下
$ cp ./SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash  ~/Desktop/crashLog
# /path/to/crashLog
```

### 3. 复制temp.crash文件到crashLog文件夹

### 4. 复制xxApp.app.dSYM文件到crashLog文件夹

### 4. 在crashLog文件夹下解析

```shell
 # 1./path/to/crashLog
 $ cd ~/Desktop/crashLog
 
 # 2.执行命令
 $ xcode-select -p
 # /Applications/Xcode.app/Contents/Developer
 $ export DEVELOPER_DIR="/Applications/XCode.app/Contents/Developer"

 # 开始解析
 $ ./symbolicatecrash temp.crash xxApp.app.dSYM > crash.log
```





