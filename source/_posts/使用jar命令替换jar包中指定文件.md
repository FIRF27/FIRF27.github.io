---
title: 使用jar命令替换jar包中指定文件
tags: [Java]
categories:
  - [Java]
date: 2025-03-18 17:53:47
---

# 使用jar命令替换jar包中指定文件

### 替换jar包中指定文件

#### 1. 列出指定文件的路径

```
jar -tvf xxx.jar | grep xxx.class

 # 输出结果示例 BOOT-INF/classes/com/netinfo/system/service/device/impl/SyncDevInfoServiceImpl.class
```

### 2. 解压指定路径下的文件

```
jar -xvf xxx.jar /path/to/jar/location

eg. 
jar -xvf xx.jar  BOOT-INF/classes/com/netinfo/system/service/device/impl/SyncDevInfoServiceImpl.class
```

###  3. 替换解压目录的下的文件

```
mv -i newYY.class BOOT-INF/classes/com/netinfo/system/service/device/impl
```

### 4. 更新到jar包中

```
jar -uvf xxx.jar /path/to/newYY.class
```

 
