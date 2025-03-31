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
jar -tvf xxx.jar | grep abc.class

 # 输出结果示例 BOOT-INF/classes/com/netinfo/system/service/device/impl/abc.class
```

### 2. 解压指定路径下的文件

```
jar -xvf xxx.jar /path/to/jar/file/location

eg. 
jar -xvf xxx.jar BOOT-INF/classes/com/netinfo/system/service/device/impl/abc.class
```

###  3. 替换解压目录的下的文件

```
mv -i abc.class BOOT-INF/classes/com/netinfo/system/service/device/impl/abc.class
```

### 4. 更新到jar包中

```
jar -uvf xxx.jar /path/to/jar/file/location

eg.
jar -xvf xxx.jar BOOT-INF/classes/com/netinfo/system/service/device/impl/abc.class

# 如果一次替换多个文件, 直接拼接多个文件地址
eg. 
jar -xvf xxx.jar BOOT-INF/classes/com/netinfo/system/service/device/impl/abc.class BOOT-INF/classes/com/netinfo/system/service/device/impl/efg.class

```

> 如果替换的文件名带有`$`, 需将`$`转义.  `$`前加`\`
    
 
